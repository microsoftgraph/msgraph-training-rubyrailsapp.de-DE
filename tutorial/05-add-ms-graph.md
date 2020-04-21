<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="fe07a-101">In dieser Übung werden Sie das Microsoft Graph in die Anwendung integrieren.</span><span class="sxs-lookup"><span data-stu-id="fe07a-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="fe07a-102">Für diese Anwendung verwenden Sie das [httparty](https://github.com/jnunemaker/httparty) -gem, um Anrufe an Microsoft Graph zu tätigen.</span><span class="sxs-lookup"><span data-stu-id="fe07a-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="fe07a-103">Erstellen einer Graph-Hilfsfunktion</span><span class="sxs-lookup"><span data-stu-id="fe07a-103">Create a Graph helper</span></span>

1. <span data-ttu-id="fe07a-104">Erstellen Sie einen Helfer zum Verwalten aller API-Aufrufe.</span><span class="sxs-lookup"><span data-stu-id="fe07a-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="fe07a-105">Führen Sie den folgenden Befehl in der CLI aus, um den Helfer zu generieren.</span><span class="sxs-lookup"><span data-stu-id="fe07a-105">Run the following command in your CLI to generate the helper.</span></span>

    ```Shell
    rails generate helper Graph
    ```

1. <span data-ttu-id="fe07a-106">Öffnen Sie **/App/Helpers/graph_helper. RB** , und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="fe07a-106">Open **./app/helpers/graph_helper.rb** and replace the contents with the following.</span></span>

    ```ruby
    require 'httparty'

    # Graph API helper methods
    module GraphHelper
      GRAPH_HOST = 'https://graph.microsoft.com'.freeze

      def make_api_call(endpoint, token, params = nil)
        headers = {
          Authorization: "Bearer #{token}"
        }

        query = params || {}

        HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                     headers: headers,
                     query: query
      end
    end
    ```

<span data-ttu-id="fe07a-107">Nehmen Sie sich einen Moment Zeit, um zu überprüfen, was dieser Code tut.</span><span class="sxs-lookup"><span data-stu-id="fe07a-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="fe07a-108">Es stellt eine einfache Get-Anforderung über `httparty` das gem an den angeforderten Endpunkt.</span><span class="sxs-lookup"><span data-stu-id="fe07a-108">It makes a simple GET request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="fe07a-109">Er sendet das Zugriffstoken in der `Authorization` Kopfzeile und enthält alle übergebenen Abfrageparameter.</span><span class="sxs-lookup"><span data-stu-id="fe07a-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="fe07a-110">Um beispielsweise die `make_api_call` -Methode zu verwenden, um einen get `https://graph.microsoft.com/v1.0/me?$select=displayName`-to durchführen zu können, können Sie es wie folgt aufrufen:</span><span class="sxs-lookup"><span data-stu-id="fe07a-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

<span data-ttu-id="fe07a-111">Sie werden später darauf aufbauen, wenn Sie weitere Features von Microsoft Graph in die APP implementieren.</span><span class="sxs-lookup"><span data-stu-id="fe07a-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="fe07a-112">Abrufen von Kalenderereignissen von Outlook</span><span class="sxs-lookup"><span data-stu-id="fe07a-112">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="fe07a-113">Führen Sie in der CLI den folgenden Befehl aus, um einen neuen Controller hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="fe07a-113">In your CLI, run the following command to add a new controller.</span></span>

    ```Shell
    rails generate controller Calendar index
    ```

1. <span data-ttu-id="fe07a-114">Fügen Sie die neue Route zu **./config/routes.RB**.</span><span class="sxs-lookup"><span data-stu-id="fe07a-114">Add the new route to **./config/routes.rb**.</span></span>

    ```ruby
    get 'calendar', to: 'calendar#index'
    ```

1. <span data-ttu-id="fe07a-115">Fügen Sie der Graph-Hilfe eine neue Methode hinzu, um [die Ereignisse des Benutzers aufzulisten](/graph/api/user-list-events?view=graph-rest-1.0).</span><span class="sxs-lookup"><span data-stu-id="fe07a-115">Add a new method to the Graph helper to [list the user's events](/graph/api/user-list-events?view=graph-rest-1.0).</span></span> <span data-ttu-id="fe07a-116">Öffnen Sie **/App/Helpers/graph_helper. RB** , und fügen Sie dem `GraphHelper` Modul die folgende Methode hinzu.</span><span class="sxs-lookup"><span data-stu-id="fe07a-116">Open **./app/helpers/graph_helper.rb** and add the following method to the `GraphHelper` module.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    <span data-ttu-id="fe07a-117">Überlegen Sie sich, was dieser Code macht.</span><span class="sxs-lookup"><span data-stu-id="fe07a-117">Consider what this code is doing.</span></span>

    - <span data-ttu-id="fe07a-118">Die URL, die aufgerufen wird, lautet `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="fe07a-118">The URL that will be called is `/v1.0/me/events`.</span></span>
    - <span data-ttu-id="fe07a-119">Der `$select` Parameter schränkt die zurückgegebenen Felder für die einzelnen Ereignisse auf diejenigen ein, die von unserer Ansicht tatsächlich verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="fe07a-119">The `$select` parameter limits the fields returned for each events to just those our view will actually use.</span></span>
    - <span data-ttu-id="fe07a-120">Der `$orderby` Parameter sortiert die Ergebnisse nach dem Datum und der Uhrzeit, zu der Sie erstellt wurden, wobei das letzte Element zuerst angezeigt wird.</span><span class="sxs-lookup"><span data-stu-id="fe07a-120">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
    - <span data-ttu-id="fe07a-121">Für eine erfolgreiche Antwort gibt es das Array von Elementen zurück, die `value` im Schlüssel enthalten sind.</span><span class="sxs-lookup"><span data-stu-id="fe07a-121">For a successful response, it returns the array of items contained in the `value` key.</span></span>

1. <span data-ttu-id="fe07a-122">Öffnen Sie **/App/Controllers/calendar_controller. RB** , und ersetzen Sie den gesamten Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="fe07a-122">Open **./app/controllers/calendar_controller.rb** and replace its entire contents with the following.</span></span>

    ```ruby
    # Calendar controller
    class CalendarController < ApplicationController
      include GraphHelper

      def index
        @events = get_calendar_events access_token || []
        render json: @events
      rescue RuntimeError => e
        @errors = [
          {
            message: 'Microsoft Graph returned an error getting events.',
            debug: e
          }
        ]
      end
    end
    ```

1. <span data-ttu-id="fe07a-123">Starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="fe07a-123">Restart the server.</span></span> <span data-ttu-id="fe07a-124">Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** .</span><span class="sxs-lookup"><span data-stu-id="fe07a-124">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="fe07a-125">Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="fe07a-125">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="fe07a-126">Anzeigen der Ergebnisse</span><span class="sxs-lookup"><span data-stu-id="fe07a-126">Display the results</span></span>

<span data-ttu-id="fe07a-127">Nun können Sie HTML hinzufügen, um die Ergebnisse auf eine benutzerfreundlichere Weise anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="fe07a-127">Now you can add HTML to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="fe07a-128">Öffnen Sie **./app/views/Calendar/Index.html.Erb** , und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="fe07a-128">Open **./app/views/calendar/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    <span data-ttu-id="fe07a-129">Dadurch wird eine Ereignissammlung durchlaufen und jedem Ereignis wird jeweils eine Tabellenzeile hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="fe07a-129">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="fe07a-130">Entfernen Sie `render json: @events` die-Reihe `index` aus der Aktion in **./app/Controllers/calendar_controller. RB**.</span><span class="sxs-lookup"><span data-stu-id="fe07a-130">Remove the `render json: @events` line from the `index` action in **./app/controllers/calendar_controller.rb**.</span></span>

1. <span data-ttu-id="fe07a-131">Aktualisieren Sie die Seite, und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.</span><span class="sxs-lookup"><span data-stu-id="fe07a-131">Refresh the page and the app should now render a table of events.</span></span>

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
