<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="76901-101">In dieser Übung verwenden Sie [Ruby on Rails](https://rubyonrails.org/) , um eine Webanwendung zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="76901-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span> <span data-ttu-id="76901-102">Wenn Rails nicht bereits installiert ist, können Sie es über die Befehlszeilenschnittstelle (CLI) mit dem folgenden Befehl installieren.</span><span class="sxs-lookup"><span data-stu-id="76901-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

```Shell
gem install rails
```

<span data-ttu-id="76901-103">Öffnen Sie die CLI, navigieren Sie zu einem Verzeichnis, in dem Sie Berechtigungen zum Erstellen von Dateien haben, und führen Sie den folgenden Befehl aus, um eine neue Rails-APP zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="76901-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

```Shell
rails new graph-tutorial
```

<span data-ttu-id="76901-104">Rails erstellt ein neues Verzeichnis mit `graph-tutorial` dem Namen und Gerüste einer Rails-app.</span><span class="sxs-lookup"><span data-stu-id="76901-104">Rails creates a new directory called `graph-tutorial` and scaffolds a Rails app.</span></span> <span data-ttu-id="76901-105">Navigieren Sie zu diesem neuen Verzeichnis, und geben Sie den folgenden Befehl ein, um einen lokalen Webserver zu starten.</span><span class="sxs-lookup"><span data-stu-id="76901-105">Navigate to this new directory and enter the following command to start a local web server.</span></span>

```Shell
rails server
```

<span data-ttu-id="76901-106">Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="76901-106">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="76901-107">Wenn alles funktioniert, wird ein "yay!" angezeigt.</span><span class="sxs-lookup"><span data-stu-id="76901-107">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="76901-108">Sie sind auf Schienen! "</span><span class="sxs-lookup"><span data-stu-id="76901-108">You're on Rails!"</span></span> <span data-ttu-id="76901-109">Nachricht.</span><span class="sxs-lookup"><span data-stu-id="76901-109">message.</span></span> <span data-ttu-id="76901-110">Wenn diese Meldung nicht angezeigt wird, überprüfen Sie das [Handbuch Rails-erste Schritte](http://guides.rubyonrails.org/).</span><span class="sxs-lookup"><span data-stu-id="76901-110">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

<span data-ttu-id="76901-111">Bevor Sie fortfahren, sollten Sie einige zusätzliche Edelsteine installieren, die Sie später verwenden werden:</span><span class="sxs-lookup"><span data-stu-id="76901-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="76901-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) für die Verarbeitung von Anmelde-und OAuth-Token-Flows.</span><span class="sxs-lookup"><span data-stu-id="76901-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="76901-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) zum Hinzufügen von CSRF-Schutz zu omniauth.</span><span class="sxs-lookup"><span data-stu-id="76901-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="76901-114">[httparty](https://github.com/jnunemaker/httparty) zum tätigen von Anrufen an Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="76901-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="76901-115">[nokogiri](https://github.com/sparklemotion/nokogiri) zum Verarbeiten von HTML-Textkörpern von e-Mails.</span><span class="sxs-lookup"><span data-stu-id="76901-115">[nokogiri](https://github.com/sparklemotion/nokogiri) to process HTML bodies of email.</span></span>
- <span data-ttu-id="76901-116">[ActiveRecord-session_store](https://github.com/rails/activerecord-session_store) zum Speichern von Sitzungen in der Datenbank.</span><span class="sxs-lookup"><span data-stu-id="76901-116">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

<span data-ttu-id="76901-117">Führen Sie die folgenden Befehle in der CLI aus.</span><span class="sxs-lookup"><span data-stu-id="76901-117">Run the following commands in your CLI.</span></span>

```Shell
bundle add omniauth-oauth2
bundle add omniauth-rails_csrf_protection
bundle add httparty
bundle add nokogiri
bundle add activerecord-session_store
rails generate active_record:session_migration
```

<span data-ttu-id="76901-118">Der letzte Befehl generiert eine Ausgabe wie die folgende:</span><span class="sxs-lookup"><span data-stu-id="76901-118">The last command generates output like the following:</span></span>

```Shell
create  db/migrate/20180618172216_add_sessions_table.rb
```

<span data-ttu-id="76901-119">Öffnen Sie die Datei, die erstellt wurde, und suchen Sie nach der folgenden Position.</span><span class="sxs-lookup"><span data-stu-id="76901-119">Open the file that was created and locate the following line.</span></span>

```ruby
class AddSessionsTable < ActiveRecord::Migration
```

<span data-ttu-id="76901-120">Ändern Sie diese in die folgende Richtung.</span><span class="sxs-lookup"><span data-stu-id="76901-120">Change that line to the following.</span></span>

```ruby
class AddSessionsTable < ActiveRecord::Migration[5.2]
```

> [!NOTE]
> <span data-ttu-id="76901-121">Dabei wird davon ausgegangen, dass Sie Rails 5.2. x verwenden.</span><span class="sxs-lookup"><span data-stu-id="76901-121">This assumes that you are using Rails 5.2.x.</span></span> <span data-ttu-id="76901-122">Wenn Sie eine andere Version verwenden, ersetzen `5.2` Sie Sie durch Ihre Version.</span><span class="sxs-lookup"><span data-stu-id="76901-122">If you are using a different version, replace `5.2` with your version.</span></span>

<span data-ttu-id="76901-123">Speichern Sie die Datei, und führen Sie den folgenden Befehl aus.</span><span class="sxs-lookup"><span data-stu-id="76901-123">Save the file and run the following command.</span></span>

```Shell
rake db:migrate
```

<span data-ttu-id="76901-124">Konfigurieren Sie schließlich Rails für die Verwendung des neuen Sitzungs Speichers.</span><span class="sxs-lookup"><span data-stu-id="76901-124">Finally, configure Rails to use the new session store.</span></span> <span data-ttu-id="76901-125">Erstellen Sie eine neue Datei `session_store.rb` mit dem `./config/initializers` Namen im Verzeichnis, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="76901-125">Create a new file called `session_store.rb` in the `./config/initializers` directory, and add the following code.</span></span>

```ruby
Rails.application.config.session_store :active_record_store, key: '_graph_app_session'
```

## <a name="design-the-app"></a><span data-ttu-id="76901-126">Entwerfen der APP</span><span class="sxs-lookup"><span data-stu-id="76901-126">Design the app</span></span>

<span data-ttu-id="76901-127">Aktualisieren Sie zunächst das globale Layout für die app.</span><span class="sxs-lookup"><span data-stu-id="76901-127">Start by updating the global layout for the app.</span></span> <span data-ttu-id="76901-128">Öffnen `./app/views/layouts/application.html.erb` Sie den Inhalt, und ersetzen Sie ihn durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="76901-128">Open `./app/views/layouts/application.html.erb` and replace its contents with the following.</span></span>

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

<span data-ttu-id="76901-129">Dieser Code fügt [Bootstrap](http://getbootstrap.com/) für einfaches Styling und [Font awesome](https://fontawesome.com/) für einige einfache Symbole hinzu.</span><span class="sxs-lookup"><span data-stu-id="76901-129">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="76901-130">Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.</span><span class="sxs-lookup"><span data-stu-id="76901-130">It also defines a global layout with a nav bar.</span></span>

<span data-ttu-id="76901-131">Öffnen `./app/assets/stylesheets/application.css` Sie nun, und fügen Sie Folgendes am Ende der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="76901-131">Now open `./app/assets/stylesheets/application.css` and add the following to the end of the file.</span></span>

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

<span data-ttu-id="76901-132">Ersetzen Sie nun die Standardseite durch einen neuen.</span><span class="sxs-lookup"><span data-stu-id="76901-132">Now replace the default page with a new one.</span></span> <span data-ttu-id="76901-133">Generieren Sie mit dem folgenden Befehl einen Startseiten-Controller.</span><span class="sxs-lookup"><span data-stu-id="76901-133">Generate a home page controller with the following command.</span></span>

```Shell
rails generate controller Home index
```

<span data-ttu-id="76901-134">Konfigurieren Sie dann `index` die Aktion auf `Home` dem Controller als Standardseite für die app.</span><span class="sxs-lookup"><span data-stu-id="76901-134">Then configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="76901-135">Öffnen `./config/routes.rb` Sie den Inhalt, und ersetzen Sie ihn durch den folgenden Code:</span><span class="sxs-lookup"><span data-stu-id="76901-135">Open `./config/routes.rb` and replace the contents with the following</span></span>

```ruby
Rails.application.routes.draw do
  get 'home/index'
  root 'home#index'

  # Add future routes here

end
```

<span data-ttu-id="76901-136">Öffnen Sie nun `./app/view/home/index.html.erb` die Datei, und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="76901-136">Now open the `./app/view/home/index.html.erb` file and replace its contents with the following.</span></span>

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

<span data-ttu-id="76901-137">Speichern Sie alle Änderungen, und starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="76901-137">Save all of your changes and restart the server.</span></span> <span data-ttu-id="76901-138">Nun sollte die APP sehr unterschiedlich aussehen.</span><span class="sxs-lookup"><span data-stu-id="76901-138">Now, the app should look very different.</span></span>

![Screenshot der neu gestalteten Startseite](./images/create-app-01.png)
