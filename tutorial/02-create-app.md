<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung verwenden Sie [Ruby on Rails](https://rubyonrails.org/) , um eine Webanwendung zu erstellen. Wenn Rails nicht bereits installiert ist, können Sie es über die Befehlszeilenschnittstelle (CLI) mit dem folgenden Befehl installieren.

```Shell
gem install rails
```

Öffnen Sie die CLI, navigieren Sie zu einem Verzeichnis, in dem Sie Berechtigungen zum Erstellen von Dateien haben, und führen Sie den folgenden Befehl aus, um eine neue Rails-APP zu erstellen.

```Shell
rails new graph-tutorial
```

Rails erstellt ein neues Verzeichnis mit `graph-tutorial` dem Namen und Gerüste einer Rails-app. Navigieren Sie zu diesem neuen Verzeichnis, und geben Sie den folgenden Befehl ein, um einen lokalen Webserver zu starten.

```Shell
rails server
```

Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:3000`. Wenn alles funktioniert, wird ein "yay!" angezeigt. Sie sind auf Schienen! " Nachricht. Wenn diese Meldung nicht angezeigt wird, überprüfen Sie das [Handbuch Rails-erste Schritte](http://guides.rubyonrails.org/).

Bevor Sie fortfahren, sollten Sie einige zusätzliche Edelsteine installieren, die Sie später verwenden werden:

- [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) für die Verarbeitung von Anmelde-und OAuth-Token-Flows.
- [httparty](https://github.com/jnunemaker/httparty) zum tätigen von Anrufen an Microsoft Graph.
- [nokogiri](https://github.com/sparklemotion/nokogiri) zum Verarbeiten von HTML-Textkörpern von e-Mails.
- [ActiveRecord-session_store](https://github.com/rails/activerecord-session_store) zum Speichern von Sitzungen in der Datenbank.

Führen Sie die folgenden Befehle in der CLI aus.

```Shell
bundle add omniauth-oauth2
bundle add httparty
bundle add nokogiri
bundle add activerecord-session_store
rails generate active_record:session_migration
```

Der letzte Befehl generiert eine Ausgabe wie die folgende:

```Shell
create  db/migrate/20180618172216_add_sessions_table.rb
```

Öffnen Sie die Datei, die erstellt wurde, und suchen Sie nach der folgenden Position.

```ruby
class AddSessionsTable < ActiveRecord::Migration
```

Ändern Sie diese in die folgende Richtung.

```ruby
class AddSessionsTable < ActiveRecord::Migration[5.2]
```

> [!NOTE]
> Dabei wird davon ausgegangen, dass Sie Rails 5.2. x verwenden. Wenn Sie eine andere Version verwenden, ersetzen `5.2` Sie Sie durch Ihre Version.

Speichern Sie die Datei, und führen Sie den folgenden Befehl aus.

```Shell
rake db:migrate
```

Konfigurieren Sie schließlich Rails für die Verwendung des neuen Sitzungs Speichers. Erstellen Sie eine neue Datei `session_store.rb` mit dem `./config/initializers` Namen im Verzeichnis, und fügen Sie den folgenden Code hinzu.

```ruby
Rails.application.config.session_store :active_record_store, key: '_graph_app_session'
```

## <a name="design-the-app"></a>Entwerfen der APP

Aktualisieren Sie zunächst das globale Layout für die app. Öffnen `./app/views/layouts/application.html.erb` Sie den Inhalt, und ersetzen Sie ihn durch den folgenden Code.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Ruby Graph Tutorial</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
      integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
      integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"
      integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"
      integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container">
        <%= link_to "Ruby Graph Tutorial", root_path, class: "navbar-brand" %>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
          aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <%= link_to "Home", root_path, class: "nav-link#{' active' if controller.controller_name == 'home'}" %>
            </li>
            <% if @user_name %>
              <li class="nav-item" data-turbolinks="false">
                <a class="nav-link" href="#">Calendar</a>
              </li>
            <% end %>
          </ul>
          <ul class="navbar-nav justify-content-end">
            <li class="nav-item">
              <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                <i class="fas fa-external-link-alt mr-1"></i>Docs
              </a>
            </li>
            <% if @user_name %>
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false">
                  <% if @user_avatar %>
                    <img src=<%= @user_avatar %> class="rounded-circle align-self-center mr-2" style="width: 32px;">
                  <% else %>
                    <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                  <% end %>
                </a>
                <div class="dropdown-menu dropdown-menu-right">
                  <h5 class="dropdown-item-text mb-0"><%= @user_name %></h5>
                  <p class="dropdown-item-text text-muted mb-0"><%= @user_email %></p>
                  <div class="dropdown-divider"></div>
                  <a href="#" class="dropdown-item">Sign Out</a>
                </div>
              </li>
            <% else %>
              <li class="nav-item">
                <a href="#" class="nav-link">Sign In</a>
              </li>
            <% end %>
          </ul>
        </div>
      </div>
    </nav>
    <main role="main" class="container">
      <% if @errors %>
        <% @errors.each do |error| %>
          <div class="alert alert-danger" role="alert">
            <p class="mb-3"><%= error[:message] %></p>
            <%if error[:debug] %>
              <pre class="alert-pre border bg-light p-2"><code><%= error[:debug] %></code></pre>
            <% end %>
          </div>
        <% end %>
      <% end %>
      <%= yield %>
    </main>
  </body>
</html>
```

Dieser Code fügt [Bootstrap](http://getbootstrap.com/) für einfaches Styling und [Font awesome](https://fontawesome.com/) für einige einfache Symbole hinzu. Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.

Öffnen `./app/assets/stylesheets/application.css` Sie nun, und fügen Sie Folgendes am Ende der Datei hinzu.

```css
body {
  padding-top: 4.5rem;
}

.alert-pre {
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
}
```

Ersetzen Sie nun die Standardseite durch einen neuen. Generieren Sie mit dem folgenden Befehl einen Startseiten-Controller.

```Shell
rails generate controller Home index
```

Konfigurieren Sie dann `index` die Aktion auf `Home` dem Controller als Standardseite für die app. Öffnen `./config/routes.rb` Sie den Inhalt, und ersetzen Sie ihn durch den folgenden Code:

```ruby
Rails.application.routes.draw do
  get 'home/index'
  root 'home#index'

  # Add future routes here

end
```

Öffnen Sie nun `./app/view/home/index.html.erb` die Datei, und ersetzen Sie den Inhalt durch Folgendes.

```html
<div class="jumbotron">
  <h1>Ruby Graph Tutorial</h1>
  <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from Ruby</p>
  <% if @user_name %>
    <h4>Welcome <%= @user_name %>!</h4>
    <p>Use the navigation bar at the top of the page to get started.</p>
  <% else %>
    <a href="#" class="btn btn-primary btn-large">Click here to sign in</a>
  <% end %>
</div>
```

Speichern Sie alle Änderungen, und starten Sie den Server neu. Nun sollte die APP sehr unterschiedlich aussehen.

![Screenshot der neu gestalteten Startseite](./images/create-app-01.png)
