# <a name="how-to-run-the-completed-project"></a><span data-ttu-id="26f7a-101">Vorgehensweise Ausführen des abgeschlossenen Projekts</span><span class="sxs-lookup"><span data-stu-id="26f7a-101">How to run the completed project</span></span>

## <a name="prerequisites"></a><span data-ttu-id="26f7a-102">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="26f7a-102">Prerequisites</span></span>

<span data-ttu-id="26f7a-103">Um das abgeschlossene Projekt in diesem Ordner auszuführen, benötigen Sie Folgendes:</span><span class="sxs-lookup"><span data-stu-id="26f7a-103">To run the completed project in this folder, you need the following:</span></span>

- <span data-ttu-id="26f7a-104">[Ruby](https://www.ruby-lang.org/en/downloads/) auf dem Entwicklungscomputer installiert.</span><span class="sxs-lookup"><span data-stu-id="26f7a-104">[Ruby](https://www.ruby-lang.org/en/downloads/) installed on your development machine.</span></span> <span data-ttu-id="26f7a-105">Wenn Sie keinen Ruby haben, besuchen Sie den vorherigen Link für Downloadoptionen.</span><span class="sxs-lookup"><span data-stu-id="26f7a-105">If you do not have Ruby, visit the previous link for download options.</span></span> <span data-ttu-id="26f7a-106">(**Hinweis:** dieses Tutorial wurde mit Ruby Version 2.6.5 geschrieben.</span><span class="sxs-lookup"><span data-stu-id="26f7a-106">(**Note:** This tutorial was written with Ruby version 2.6.5.</span></span> <span data-ttu-id="26f7a-107">Die Schritte in diesem Leitfaden funktionieren möglicherweise mit anderen Versionen, aber das wurde nicht getestet.)</span><span class="sxs-lookup"><span data-stu-id="26f7a-107">The steps in this guide may work with other versions, but that has not been tested.)</span></span>
- <span data-ttu-id="26f7a-108">Entweder ein persönliches Microsoft-Konto mit einem Postfach auf Outlook.com oder ein Microsoft-Arbeits-oder Schulkonto.</span><span class="sxs-lookup"><span data-stu-id="26f7a-108">Either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span>

<span data-ttu-id="26f7a-109">Wenn Sie kein Microsoft-Konto haben, gibt es mehrere Optionen, um ein kostenloses Konto zu erhalten:</span><span class="sxs-lookup"><span data-stu-id="26f7a-109">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="26f7a-110">Sie können [sich für ein neues persönliches Microsoft-Konto registrieren](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span><span class="sxs-lookup"><span data-stu-id="26f7a-110">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="26f7a-111">Sie können sich [für das Office 365 Entwicklerprogramm registrieren](https://developer.microsoft.com/office/dev-program) , um ein kostenloses Office 365-Abonnement zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="26f7a-111">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

## <a name="register-a-web-application-with-the-azure-active-directory-admin-center"></a><span data-ttu-id="26f7a-112">Registrieren einer Webanwendung im Azure-Active Directory Admin Center</span><span class="sxs-lookup"><span data-stu-id="26f7a-112">Register a web application with the Azure Active Directory admin center</span></span>

1. <span data-ttu-id="26f7a-113">Öffnen Sie einen Browser, und navigieren Sie zum [Azure Active Directory Admin Center](https://aad.portal.azure.com).</span><span class="sxs-lookup"><span data-stu-id="26f7a-113">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com).</span></span> <span data-ttu-id="26f7a-114">Melden Sie sich mit einem **persönlichen Konto** (auch: Microsoft-Konto) oder einem **Geschäfts- oder Schulkonto** an.</span><span class="sxs-lookup"><span data-stu-id="26f7a-114">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="26f7a-115">Wählen Sie in der linken Navigationsleiste **Azure Active Directory** aus, und wählen Sie dann **App-Registrierungen** unter **Verwalten** aus.</span><span class="sxs-lookup"><span data-stu-id="26f7a-115">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="26f7a-116">Screenshot der APP-Registrierungen</span><span class="sxs-lookup"><span data-stu-id="26f7a-116">A screenshot of the App registrations</span></span> ](/tutorial/images/aad-portal-app-registrations.png)

1. <span data-ttu-id="26f7a-117">Wählen Sie **Neue Registrierung** aus.</span><span class="sxs-lookup"><span data-stu-id="26f7a-117">Select **New registration**.</span></span> <span data-ttu-id="26f7a-118">Legen Sie auf der Seite **Anwendung registrieren** die Werte wie folgt fest.</span><span class="sxs-lookup"><span data-stu-id="26f7a-118">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="26f7a-119">Legen Sie **Name** auf `Ruby Graph Tutorial` fest.</span><span class="sxs-lookup"><span data-stu-id="26f7a-119">Set **Name** to `Ruby Graph Tutorial`.</span></span>
    - <span data-ttu-id="26f7a-120">Legen Sie **Unterstützte Kontotypen** auf **Konten in allen Organisationsverzeichnissen und persönliche Microsoft-Konten** fest.</span><span class="sxs-lookup"><span data-stu-id="26f7a-120">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="26f7a-121">Legen Sie unter **Umleitungs-URI** die erste Dropdownoption auf `Web` fest, und legen Sie den Wert auf `http://localhost:3000/auth/microsoft_graph_auth/callback` fest.</span><span class="sxs-lookup"><span data-stu-id="26f7a-121">Under **Redirect URI**, set the first drop-down to `Web` and set the value to `http://localhost:3000/auth/microsoft_graph_auth/callback`.</span></span>

    ![Screenshot der Seite "Anwendung registrieren"](/tutorial/images/aad-register-an-app.png)

1. <span data-ttu-id="26f7a-123">Wählen Sie **Registrieren** aus.</span><span class="sxs-lookup"><span data-stu-id="26f7a-123">Choose **Register**.</span></span> <span data-ttu-id="26f7a-124">Kopieren Sie auf der Seite **Ruby Graph Tutorial** den Wert der **Anwendungs-ID (Client)** , und speichern Sie Sie, um Sie im nächsten Schritt zu benötigen.</span><span class="sxs-lookup"><span data-stu-id="26f7a-124">On the **Ruby Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![Screenshot der Anwendungs-ID der neuen App-Registrierung](/tutorial/images/aad-application-id.png)

1. <span data-ttu-id="26f7a-126">Wählen Sie unter **Verwalten** die Option **Zertifikate und Geheime Clientschlüssel** aus.</span><span class="sxs-lookup"><span data-stu-id="26f7a-126">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="26f7a-127">Wählen Sie die Schaltfläche **Neuen geheimen Clientschlüssel** aus.</span><span class="sxs-lookup"><span data-stu-id="26f7a-127">Select the **New client secret** button.</span></span> <span data-ttu-id="26f7a-128">Geben Sie einen Wert unter **Beschreibung** ein, wählen Sie eine der Optionen für **Gilt bis** aus, und wählen Sie dann **Hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="26f7a-128">Enter a value in **Description** and select one of the options for **Expires** and choose **Add**.</span></span>

    ![Screenshot des Dialogfelds "Geheimen Clientschlüssel hinzufügen"](/tutorial/images/aad-new-client-secret.png)

1. <span data-ttu-id="26f7a-130">Kopieren Sie den Wert des geheimen Clientschlüssels, bevor Sie diese Seite verlassen.</span><span class="sxs-lookup"><span data-stu-id="26f7a-130">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="26f7a-131">Sie benötigen ihn im nächsten Schritt.</span><span class="sxs-lookup"><span data-stu-id="26f7a-131">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="26f7a-132">Dieser geheime Clientschlüssel wird nicht noch einmal angezeigt, stellen Sie daher sicher, dass Sie ihn jetzt kopieren.</span><span class="sxs-lookup"><span data-stu-id="26f7a-132">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![Screenshot des neu hinzugefügten Clientschlüssels](/tutorial/images/aad-copy-client-secret.png)

## <a name="configure-the-sample"></a><span data-ttu-id="26f7a-134">Konfigurieren des Beispiels</span><span class="sxs-lookup"><span data-stu-id="26f7a-134">Configure the sample</span></span>

1. <span data-ttu-id="26f7a-135">Benennen `./config/oauth_environment_variables.rb.example` Sie die Datei in `oauth_environment_variables.rb` .</span><span class="sxs-lookup"><span data-stu-id="26f7a-135">Rename the `./config/oauth_environment_variables.rb.example` file to `oauth_environment_variables.rb`.</span></span>
1. <span data-ttu-id="26f7a-136">Bearbeiten Sie die `oauth_environment_variables.rb` Datei, und nehmen Sie die folgenden Änderungen vor.</span><span class="sxs-lookup"><span data-stu-id="26f7a-136">Edit the `oauth_environment_variables.rb` file and make the following changes.</span></span>
    1. <span data-ttu-id="26f7a-137">Ersetzen `YOUR_APP_ID_HERE` Sie durch die **Anwendungs-ID** , die Sie im App-Registrierungs Portal erhalten haben.</span><span class="sxs-lookup"><span data-stu-id="26f7a-137">Replace `YOUR_APP_ID_HERE` with the **Application Id** you got from the App Registration Portal.</span></span>
    1. <span data-ttu-id="26f7a-138">Ersetzen `YOUR APP PASSWORD HERE` Sie durch das Kennwort, das Sie aus dem App-Registrierungs Portal erhalten haben.</span><span class="sxs-lookup"><span data-stu-id="26f7a-138">Replace `YOUR APP PASSWORD HERE` with the password you got from the App Registration Portal.</span></span>
1. <span data-ttu-id="26f7a-139">Navigieren Sie in der Befehlszeilenschnittstelle (CLI) zu diesem Verzeichnis, und führen Sie den folgenden Befehl aus, um Anforderungen zu installieren.</span><span class="sxs-lookup"><span data-stu-id="26f7a-139">In your command-line interface (CLI), navigate to this directory and run the following command to install requirements.</span></span>

    ```Shell
    bundle install
    ```

1. <span data-ttu-id="26f7a-140">Führen Sie in ihrer CLI den folgenden Befehl aus, um Garn Pakete zu installieren.</span><span class="sxs-lookup"><span data-stu-id="26f7a-140">In your CLI, run the following command to install yarn packages.</span></span>

    ```Shell
    yarn install
    ```

1. <span data-ttu-id="26f7a-141">Führen Sie in der CLI den folgenden Befehl aus, um die Datenbank der APP zu initialisieren.</span><span class="sxs-lookup"><span data-stu-id="26f7a-141">In your CLI, run the following command to initialize the app's database.</span></span>

    ```Shell
    rake db:migrate
    ```

## <a name="run-the-sample"></a><span data-ttu-id="26f7a-142">Ausführen des Beispiels</span><span class="sxs-lookup"><span data-stu-id="26f7a-142">Run the sample</span></span>

1. <span data-ttu-id="26f7a-143">Führen Sie den folgenden Befehl in der CLI aus, um die Anwendung zu starten.</span><span class="sxs-lookup"><span data-stu-id="26f7a-143">Run the following command in your CLI to start the application.</span></span>

    ```Shell
    rails server
    ```

1. <span data-ttu-id="26f7a-144">Öffnen Sie einen Browser, und navigieren Sie zu `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="26f7a-144">Open a browser and browse to `http://localhost:3000`.</span></span>
