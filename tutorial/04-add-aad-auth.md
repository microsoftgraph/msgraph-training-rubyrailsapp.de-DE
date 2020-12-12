<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen. Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten. In diesem Schritt integrieren Sie das [omniauth-oauth2-](https://github.com/omniauth/omniauth-oauth2) Juwel in die Anwendung und erstellen eine benutzerdefinierte omniauth-Strategie.

1. Erstellen Sie eine separate Datei, die Ihre APP-ID und den geheimen Schlüssel aufbewahren soll. Erstellen Sie eine neue Datei `oauth_environment_variables.rb` mit dem Namen im Ordner **./config** , und fügen Sie den folgenden Code hinzu.

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. Ersetzen `YOUR_APP_ID_HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, und ersetzen `YOUR_APP_SECRET_HERE` Sie durch das Kennwort, das Sie generiert haben.

    > [!IMPORTANT]
    > Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die `oauth_environment_variables.rb` Datei aus der Quellcodeverwaltung auszuschließen, damit versehentlich keine APP-ID und Ihr Kennwort verloren gehen.

1. Öffnen Sie **/config/Environment.RB** , und fügen Sie vor der Codezeile den folgenden Code hinzu `Rails.application.initialize!` .

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a>Setup OmniAuth

Sie haben das gem bereits installiert `omniauth-oauth2` , aber damit es mit den Azure OAuth-Endpunkten funktioniert, müssen Sie [eine OAuth2-Strategie erstellen](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy). Dies ist eine Ruby-Klasse, die die Parameter für die Erstellung von OAuth-Anforderungen an den Azure-Anbieter definiert.

1. Erstellen Sie eine neue Datei `microsoft_graph_auth.rb` mit dem Namen im Ordner **./lib**' * *, und fügen Sie den folgenden Code hinzu.

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    Nehmen Sie sich einen Moment Zeit, um zu überprüfen, was dieser Code tut.

    - Sie legt die fest `client_options` , um die Microsoft Identity Platform-Endpunkte anzugeben.
    - Er gibt an, dass der `scope` Parameter während der Autorisierungsphase gesendet werden soll.
    - Die `id` Eigenschaft des Benutzers wird als eindeutige ID für den Benutzer zugeordnet.
    - Es verwendet das Zugriffstoken zum Abrufen des Benutzerprofils aus Microsoft Graph, um den Hash auszufüllen `raw_info` .
    - Die Rückruf-URL wird überschrieben, um sicherzustellen, dass Sie mit dem registrierten Rückruf im App-Registrierungs Portal übereinstimmt.

1. Erstellen Sie eine neue Datei `omniauth_graph.rb` mit dem Namen im Ordner **./config/initializers** , und fügen Sie den folgenden Code hinzu.

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    Dieser Code wird ausgeführt, wenn die APP gestartet wird. Es lädt die OmniAuth-Middleware mit dem `microsoft_graph_auth` Anbieter, der mit den Umgebungsvariablen konfiguriert ist, die in **oauth_environment_variables. RB** festgelegt sind.

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

Nachdem die OmniAuth-Middleware nun konfiguriert wurde, können Sie mit dem Hinzufügen der Anmeldung zur APP fortfahren.

1. Führen Sie den folgenden Befehl in der CLI aus, um einen Controller für die Anmeldung und Abmeldung zu generieren.

    ```Shell
    rails generate controller Auth
    ```

1. Öffnen Sie **./app/Controllers/auth_controller. RB**. Fügen Sie der-Klasse eine Rückrufmethode hinzu `AuthController` . Diese Methode wird von der OmniAuth-Middleware aufgerufen, sobald der OAuth-Fluss abgeschlossen ist.

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    Im Moment wird der von OmniAuth bereitgestellte Hash gerendert. Verwenden Sie diese, um zu überprüfen, ob die Anmeldung funktionsfähig ist, bevor Sie fortfahren.

1. Fügen Sie die Routen zu **./config/routes.RB** hinzu.

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', :to => 'auth#callback', :via => [:get, :post]
    ```

1. Starten Sie den Server, und navigieren Sie zu `https://localhost:3000` . Klicken Sie auf die Schaltfläche zum Anmelden, um zu `https://login.microsoftonline.com`weitergeleitet zu werden. Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu. Der Browser wird an die APP umgeleitet, wobei der von OmniAuth generierte Hash angezeigt wird.

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
          "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users(displayName,mail,mailboxSettings,userPrincipalName)/$entity",
          "displayName": "Lynne Robbins",
          "mail": "LynneR@contoso.OnMicrosoft.com",
          "userPrincipalName": "LynneR@contoso.OnMicrosoft.com",
          "id": "d294e784-840e-4f9f-bb1e-95c0a75f2f18@2d18179c-4386-4cbd-8891-7fd867c4f62e",
          "mailboxSettings": {
            "archiveFolder": "AAMkAGI2...",
            "timeZone": "Pacific Standard Time",
            "delegateMeetingMessageDeliveryOptions": "sendToDelegateOnly",
            "dateFormat": "M/d/yyyy",
            "timeFormat": "h:mm tt",
            "automaticRepliesSetting": {
              "status": "disabled",
              "externalAudience": "all",
              "internalReplyMessage": "",
              "externalReplyMessage": "",
              "scheduledStartDateTime": {
                "dateTime": "2020-12-09T17:00:00.0000000",
                "timeZone": "UTC"
              },
              "scheduledEndDateTime": {
                "dateTime": "2020-12-10T17:00:00.0000000",
                "timeZone": "UTC"
              }
            },
            "language": {
              "locale": "en-US",
              "displayName": "English (United States)"
            },
            "workingHours": {
              "daysOfWeek": [
                "monday",
                "tuesday",
                "wednesday",
                "thursday",
                "friday"
              ],
              "startTime": "08:00:00.0000000",
              "endTime": "17:00:00.0000000",
              "timeZone": {
                "name": "Pacific Standard Time"
              }
            }
          }
        }
      }
    }
    ```

## <a name="storing-the-tokens"></a>Speichern des Tokens

Nun, da Sie Token abrufen können, ist es an der Zeit, eine Möglichkeit einzurichten, diese in der App zu speichern. Da es sich um eine Beispiel-App handelt, speichern Sie Sie aus Gründen der Einfachheit in der Sitzung. Eine echte App würde eine zuverlässigere sichere Speicherlösung wie eine Datenbank verwenden.

1. Öffnen Sie **./app/Controllers/application_controller. RB**. Fügen Sie der Klasse `ApplicationController` die folgende Methode hinzu.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    Die-Methode verwendet den OmniAuth-Hash als Parameter und extrahiert die relevanten Informationsbits und speichert diese dann in der Sitzung.

1. Fügen Sie der Klasse Accessorfunktionen hinzu, `ApplicationController` um den Benutzernamen, die e-Mail-Adresse und das Zugriffstoken wieder aus der Sitzung abzurufen.

    ```ruby
    def user_name
      session[:user_name]
    end

    def user_email
      session[:user_email]
    end

    def user_timezone
      session[:user_timezone]
    end

    def access_token
      session[:graph_token_hash][:token]
    end
    ```

1. Fügen Sie den folgenden Code zur `ApplicationController` Klasse hinzu, die ausgeführt wird, bevor eine Aktion verarbeitet wird.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    Mit dieser Methode werden die Variablen festgelegt, die das Layout (in **application.html. Erb**) verwendet, um die Benutzerinformationen in der Navigationsleiste anzuzeigen. Wenn Sie ihn hier hinzufügen, müssen Sie diesen Code nicht in jeder einzelnen Controller Aktion hinzufügen. Dies wird jedoch auch für Aktionen in der ausgeführt `AuthController` , was nicht optimal ist.

1. Fügen Sie der `AuthController` Klasse in **./app/Controllers/auth_controller. RB** den folgenden Code hinzu, um die before-Aktion zu überspringen.

    ```ruby
    skip_before_action :set_user
    ```

1. Aktualisieren Sie die `callback` -Funktion in der `AuthController` -Klasse, um die Token in der Sitzung zu speichern und zur Hauptseite zurückzuleiten. Ersetzen Sie die vorhandene `callback`-Funktion durch Folgendes.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a>Implementieren der Abmeldung

Bevor Sie dieses neue Feature testen, fügen Sie eine Möglichkeit zum Abmelden hinzu.

1. Fügen Sie der Klasse die folgende Aktion hinzu `AuthController` .

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. Fügen Sie diese Aktion zu **./config/routes.RB**.

    ```ruby
    get 'auth/signout'
    ```

1. Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort. Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den **Abmelde** Link zuzugreifen. Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Aktualisieren von Token

Wenn Sie sich den von OmniAuth generierten Hash genauer ansehen, werden Sie feststellen, dass der Hash zwei Token enthält: `token` und `refresh_token` . Der Wert in `token` ist das Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird. Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.

Dieses Token ist jedoch nur kurzzeitig verfügbar. Das Token läuft eine Stunde nach seiner Ausgabe ab. Hier wird der `refresh_token` Wert nützlich. Anhand des Aktualisierungstoken ist die App in der Lage, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss. Aktualisieren Sie den Token-Verwaltungscode, um die Token-Aktualisierung zu implementieren.

1. Öffnen Sie **./app/Controllers/application_controller. RB** , und fügen Sie die folgenden `require` Anweisungen am oberen Rand hinzu:

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. Fügen Sie der Klasse `ApplicationController` die folgende Methode hinzu.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    Diese Methode verwendet das [oauth2](https://github.com/oauth-xx/oauth2) -gem (eine Abhängigkeit des `omniauth-oauth2` gem), um die Tokens zu aktualisieren, und aktualisiert die Sitzung.

1. Ersetzen Sie die aktuelle `access_token` Methode durch Folgendes.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    Anstatt lediglich das Token aus der Sitzung zurückzugeben, wird zunächst überprüft, ob es in der Nähe des Ablaufs ist. Wenn dies der Fall ist, wird es vor dem Zurückgeben des Tokens aktualisiert.
