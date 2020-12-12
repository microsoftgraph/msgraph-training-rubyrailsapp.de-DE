<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung verwenden Sie [Ruby on Rails](https://rubyonrails.org/) , um eine Webanwendung zu erstellen.

1. Wenn Rails nicht bereits installiert ist, können Sie es über die Befehlszeilenschnittstelle (CLI) mit dem folgenden Befehl installieren.

    ```Shell
    gem install rails -v 6.0.3.4
    ```

1. Öffnen Sie die CLI, navigieren Sie zu einem Verzeichnis, in dem Sie Berechtigungen zum Erstellen von Dateien haben, und führen Sie den folgenden Befehl aus, um eine neue Rails-APP zu erstellen.

    ```Shell
    rails new graph-tutorial
    ```

1. Navigieren Sie zu diesem neuen Verzeichnis, und geben Sie den folgenden Befehl ein, um einen lokalen Webserver zu starten.

    ```Shell
    rails server
    ```

1. Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:3000`. Wenn alles funktioniert, wird ein "yay!" angezeigt. Sie sind auf Schienen! " Nachricht. Wenn diese Meldung nicht angezeigt wird, überprüfen Sie das [Handbuch Rails-erste Schritte](http://guides.rubyonrails.org/).

## <a name="install-gems"></a>Installieren von Gems

Bevor Sie fortfahren, sollten Sie einige zusätzliche Edelsteine installieren, die Sie später verwenden werden:

- [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) für die Verarbeitung von Anmelde-und OAuth-Token-Flows.
- [omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) zum Hinzufügen von CSRF-Schutz zu omniauth.
- [httparty](https://github.com/jnunemaker/httparty) zum tätigen von Anrufen an Microsoft Graph.
- [ActiveRecord-session_store](https://github.com/rails/activerecord-session_store) zum Speichern von Sitzungen in der Datenbank.

1. Öffnen Sie **/Gemfile** , und fügen Sie die folgenden Zeilen hinzu.

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. Führen Sie in der CLI den folgenden Befehl aus.

    ```Shell
    bundle install
    ```

1. Führen Sie in der CLI die folgenden Befehle aus, um die Datenbank zum Speichern von Sitzungen zu konfigurieren.

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. Erstellen Sie eine neue Datei `session_store.rb` mit dem Namen im Verzeichnis **./config/initializers** , und fügen Sie den folgenden Code hinzu.

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a>Entwerfen der App

In diesem Abschnitt erstellen Sie die grundlegende Benutzeroberfläche für die app.

1. Öffnen Sie **./app/views/Layouts/application.html. Erb** , und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    Dieser Code fügt [Bootstrap](http://getbootstrap.com/) für einfaches Styling und [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) für einige einfache Symbole hinzu. Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.

1. Öffnen Sie **/App/Assets/Stylesheets/Application.CSS** , und fügen Sie Folgendes am Ende der Datei hinzu.

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. Generieren Sie mit dem folgenden Befehl einen Startseiten-Controller.

    ```Shell
    rails generate controller Home index
    ```

1. Konfigurieren Sie die `index` Aktion auf dem `Home` Controller als Standardseite für die app. Öffnen Sie **./config/routes.RB** , und ersetzen Sie den Inhalt durch Folgendes:

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. Öffnen Sie **./app/View/Home/index.html. Erb** , und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. Fügen Sie im Verzeichnis **./app/Assets/Images** eine PNG-Datei mit dem Namen **no-profile-photo.png** hinzu.

1. Speichern Sie alle Änderungen, und starten Sie den Server neu. Nun sollte die APP sehr unterschiedlich aussehen.

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
