<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="0d827-101">In dieser Übung verwenden Sie [Ruby on Rails, um](https://rubyonrails.org/) eine Web-App zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="0d827-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span>

1. <span data-ttu-id="0d827-102">Wenn Sie Rails noch nicht installiert haben, können Sie es über die Befehlszeilenschnittstelle (CLI) mit dem folgenden Befehl installieren.</span><span class="sxs-lookup"><span data-stu-id="0d827-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    gem install rails -v 6.1.3.1
    ```

1. <span data-ttu-id="0d827-103">Öffnen Sie Ihre CLI, navigieren Sie zu einem Verzeichnis, in dem Sie über die Rechte zum Erstellen von Dateien verfügen, und führen Sie den folgenden Befehl aus, um eine neue Rails-App zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="0d827-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

    ```Shell
    rails new graph-tutorial
    ```

1. <span data-ttu-id="0d827-104">Navigieren Sie zu diesem neuen Verzeichnis, und geben Sie den folgenden Befehl ein, um einen lokalen Webserver zu starten.</span><span class="sxs-lookup"><span data-stu-id="0d827-104">Navigate to this new directory and enter the following command to start a local web server.</span></span>

    ```Shell
    rails server
    ```

1. <span data-ttu-id="0d827-105">Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="0d827-105">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="0d827-106">Wenn alles funktioniert, wird ein "Yay!</span><span class="sxs-lookup"><span data-stu-id="0d827-106">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="0d827-107">You're on Rails!"</span><span class="sxs-lookup"><span data-stu-id="0d827-107">You're on Rails!"</span></span> <span data-ttu-id="0d827-108">Nachricht.</span><span class="sxs-lookup"><span data-stu-id="0d827-108">message.</span></span> <span data-ttu-id="0d827-109">Wenn diese Meldung nicht angezeigt wird, überprüfen Sie die [Anleitung "Rails getting started".](http://guides.rubyonrails.org/)</span><span class="sxs-lookup"><span data-stu-id="0d827-109">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

## <a name="install-gems"></a><span data-ttu-id="0d827-110">Installieren von Gems</span><span class="sxs-lookup"><span data-stu-id="0d827-110">Install gems</span></span>

<span data-ttu-id="0d827-111">Installieren Sie vor dem Wechsel einige zusätzliche Gems, die Sie später verwenden werden:</span><span class="sxs-lookup"><span data-stu-id="0d827-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="0d827-112">[omniauth-oauth2 für](https://github.com/omniauth/omniauth-oauth2) die Verarbeitung von Anmelde- und OAuth-Tokenflüssen.</span><span class="sxs-lookup"><span data-stu-id="0d827-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="0d827-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) zum Hinzufügen von CSRF-Schutz zu OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="0d827-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="0d827-114">[httparty](https://github.com/jnunemaker/httparty) für Anrufe an Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="0d827-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="0d827-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) zum Speichern von Sitzungen in der Datenbank.</span><span class="sxs-lookup"><span data-stu-id="0d827-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

1. <span data-ttu-id="0d827-116">Öffnen **Sie ./Gemfile,** und fügen Sie die folgenden Zeilen hinzu.</span><span class="sxs-lookup"><span data-stu-id="0d827-116">Open **./Gemfile** and add the following lines.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. <span data-ttu-id="0d827-117">Führen Sie in Ihrer CLI den folgenden Befehl aus.</span><span class="sxs-lookup"><span data-stu-id="0d827-117">In your CLI, run the following command.</span></span>

    ```Shell
    bundle install
    ```

1. <span data-ttu-id="0d827-118">Führen Sie in Der CLI die folgenden Befehle aus, um die Datenbank zum Speichern von Sitzungen zu konfigurieren.</span><span class="sxs-lookup"><span data-stu-id="0d827-118">In your CLI, run the following commands to configure the database for storing sessions.</span></span>

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. <span data-ttu-id="0d827-119">Erstellen Sie eine neue Datei, die `session_store.rb` im **Verzeichnis ./config/initializers** aufgerufen wird, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="0d827-119">Create a new file called `session_store.rb` in the **./config/initializers** directory, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a><span data-ttu-id="0d827-120">Entwerfen der App</span><span class="sxs-lookup"><span data-stu-id="0d827-120">Design the app</span></span>

<span data-ttu-id="0d827-121">In diesem Abschnitt erstellen Sie die grundlegende Benutzeroberfläche für die App.</span><span class="sxs-lookup"><span data-stu-id="0d827-121">In this section you'll create the basic UI for the app.</span></span>

1. <span data-ttu-id="0d827-122">Öffnen **Sie ./app/views/layouts/application.html.erb,** und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="0d827-122">Open **./app/views/layouts/application.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    <span data-ttu-id="0d827-123">Dieser Code fügt [Bootstrap für](http://getbootstrap.com/) einfache Formatierung und [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) für einige einfache Symbole hinzu.</span><span class="sxs-lookup"><span data-stu-id="0d827-123">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) for some simple icons.</span></span> <span data-ttu-id="0d827-124">Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.</span><span class="sxs-lookup"><span data-stu-id="0d827-124">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="0d827-125">Öffnen **Sie ./app/assets/stylesheets/application.css,** und fügen Sie am Ende der Datei Folgendes hinzu.</span><span class="sxs-lookup"><span data-stu-id="0d827-125">Open **./app/assets/stylesheets/application.css** and add the following to the end of the file.</span></span>

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. <span data-ttu-id="0d827-126">Generieren Sie einen Startseitencontroller mit dem folgenden Befehl.</span><span class="sxs-lookup"><span data-stu-id="0d827-126">Generate a home page controller with the following command.</span></span>

    ```Shell
    rails generate controller Home index
    ```

1. <span data-ttu-id="0d827-127">Konfigurieren Sie `index` die Aktion auf dem Controller als `Home` Standardseite für die App.</span><span class="sxs-lookup"><span data-stu-id="0d827-127">Configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="0d827-128">Öffnen **Sie ./config/routes.rb,** und ersetzen Sie dessen Inhalt durch Folgendes:</span><span class="sxs-lookup"><span data-stu-id="0d827-128">Open **./config/routes.rb** and replace its contents with the following</span></span>

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. <span data-ttu-id="0d827-129">Öffnen **Sie ./app/view/home/index.html.erb,** und ersetzen Sie dessen Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="0d827-129">Open **./app/view/home/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. <span data-ttu-id="0d827-130">Fügen Sie eine PNG-Datei **no-profile-photo.png** **im Verzeichnis ./app/assets/images** hinzu.</span><span class="sxs-lookup"><span data-stu-id="0d827-130">Add a PNG file named **no-profile-photo.png** in the **./app/assets/images** directory.</span></span>

1. <span data-ttu-id="0d827-131">Speichern Sie alle Änderungen, und starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="0d827-131">Save all of your changes and restart the server.</span></span> <span data-ttu-id="0d827-132">Jetzt sollte die App ganz anders aussehen.</span><span class="sxs-lookup"><span data-stu-id="0d827-132">Now, the app should look very different.</span></span>

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
