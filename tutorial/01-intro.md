<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="e9e66-101">In diesem Lernprogramm erfahren Sie, wie Sie eine Ruby on Rails-Web-App erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.</span><span class="sxs-lookup"><span data-stu-id="e9e66-101">This tutorial teaches you how to build a Ruby on Rails web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="e9e66-102">Wenn Sie das fertige Lernprogramm einfach herunterladen möchten, können Sie es auf zwei Arten herunterladen.</span><span class="sxs-lookup"><span data-stu-id="e9e66-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="e9e66-103">Laden Sie den [Ruby-Schnellstart herunter,](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) um Arbeitscode in Minuten zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="e9e66-103">Download the [Ruby quick start](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) to get working code in minutes.</span></span>
> - <span data-ttu-id="e9e66-104">Laden Sie das GitHub [herunter oder klonen Sie es.](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)</span><span class="sxs-lookup"><span data-stu-id="e9e66-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="e9e66-105">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="e9e66-105">Prerequisites</span></span>

<span data-ttu-id="e9e66-106">Bevor Sie dieses Lernprogramm starten, sollten Sie die folgenden Tools auf Ihrem Entwicklungscomputer installiert haben.</span><span class="sxs-lookup"><span data-stu-id="e9e66-106">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="e9e66-107">Ruby</span><span class="sxs-lookup"><span data-stu-id="e9e66-107">Ruby</span></span>](https://www.ruby-lang.org/en/downloads/)
- [<span data-ttu-id="e9e66-108">SQLite3</span><span class="sxs-lookup"><span data-stu-id="e9e66-108">SQLite3</span></span>](https://sqlite.org/index.html)
- [<span data-ttu-id="e9e66-109">Node.js</span><span class="sxs-lookup"><span data-stu-id="e9e66-109">Node.js</span></span>](https://nodejs.org/en/)
- [<span data-ttu-id="e9e66-110">Garn</span><span class="sxs-lookup"><span data-stu-id="e9e66-110">Yarn</span></span>](https://yarnpkg.com/)

<span data-ttu-id="e9e66-111">Sie sollten auch über ein persönliches Microsoft-Konto mit einem Postfach auf Outlook.com oder über ein Microsoft-Arbeits- oder Schulkonto verfügen.</span><span class="sxs-lookup"><span data-stu-id="e9e66-111">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="e9e66-112">Wenn Sie kein Microsoft-Konto haben, gibt es mehrere Optionen, um ein kostenloses Konto zu erhalten:</span><span class="sxs-lookup"><span data-stu-id="e9e66-112">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="e9e66-113">Sie können [sich für ein neues persönliches Microsoft-Konto registrieren.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)</span><span class="sxs-lookup"><span data-stu-id="e9e66-113">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="e9e66-114">Sie können [sich für das Office 365-Entwicklerprogramm](https://developer.microsoft.com/office/dev-program) registrieren, um ein kostenloses Office 365 erhalten.</span><span class="sxs-lookup"><span data-stu-id="e9e66-114">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="e9e66-115">Dieses Lernprogramm wurde mit den folgenden Versionen der erforderlichen Tools geschrieben.</span><span class="sxs-lookup"><span data-stu-id="e9e66-115">This tutorial was written with the following versions of the required tools.</span></span> <span data-ttu-id="e9e66-116">Die Schritte in diesem Handbuch funktionieren möglicherweise mit anderen Versionen, aber dies wurde nicht getestet.</span><span class="sxs-lookup"><span data-stu-id="e9e66-116">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="e9e66-117">Ruby Version 3.0.1</span><span class="sxs-lookup"><span data-stu-id="e9e66-117">Ruby version 3.0.1</span></span>
> - <span data-ttu-id="e9e66-118">SQLite3 Version 3.35.5</span><span class="sxs-lookup"><span data-stu-id="e9e66-118">SQLite3 version 3.35.5</span></span>
> - <span data-ttu-id="e9e66-119">Node.js Version 14.15.0</span><span class="sxs-lookup"><span data-stu-id="e9e66-119">Node.js version 14.15.0</span></span>
> - <span data-ttu-id="e9e66-120">Garnversion 1.22.0</span><span class="sxs-lookup"><span data-stu-id="e9e66-120">Yarn version 1.22.0</span></span>

## <a name="feedback"></a><span data-ttu-id="e9e66-121">Feedback</span><span class="sxs-lookup"><span data-stu-id="e9e66-121">Feedback</span></span>

<span data-ttu-id="e9e66-122">Bitte geben Sie Feedback zu diesem Lernprogramm im [GitHub Repository.](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)</span><span class="sxs-lookup"><span data-stu-id="e9e66-122">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>
