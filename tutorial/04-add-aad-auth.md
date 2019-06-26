<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="99934-101">In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen.</span><span class="sxs-lookup"><span data-stu-id="99934-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="99934-102">Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="99934-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="99934-103">In diesem Schritt integrieren Sie das [omniauth-oauth2-](https://github.com/omniauth/omniauth-oauth2) Juwel in die Anwendung und erstellen eine benutzerdefinierte omniauth-Strategie.</span><span class="sxs-lookup"><span data-stu-id="99934-103">In this step you will integrate the [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem into the application, and create a custom OmniAuth strategy.</span></span>

<span data-ttu-id="99934-104">Erstellen Sie zunächst eine separate Datei, in der die APP-ID und der Schlüssel gespeichert sind.</span><span class="sxs-lookup"><span data-stu-id="99934-104">First, create a separate file to hold your app ID and secret.</span></span> <span data-ttu-id="99934-105">Erstellen Sie eine neue Datei `oauth_environment_variables.rb` mit dem `./config` Namen im Ordner, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="99934-105">Create a new file called `oauth_environment_variables.rb` in the `./config` folder, and add the following code.</span></span>

```ruby
ENV['AZURE_APP_ID'] = 'YOUR_APP_ID_HERE'
ENV['AZURE_APP_SECRET'] = 'YOUR_APP_SECRET_HERE'
ENV['AZURE_SCOPES'] = 'openid profile email offline_access user.read calendars.read'
```

<span data-ttu-id="99934-106">Ersetzen `YOUR_APP_ID_HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR_APP_SECRET_HERE` und ersetzen Sie durch das Kennwort, das Sie generiert haben.</span><span class="sxs-lookup"><span data-stu-id="99934-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="99934-107">Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die Datei `oauth_environment_variables.rb` aus der Quellcodeverwaltung auszuschließen, damit versehentlich keine APP-ID und Ihr Kennwort verloren gehen.</span><span class="sxs-lookup"><span data-stu-id="99934-107">If you're using source control such as git, now would be a good time to exclude the `oauth_environment_variables.rb` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

<span data-ttu-id="99934-108">Fügen Sie jetzt Code hinzu, um diese Datei zu laden, falls Sie vorhanden ist.</span><span class="sxs-lookup"><span data-stu-id="99934-108">Now add code to load this file if it's present.</span></span> <span data-ttu-id="99934-109">Öffnen Sie `./config/environment.rb` die Datei, und fügen Sie den folgenden `Rails.application.initialize!` Code vor der Codezeile hinzu.</span><span class="sxs-lookup"><span data-stu-id="99934-109">Open the `./config/environment.rb` file and add the following code before the `Rails.application.initialize!` line.</span></span>

```ruby
# Load OAuth settings
oauth_environment_variables = File.join(Rails.root, 'config', 'oauth_environment_variables.rb')
load(oauth_environment_variables) if File.exist?(oauth_environment_variables)
```

## <a name="setup-omniauth"></a><span data-ttu-id="99934-110">Setup OmniAuth</span><span class="sxs-lookup"><span data-stu-id="99934-110">Setup OmniAuth</span></span>

<span data-ttu-id="99934-111">Sie haben das `omniauth-oauth2` gem bereits installiert, aber damit es mit den Azure OAuth-Endpunkten funktioniert, müssen Sie [eine OAuth2-Strategie erstellen](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span><span class="sxs-lookup"><span data-stu-id="99934-111">You've already installed the `omniauth-oauth2` gem, but in order to make it work with the Azure OAuth endpoints, you need to [create an OAuth2 strategy](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span></span> <span data-ttu-id="99934-112">Dies ist eine Ruby-Klasse, die die Parameter für die Erstellung von OAuth-Anforderungen an den Azure-Anbieter definiert.</span><span class="sxs-lookup"><span data-stu-id="99934-112">This is a Ruby class that defines the parameters for making OAuth requests to the Azure provider.</span></span>

<span data-ttu-id="99934-113">Erstellen Sie eine neue Datei `microsoft_graph_auth.rb` mit dem `./lib` Namen im Ordner, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="99934-113">Create a new file called `microsoft_graph_auth.rb` in the `./lib` folder, and add the following code.</span></span>

```ruby
require 'omniauth-oauth2'

module OmniAuth
  module Strategies
    # Implements an OmniAuth strategy to get a Microsoft Graph
    # compatible token from Azure AD
    class MicrosoftGraphAuth < OmniAuth::Strategies::OAuth2
      option :name, :microsoft_graph_auth

      DEFAULT_SCOPE = 'openid email profile User.Read'.freeze

      # Configure the Azure v2 endpoints
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

<span data-ttu-id="99934-114">Nehmen Sie sich einen Moment Zeit, um zu überprüfen, was dieser Code tut.</span><span class="sxs-lookup"><span data-stu-id="99934-114">Take a moment to review what this code does.</span></span>

- <span data-ttu-id="99934-115">Sie legt die `client_options` fest, um die Azure v2-Endpunkte anzugeben.</span><span class="sxs-lookup"><span data-stu-id="99934-115">It sets the `client_options` to specify the Azure v2 endpoints.</span></span>
- <span data-ttu-id="99934-116">Er gibt an, `scope` dass der Parameter während der Autorisierungsphase gesendet werden soll.</span><span class="sxs-lookup"><span data-stu-id="99934-116">It specifies that the `scope` parameter should be sent during the authorize phase.</span></span>
- <span data-ttu-id="99934-117">Die `id` Eigenschaft des Benutzers wird als eindeutige ID für den Benutzer zugeordnet.</span><span class="sxs-lookup"><span data-stu-id="99934-117">It maps the `id` property of the user as the unique ID for the user.</span></span>
- <span data-ttu-id="99934-118">Es verwendet das Zugriffstoken zum Abrufen des Benutzerprofils aus Microsoft Graph, um den `raw_info` Hash auszufüllen.</span><span class="sxs-lookup"><span data-stu-id="99934-118">It uses the access token to retrieve the user's profile from Microsoft Graph to fill in the `raw_info` hash.</span></span>
- <span data-ttu-id="99934-119">Die Rückruf-URL wird überschrieben, um sicherzustellen, dass Sie mit dem registrierten Rückruf im App-Registrierungs Portal übereinstimmt.</span><span class="sxs-lookup"><span data-stu-id="99934-119">It overrides the callback URL to ensure that it matches the registered callback in the app registration portal.</span></span>

<span data-ttu-id="99934-120">Nachdem wir die Strategie definiert haben, müssen wir OmniAuth so konfigurieren, dass Sie verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="99934-120">Now that we've defined the strategy, we need to configure OmniAuth to use it.</span></span> <span data-ttu-id="99934-121">Erstellen Sie eine neue Datei `omniauth_graph.rb` mit dem `./config/initializers` Namen im Ordner, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="99934-121">Create a new file called `omniauth_graph.rb` in the `./config/initializers` folder, and add the following code.</span></span>

```ruby
require 'microsoft_graph_auth'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :microsoft_graph_auth,
           ENV['AZURE_APP_ID'],
           ENV['AZURE_APP_SECRET'],
           scope: ENV['AZURE_SCOPES']
end
```

<span data-ttu-id="99934-122">Dieser Code wird ausgeführt, wenn die APP gestartet wird.</span><span class="sxs-lookup"><span data-stu-id="99934-122">This code will execute when the app starts.</span></span> <span data-ttu-id="99934-123">Es lädt die OmniAuth-Middleware mit dem `microsoft_graph_auth` Anbieter, der mit den Umgebungsvariablen in `oauth_environment_variables.rb`festgelegt konfiguriert ist.</span><span class="sxs-lookup"><span data-stu-id="99934-123">It loads up the OmniAuth middleware with the `microsoft_graph_auth` provider, configured with the environment variables set in `oauth_environment_variables.rb`.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="99934-124">Implementieren der Anmeldung</span><span class="sxs-lookup"><span data-stu-id="99934-124">Implement sign-in</span></span>

<span data-ttu-id="99934-125">Nachdem die OmniAuth-Middleware nun konfiguriert wurde, können Sie mit dem Hinzufügen der Anmeldung zur APP fortfahren.</span><span class="sxs-lookup"><span data-stu-id="99934-125">Now that the OmniAuth middleware is configured, you can move on to adding sign-in to the app.</span></span> <span data-ttu-id="99934-126">Führen Sie den folgenden Befehl in der CLI aus, um einen Controller für die Anmeldung und Abmeldung zu generieren.</span><span class="sxs-lookup"><span data-stu-id="99934-126">Run the following command in your CLI to generate a controller for sign-in and sign-out.</span></span>

```Shell
rails generate controller Auth
```

<span data-ttu-id="99934-127">Öffnen Sie die Datei `./app/controllers/auth_controller.rb`.</span><span class="sxs-lookup"><span data-stu-id="99934-127">Open the `./app/controllers/auth_controller.rb` file.</span></span> <span data-ttu-id="99934-128">Fügen Sie der Klasse `AuthController` die folgende Methode hinzu.</span><span class="sxs-lookup"><span data-stu-id="99934-128">Add the following method to the `AuthController` class.</span></span>

```ruby
def signin
  redirect_to '/auth/microsoft_graph_auth'
end
```

<span data-ttu-id="99934-129">All diese Methode wird auf die Route umgeleitet, die von OmniAuth erwartet wird, um unsere benutzerdefinierte Strategie aufzurufen.</span><span class="sxs-lookup"><span data-stu-id="99934-129">All this method does is redirect to the route that OmniAuth expects to invoke our custom strategy.</span></span>

<span data-ttu-id="99934-130">Fügen Sie der `AuthController` -Klasse als nächstes eine Rückrufmethode hinzu.</span><span class="sxs-lookup"><span data-stu-id="99934-130">Next, add a callback method to the `AuthController` class.</span></span> <span data-ttu-id="99934-131">Diese Methode wird von der OmniAuth-Middleware aufgerufen, sobald der OAuth-Fluss abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="99934-131">This method will be called by the OmniAuth middleware once the OAuth flow is complete.</span></span>

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Temporary for testing!
  render json: data.to_json
end
```

<span data-ttu-id="99934-132">Im Moment wird der von OmniAuth bereitgestellte Hash gerendert.</span><span class="sxs-lookup"><span data-stu-id="99934-132">For now all this does is render the hash provided by OmniAuth.</span></span> <span data-ttu-id="99934-133">Wir verwenden dies, um zu überprüfen, ob unsere Anmeldung funktionsfähig ist, bevor Sie fortfahren.</span><span class="sxs-lookup"><span data-stu-id="99934-133">We'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="99934-134">Bevor wir testen, müssen wir die Routen zu `./config/routes.rb`hinzufügen.</span><span class="sxs-lookup"><span data-stu-id="99934-134">Before we test, we need to add the routes to `./config/routes.rb`.</span></span>

```ruby
get 'auth/signin'

# Add route for OmniAuth callback
match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
```

<span data-ttu-id="99934-135">Aktualisieren Sie nun die Ansichten, um `signin` die Aktion zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="99934-135">Now update the views to use the `signin` action.</span></span> <span data-ttu-id="99934-136">Öffnen `./app/views/layouts/application.html.erb`Sie.</span><span class="sxs-lookup"><span data-stu-id="99934-136">Open `./app/views/layouts/application.html.erb`.</span></span> <span data-ttu-id="99934-137">Ersetzen Sie die `<a href="#" class="nav-link">Sign In</a>` -Verbindung durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="99934-137">Replace the line `<a href="#" class="nav-link">Sign In</a>` with the following.</span></span>

```html
<%= link_to "Sign In", {:controller => :auth, :action => :signin}, :class => "nav-link" %>
```

<span data-ttu-id="99934-138">Öffnen Sie `./app/views/home/index.html.erb` die Datei, und `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` ersetzen Sie die folgende.</span><span class="sxs-lookup"><span data-stu-id="99934-138">Open the `./app/views/home/index.html.erb` file and replace the `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` line with the following.</span></span>

```html
<%= link_to "Click here to sign in", {:controller => :auth, :action => :signin}, :class => "btn btn-primary btn-large" %>
```

<span data-ttu-id="99934-139">Starten Sie den Server, und `https://localhost:3000`navigieren Sie zu.</span><span class="sxs-lookup"><span data-stu-id="99934-139">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="99934-140">Klicken Sie auf die Anmeldeschaltfläche, und Sie sollten zu `https://login.microsoftonline.com`umgeleitet werden.</span><span class="sxs-lookup"><span data-stu-id="99934-140">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="99934-141">Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu.</span><span class="sxs-lookup"><span data-stu-id="99934-141">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="99934-142">Der Browser wird an die APP umgeleitet, wobei der von OmniAuth generierte Hash angezeigt wird.</span><span class="sxs-lookup"><span data-stu-id="99934-142">The browser redirects to the app, showing the hash generated by OmniAuth.</span></span>

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

## <a name="storing-the-tokens"></a><span data-ttu-id="99934-143">Speichern der Token</span><span class="sxs-lookup"><span data-stu-id="99934-143">Storing the tokens</span></span>

<span data-ttu-id="99934-144">Nachdem Sie nun Token erhalten können, ist es an der Zeit, eine Möglichkeit zum Speichern in der APP zu implementieren.</span><span class="sxs-lookup"><span data-stu-id="99934-144">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="99934-145">Da es sich um eine Beispiel-App handelt, speichern Sie Sie aus Gründen der Einfachheit in der Sitzung.</span><span class="sxs-lookup"><span data-stu-id="99934-145">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="99934-146">Eine reale APP würde eine zuverlässigere sichere Speicherlösung wie eine Datenbank verwenden.</span><span class="sxs-lookup"><span data-stu-id="99934-146">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="99934-147">Öffnen Sie die Datei `./app/controllers/application_controller.rb`.</span><span class="sxs-lookup"><span data-stu-id="99934-147">Open the `./app/controllers/application_controller.rb` file.</span></span> <span data-ttu-id="99934-148">Hier werden alle unsere Methoden zur Tokenverwaltung hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="99934-148">You'll add all of our token management methods here.</span></span> <span data-ttu-id="99934-149">Da alle anderen Controller die `ApplicationController` Klasse erben, können Sie diese Methoden verwenden, um auf die Token zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="99934-149">Because all of the other controllers inherit the `ApplicationController` class, they'll be able to use these methods to access the tokens.</span></span>

<span data-ttu-id="99934-150">Fügen Sie der Klasse `ApplicationController` die folgende Methode hinzu.</span><span class="sxs-lookup"><span data-stu-id="99934-150">Add the following method to the `ApplicationController` class.</span></span> <span data-ttu-id="99934-151">Die-Methode verwendet den OmniAuth-Hash als Parameter und extrahiert die relevanten Informationsbits und speichert diese dann in der Sitzung.</span><span class="sxs-lookup"><span data-stu-id="99934-151">The method takes the OmniAuth hash as a parameter and extracts the relevant bits of information, then stores that in the session.</span></span>

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

<span data-ttu-id="99934-152">Fügen Sie nun Accessorfunktionen hinzu, um den Benutzernamen, die e-Mail-Adresse und das Zugriffstoken wieder aus der Sitzung abzurufen.</span><span class="sxs-lookup"><span data-stu-id="99934-152">Now add accessor functions to retrieve the user name, email address, and access token back out of the session.</span></span>

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

<span data-ttu-id="99934-153">Fügen Sie schließlich Code hinzu, der ausgeführt wird, bevor eine Aktion verarbeitet wird.</span><span class="sxs-lookup"><span data-stu-id="99934-153">Finally, add some code that will run before any action is processed.</span></span>

```ruby
before_action :set_user

def set_user
  @user_name = user_name
  @user_email = user_email
end
```

<span data-ttu-id="99934-154">Mit dieser Methode werden die Variablen festgelegt, die `application.html.erb`das Layout (in) verwendet, um die Benutzerinformationen in der Navigationsleiste anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="99934-154">This method sets the variables that the layout (in `application.html.erb`) uses to show the user's information in the nav bar.</span></span> <span data-ttu-id="99934-155">Wenn Sie ihn hier hinzufügen, müssen Sie diesen Code nicht in jeder einzelnen Controller Aktion hinzufügen.</span><span class="sxs-lookup"><span data-stu-id="99934-155">By adding it here, you don't have to add this code in every single controller action.</span></span> <span data-ttu-id="99934-156">Dies wird jedoch auch für Aktionen in der `AuthController`ausgeführt, was nicht optimal ist.</span><span class="sxs-lookup"><span data-stu-id="99934-156">However, this will also run for actions in the `AuthController`, which isn't optimal.</span></span> <span data-ttu-id="99934-157">Fügen Sie der `AuthController` Klasse in `./app/controllers/auth_controller.rb` den folgenden Code hinzu, um die before-Aktion zu überspringen.</span><span class="sxs-lookup"><span data-stu-id="99934-157">Add the following code to the `AuthController` class in `./app/controllers/auth_controller.rb` to skip the before action.</span></span>

```ruby
skip_before_action :set_user
```

<span data-ttu-id="99934-158">Aktualisieren Sie dann die `callback` -Funktion in `AuthController` der-Klasse, um die Token in der Sitzung zu speichern und zur Hauptseite zurückzuleiten.</span><span class="sxs-lookup"><span data-stu-id="99934-158">Then, update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="99934-159">Ersetzen Sie die vorhandene `callback`-Funktion durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="99934-159">Replace the existing `callback` function with the following.</span></span>

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Save the data in the session
  save_in_session data

  redirect_to root_url
end
```

## <a name="implement-sign-out"></a><span data-ttu-id="99934-160">Implementieren der Abmeldung</span><span class="sxs-lookup"><span data-stu-id="99934-160">Implement sign-out</span></span>

<span data-ttu-id="99934-161">Bevor Sie dieses neue Feature testen, fügen Sie eine Möglichkeit zum Abmelden hinzu. Fügen Sie der `AuthController` Klasse die folgende Aktion hinzu.</span><span class="sxs-lookup"><span data-stu-id="99934-161">Before you test this new feature, add a way to sign out. Add the following action to the `AuthController` class.</span></span>

```ruby
def signout
  reset_session
  redirect_to root_url
end
```

<span data-ttu-id="99934-162">Fügen Sie diese Aktion `./config/routes.rb`zu hinzu.</span><span class="sxs-lookup"><span data-stu-id="99934-162">Add this action to `./config/routes.rb`.</span></span>

```ruby
get 'auth/signout'
```

<span data-ttu-id="99934-163">Aktualisieren Sie nun die Ansicht, um `signout` die Aktion zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="99934-163">Now update the view to use the `signout` action.</span></span> <span data-ttu-id="99934-164">Öffnen `./app/views/layouts/application.html.erb`Sie.</span><span class="sxs-lookup"><span data-stu-id="99934-164">Open `./app/views/layouts/application.html.erb`.</span></span> <span data-ttu-id="99934-165">Ersetzen Sie die `<a href="#" class="dropdown-item">Sign Out</a>` -Verbindung durch Folgendes:</span><span class="sxs-lookup"><span data-stu-id="99934-165">Replace the line `<a href="#" class="dropdown-item">Sign Out</a>` with:</span></span>

```html
<%= link_to "Sign Out", {:controller => :auth, :action => :signout}, :class => "dropdown-item" %>
```

<span data-ttu-id="99934-166">Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort.</span><span class="sxs-lookup"><span data-stu-id="99934-166">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="99934-167">Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.</span><span class="sxs-lookup"><span data-stu-id="99934-167">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![Ein Screenshot der Startseite nach der Anmeldung](./images/add-aad-auth-01.png)

<span data-ttu-id="99934-169">Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers \*\*\*\* , um auf den Abmeldelink zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="99934-169">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="99934-170">Durch \*\*\*\* klicken auf Abmelden wird die Sitzung zurückgesetzt, und Sie kehren zur Startseite zurück.</span><span class="sxs-lookup"><span data-stu-id="99934-170">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![Screenshot des Dropdownmenüs mit dem Link zum Abmelden](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="99934-172">Aktualisieren von Token</span><span class="sxs-lookup"><span data-stu-id="99934-172">Refreshing tokens</span></span>

<span data-ttu-id="99934-173">Wenn Sie sich den von OmniAuth generierten Hash genauer ansehen, werden Sie feststellen, dass der Hash zwei Token enthält `token` : `refresh_token`und.</span><span class="sxs-lookup"><span data-stu-id="99934-173">If you look closely at the hash generated by OmniAuth, you'll notice there are two tokens in the hash: `token` and `refresh_token`.</span></span> <span data-ttu-id="99934-174">Der Wert in `token` ist das Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="99934-174">The value in `token` is the access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="99934-175">Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="99934-175">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="99934-176">Dieses Token ist jedoch nur kurzlebig.</span><span class="sxs-lookup"><span data-stu-id="99934-176">However, this token is short-lived.</span></span> <span data-ttu-id="99934-177">Das Token läuft eine Stunde nach seiner Ausgabe ab.</span><span class="sxs-lookup"><span data-stu-id="99934-177">The token expires an hour after it is issued.</span></span> <span data-ttu-id="99934-178">Hier wird der `refresh_token` Wert nützlich.</span><span class="sxs-lookup"><span data-stu-id="99934-178">This is where the `refresh_token` value becomes useful.</span></span> <span data-ttu-id="99934-179">Das Aktualisierungstoken ermöglicht der APP, ein neues Zugriffstoken anzufordern, ohne dass sich der Benutzer erneut anmelden muss.</span><span class="sxs-lookup"><span data-stu-id="99934-179">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="99934-180">Aktualisieren Sie den Token-Verwaltungscode, um die Token-Aktualisierung zu implementieren.</span><span class="sxs-lookup"><span data-stu-id="99934-180">Update the token management code to implement token refresh.</span></span>

<span data-ttu-id="99934-181">Öffnen `./app/controllers/application_controller.rb` Sie und fügen Sie `require` die folgenden Anweisungen am oberen Rand hinzu:</span><span class="sxs-lookup"><span data-stu-id="99934-181">Open `./app/controllers/application_controller.rb` and add the following `require` statements at the top:</span></span>

```ruby
require 'microsoft_graph_auth'
require 'oauth2'
```

<span data-ttu-id="99934-182">Fügen Sie dann die folgende Methode zur `ApplicationController` -Klasse hinzu.</span><span class="sxs-lookup"><span data-stu-id="99934-182">Then add the following method to the `ApplicationController` class.</span></span>

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

<span data-ttu-id="99934-183">Diese Methode verwendet das [oauth2](https://github.com/oauth-xx/oauth2) -gem (eine Abhängigkeit des `omniauth-oauth2` gem), um die Tokens zu aktualisieren, und aktualisiert die Sitzung.</span><span class="sxs-lookup"><span data-stu-id="99934-183">This method uses the [oauth2](https://github.com/oauth-xx/oauth2) gem (a dependency of the `omniauth-oauth2` gem) to refresh the tokens, and updates the session.</span></span>

<span data-ttu-id="99934-184">Legen Sie nun diese Methode zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="99934-184">Now put this method to use.</span></span> <span data-ttu-id="99934-185">Machen Sie dazu den `access_token` -Accessor in der `ApplicationController` Klasse etwas intelligenter.</span><span class="sxs-lookup"><span data-stu-id="99934-185">To do that, make the `access_token` accessor in the `ApplicationController` class a bit smarter.</span></span> <span data-ttu-id="99934-186">Anstatt lediglich das Token aus der Sitzung zurückzugeben, wird zunächst überprüft, ob es in der Nähe des Ablaufs ist.</span><span class="sxs-lookup"><span data-stu-id="99934-186">Instead of just returning the token from the session, it will first check if it is close to expiration.</span></span> <span data-ttu-id="99934-187">Wenn dies der Fall ist, wird es vor dem Zurückgeben des Tokens aktualisiert.</span><span class="sxs-lookup"><span data-stu-id="99934-187">If it is, then it will refresh before returning the token.</span></span> <span data-ttu-id="99934-188">Ersetzen Sie die `access_token` aktuelle Methode durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="99934-188">Replace the current `access_token` method with the following.</span></span>

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
