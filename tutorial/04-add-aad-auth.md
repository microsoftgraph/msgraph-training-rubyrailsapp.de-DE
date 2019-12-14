<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen. Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten. In diesem Schritt integrieren Sie das [omniauth-oauth2-](https://github.com/omniauth/omniauth-oauth2) Juwel in die Anwendung und erstellen eine benutzerdefinierte omniauth-Strategie.

Erstellen Sie zunächst eine separate Datei, in der die APP-ID und der Schlüssel gespeichert sind. Erstellen Sie eine neue Datei `oauth_environment_variables.rb` mit dem `./config` Namen im Ordner, und fügen Sie den folgenden Code hinzu.

```ruby
ENV['AZURE_APP_ID'] = 'YOUR_APP_ID_HERE'
ENV['AZURE_APP_SECRET'] = 'YOUR_APP_SECRET_HERE'
ENV['AZURE_SCOPES'] = 'openid profile email offline_access user.read calendars.read'
```

Ersetzen `YOUR_APP_ID_HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR_APP_SECRET_HERE` und ersetzen Sie durch das Kennwort, das Sie generiert haben.

> [!IMPORTANT]
> Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die Datei `oauth_environment_variables.rb` aus der Quellcodeverwaltung auszuschließen, damit versehentlich keine APP-ID und Ihr Kennwort verloren gehen.

Fügen Sie jetzt Code hinzu, um diese Datei zu laden, falls Sie vorhanden ist. Öffnen Sie `./config/environment.rb` die Datei, und fügen Sie den folgenden `Rails.application.initialize!` Code vor der Codezeile hinzu.

```ruby
# Load OAuth settings
oauth_environment_variables = File.join(Rails.root, 'config', 'oauth_environment_variables.rb')
load(oauth_environment_variables) if File.exist?(oauth_environment_variables)
```

## <a name="setup-omniauth"></a>Setup OmniAuth

Sie haben das `omniauth-oauth2` gem bereits installiert, aber damit es mit den Azure OAuth-Endpunkten funktioniert, müssen Sie [eine OAuth2-Strategie erstellen](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy). Dies ist eine Ruby-Klasse, die die Parameter für die Erstellung von OAuth-Anforderungen an den Azure-Anbieter definiert.

Erstellen Sie eine neue Datei `microsoft_graph_auth.rb` mit dem `./lib` Namen im Ordner, und fügen Sie den folgenden Code hinzu.

```ruby
require 'omniauth-oauth2'

module OmniAuth
  module Strategies
    # Implements an OmniAuth strategy to get a Microsoft Graph
    # compatible token from Azure AD
    class MicrosoftGraphAuth < OmniAuth::Strategies::OAuth2
      option :name, :microsoft_graph_auth

      DEFAULT_SCOPE = 'openid email profile User.Read'.freeze

      # Configure the Microsoft identity platform endpoints
      option  :client_options,
              site:          'https://login.microsoftonline.com',
              authorize_url: '/common/oauth2/v2.0/authorize',
              token_url:     '/common/oauth2/v2.0/token'

      # Send the scope parameter during authorize
      option :authorize_options, [:scope]

      # Unique ID for the user is the id field
      uid { raw_info['id'] }

      # Get additional information after token is retrieved
      extra do
        {
          'raw_info' => raw_info
        }
      end

      def raw_info
        # Get user profile information from the /me endpoint
        @raw_info ||= access_token.get('https://graph.microsoft.com/v1.0/me').parsed
      end

      def authorize_params
        super.tap do |params|
          params['scope'.to_sym] = request.params['scope'] if request.params['scope']
          params[:scope] ||= DEFAULT_SCOPE
        end
      end

      # Override callback URL
      # OmniAuth by default passes the entire URL of the callback, including
      # query parameters. Azure fails validation because that doesn't match the
      # registered callback.
      def callback_url
        options[:redirect_uri] || (full_host + script_name + callback_path)
      end
    end
  end
end
```

Nehmen Sie sich einen Moment Zeit, um zu überprüfen, was dieser Code tut.

- Sie legt die `client_options` fest, um die Microsoft Identity Platform-Endpunkte anzugeben.
- Er gibt an, `scope` dass der Parameter während der Autorisierungsphase gesendet werden soll.
- Die `id` Eigenschaft des Benutzers wird als eindeutige ID für den Benutzer zugeordnet.
- Es verwendet das Zugriffstoken zum Abrufen des Benutzerprofils aus Microsoft Graph, um den `raw_info` Hash auszufüllen.
- Die Rückruf-URL wird überschrieben, um sicherzustellen, dass Sie mit dem registrierten Rückruf im App-Registrierungs Portal übereinstimmt.

Nachdem wir die Strategie definiert haben, müssen wir OmniAuth so konfigurieren, dass Sie verwendet wird. Erstellen Sie eine neue Datei `omniauth_graph.rb` mit dem `./config/initializers` Namen im Ordner, und fügen Sie den folgenden Code hinzu.

```ruby
require 'microsoft_graph_auth'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :microsoft_graph_auth,
           ENV['AZURE_APP_ID'],
           ENV['AZURE_APP_SECRET'],
           scope: ENV['AZURE_SCOPES']
end
```

Dieser Code wird ausgeführt, wenn die APP gestartet wird. Es lädt die OmniAuth-Middleware mit dem `microsoft_graph_auth` Anbieter, der mit den Umgebungsvariablen in `oauth_environment_variables.rb`festgelegt konfiguriert ist.

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

Nachdem die OmniAuth-Middleware nun konfiguriert wurde, können Sie mit dem Hinzufügen der Anmeldung zur APP fortfahren. Führen Sie den folgenden Befehl in der CLI aus, um einen Controller für die Anmeldung und Abmeldung zu generieren.

```Shell
rails generate controller Auth
```

Öffnen Sie die Datei `./app/controllers/auth_controller.rb`. Fügen Sie der `AuthController` -Klasse eine Rückrufmethode hinzu. Diese Methode wird von der OmniAuth-Middleware aufgerufen, sobald der OAuth-Fluss abgeschlossen ist.

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Temporary for testing!
  render json: data.to_json
end
```

Im Moment wird der von OmniAuth bereitgestellte Hash gerendert. Wir verwenden dies, um zu überprüfen, ob unsere Anmeldung funktionsfähig ist, bevor Sie fortfahren. Bevor wir testen, müssen wir die Routen zu `./config/routes.rb`hinzufügen.

```ruby
# Add route for OmniAuth callback
match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
```

Aktualisieren Sie nun die Ansichten für die Anmeldung. Öffnen `./app/views/layouts/application.html.erb`Sie. Ersetzen Sie die `<a href="#" class="nav-link">Sign In</a>` -Verbindung durch Folgendes.

```html
<%= link_to "Click here to sign in", "/auth/microsoft_graph_auth", method: :post, class: "nav-link" %>
```

Öffnen Sie `./app/views/home/index.html.erb` die Datei, und `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` ersetzen Sie die folgende.

```html
<%= link_to "Click here to sign in", "/auth/microsoft_graph_auth", method: :post, class: "btn btn-primary btn-large" %>
```

Starten Sie den Server, und `https://localhost:3000`navigieren Sie zu. Klicken Sie auf die Anmeldeschaltfläche, und Sie sollten zu `https://login.microsoftonline.com`umgeleitet werden. Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu. Der Browser wird an die APP umgeleitet, wobei der von OmniAuth generierte Hash angezeigt wird.

```json
{
  "provider": "microsoft_graph_auth",
  "uid": "eb52b3b2-c4ac-4b4f-bacd-d5f7ece55df0",
  "info": {
    "name": null
  },
  "credentials": {
    "token": "eyJ0eXAi...",
    "refresh_token": "OAQABAAA...",
    "expires_at": 1529517383,
    "expires": true
  },
  "extra": {
    "raw_info": {
      "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
      "id": "eb52b3b2-c4ac-4b4f-bacd-d5f7ece55df0",
      "businessPhones": [
        "+1 425 555 0109"
      ],
      "displayName": "Adele Vance",
      "givenName": "Adele",
      "jobTitle": "Retail Manager",
      "mail": "AdeleV@contoso.onmicrosoft.com",
      "mobilePhone": null,
      "officeLocation": "18/2111",
      "preferredLanguage": "en-US",
      "surname": "Vance",
      "userPrincipalName": "AdeleV@contoso.onmicrosoft.com"
    }
  }
}
```

## <a name="storing-the-tokens"></a>Speichern der Token

Nachdem Sie nun Token erhalten können, ist es an der Zeit, eine Möglichkeit zum Speichern in der APP zu implementieren. Da es sich um eine Beispiel-App handelt, speichern Sie Sie aus Gründen der Einfachheit in der Sitzung. Eine reale APP würde eine zuverlässigere sichere Speicherlösung wie eine Datenbank verwenden.

Öffnen Sie die Datei `./app/controllers/application_controller.rb`. Hier werden alle unsere Methoden zur Tokenverwaltung hinzugefügt. Da alle anderen Controller die `ApplicationController` Klasse erben, können Sie diese Methoden verwenden, um auf die Token zuzugreifen.

Fügen Sie der Klasse `ApplicationController` die folgende Methode hinzu. Die-Methode verwendet den OmniAuth-Hash als Parameter und extrahiert die relevanten Informationsbits und speichert diese dann in der Sitzung.

```ruby
def save_in_session(auth_hash)
  # Save the token info
  session[:graph_token_hash] = auth_hash.dig(:credentials)
  # Save the user's display name
  session[:user_name] = auth_hash.dig(:extra, :raw_info, :displayName)
  # Save the user's email address
  # Use the mail field first. If that's empty, fall back on
  # userPrincipalName
  session[:user_email] = auth_hash.dig(:extra, :raw_info, :mail) ||
                         auth_hash.dig(:extra, :raw_info, :userPrincipalName)
end
```

Fügen Sie nun Accessorfunktionen hinzu, um den Benutzernamen, die e-Mail-Adresse und das Zugriffstoken wieder aus der Sitzung abzurufen.

```ruby
def user_name
  session[:user_name]
end

def user_email
  session[:user_email]
end

def access_token
  session[:graph_token_hash][:token]
end
```

Fügen Sie schließlich Code hinzu, der ausgeführt wird, bevor eine Aktion verarbeitet wird.

```ruby
before_action :set_user

def set_user
  @user_name = user_name
  @user_email = user_email
end
```

Mit dieser Methode werden die Variablen festgelegt, die `application.html.erb`das Layout (in) verwendet, um die Benutzerinformationen in der Navigationsleiste anzuzeigen. Wenn Sie ihn hier hinzufügen, müssen Sie diesen Code nicht in jeder einzelnen Controller Aktion hinzufügen. Dies wird jedoch auch für Aktionen in der `AuthController`ausgeführt, was nicht optimal ist. Fügen Sie der `AuthController` Klasse in `./app/controllers/auth_controller.rb` den folgenden Code hinzu, um die before-Aktion zu überspringen.

```ruby
skip_before_action :set_user
```

Aktualisieren Sie dann die `callback` -Funktion in `AuthController` der-Klasse, um die Token in der Sitzung zu speichern und zur Hauptseite zurückzuleiten. Ersetzen Sie die vorhandene `callback`-Funktion durch Folgendes.

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Save the data in the session
  save_in_session data

  redirect_to root_url
end
```

## <a name="implement-sign-out"></a>Implementieren der Abmeldung

Bevor Sie dieses neue Feature testen, fügen Sie eine Möglichkeit zum Abmelden hinzu. Fügen Sie der `AuthController` Klasse die folgende Aktion hinzu.

```ruby
def signout
  reset_session
  redirect_to root_url
end
```

Fügen Sie diese Aktion `./config/routes.rb`zu hinzu.

```ruby
get 'auth/signout'
```

Aktualisieren Sie nun die Ansicht, um `signout` die Aktion zu verwenden. Öffnen `./app/views/layouts/application.html.erb`Sie. Ersetzen Sie die `<a href="#" class="dropdown-item">Sign Out</a>` -Verbindung durch Folgendes:

```html
<%= link_to "Sign Out", {:controller => :auth, :action => :signout}, :class => "dropdown-item" %>
```

Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort. Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.

![Ein Screenshot der Startseite nach der Anmeldung](./images/add-aad-auth-01.png)

Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den **Abmelde** Link zuzugreifen. Durch Klicken auf **Abmelden** wird die Sitzung zurückgesetzt, und Sie kehren zur Startseite zurück.

![Screenshot des Dropdownmenüs mit dem Link zum Abmelden](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Aktualisieren von Token

Wenn Sie sich den von OmniAuth generierten Hash genauer ansehen, werden Sie feststellen, dass der Hash zwei Token enthält `token` : `refresh_token`und. Der Wert in `token` ist das Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird. Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.

Dieses Token ist jedoch nur kurzlebig. Das Token läuft eine Stunde nach seiner Ausgabe ab. Hier wird der `refresh_token` Wert nützlich. Das Aktualisierungstoken ermöglicht der APP, ein neues Zugriffstoken anzufordern, ohne dass sich der Benutzer erneut anmelden muss. Aktualisieren Sie den Token-Verwaltungscode, um die Token-Aktualisierung zu implementieren.

Öffnen `./app/controllers/application_controller.rb` Sie und fügen Sie `require` die folgenden Anweisungen am oberen Rand hinzu:

```ruby
require 'microsoft_graph_auth'
require 'oauth2'
```

Fügen Sie dann die folgende Methode zur `ApplicationController` -Klasse hinzu.

```ruby
def refresh_tokens(token_hash)
  oauth_strategy = OmniAuth::Strategies::MicrosoftGraphAuth.new(
    nil, ENV['AZURE_APP_ID'], ENV['AZURE_APP_SECRET']
  )

  token = OAuth2::AccessToken.new(
    oauth_strategy.client, token_hash[:token],
    refresh_token: token_hash[:refresh_token]
  )

  # Refresh the tokens
  new_tokens = token.refresh!.to_hash.slice(:access_token, :refresh_token, :expires_at)

  # Rename token key
  new_tokens[:token] = new_tokens.delete :access_token

  # Store the new hash
  session[:graph_token_hash] = new_tokens
end
```

Diese Methode verwendet das [oauth2](https://github.com/oauth-xx/oauth2) -gem (eine Abhängigkeit des `omniauth-oauth2` gem), um die Tokens zu aktualisieren, und aktualisiert die Sitzung.

Legen Sie nun diese Methode zu verwenden. Machen Sie dazu den `access_token` -Accessor in der `ApplicationController` Klasse etwas intelligenter. Anstatt lediglich das Token aus der Sitzung zurückzugeben, wird zunächst überprüft, ob es in der Nähe des Ablaufs ist. Wenn dies der Fall ist, wird es vor dem Zurückgeben des Tokens aktualisiert. Ersetzen Sie die `access_token` aktuelle Methode durch Folgendes.

```ruby
def access_token
  token_hash = session[:graph_token_hash]

  # Get the expiry time - 5 minutes
  expiry = Time.at(token_hash[:expires_at] - 300)

  if Time.now > expiry
    # Token expired, refresh
    new_hash = refresh_tokens token_hash
    new_hash[:token]
  else
    token_hash[:token]
  end
end
```
