<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="fd442-101">In dieser Übung werden Sie das Microsoft Graph in die Anwendung integrieren.</span><span class="sxs-lookup"><span data-stu-id="fd442-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="fd442-102">Für diese Anwendung verwenden Sie das [httparty](https://github.com/jnunemaker/httparty) -gem, um Anrufe an Microsoft Graph zu tätigen.</span><span class="sxs-lookup"><span data-stu-id="fd442-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="fd442-103">Erstellen einer Graph-Hilfsfunktion</span><span class="sxs-lookup"><span data-stu-id="fd442-103">Create a Graph helper</span></span>

1. <span data-ttu-id="fd442-104">Erstellen Sie einen Helfer zum Verwalten aller API-Aufrufe.</span><span class="sxs-lookup"><span data-stu-id="fd442-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="fd442-105">Führen Sie den folgenden Befehl in der CLI aus, um den Helfer zu generieren.</span><span class="sxs-lookup"><span data-stu-id="fd442-105">Run the following command in your CLI to generate the helper.</span></span>

    ```Shell
    rails generate helper Graph
    ```

1. <span data-ttu-id="fd442-106">Öffnen Sie **/App/Helpers/graph_helper. RB** , und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="fd442-106">Open **./app/helpers/graph_helper.rb** and replace the contents with the following.</span></span>

    ```ruby
    require 'httparty'

    # Graph API helper methods
    module GraphHelper
      GRAPH_HOST = 'https://graph.microsoft.com'.freeze

      def make_api_call(method, endpoint, token, headers = nil, params = nil, payload = nil)
        headers ||= {}
        headers[:Authorization] = "Bearer #{token}"
        headers[:Accept] = 'application/json'

        params ||= {}

        case method.upcase
        when 'GET'
          HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                       :headers => headers,
                       :query => params
        when 'POST'
          headers['Content-Type'] = 'application/json'
          HTTParty.post "#{GRAPH_HOST}#{endpoint}",
                        :headers => headers,
                        :query => params,
                        :body => payload ? payload.to_json : nil
        else
          raise "HTTP method #{method.upcase} not implemented"
        end
      end
    end
    ```

<span data-ttu-id="fd442-107">Nehmen Sie sich einen Moment Zeit, um zu überprüfen, was dieser Code tut.</span><span class="sxs-lookup"><span data-stu-id="fd442-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="fd442-108">Es wird eine einfache Get-oder Post-Anforderung über das `httparty` gem an den angeforderten Endpunkt gesendet.</span><span class="sxs-lookup"><span data-stu-id="fd442-108">It makes a simple GET or POST request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="fd442-109">Er sendet das Zugriffstoken in der `Authorization` Kopfzeile und enthält alle übergebenen Abfrageparameter.</span><span class="sxs-lookup"><span data-stu-id="fd442-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="fd442-110">Um beispielsweise die `make_api_call` -Methode zu verwenden, um einen get-to durchführen zu können `https://graph.microsoft.com/v1.0/me?$select=displayName` , können Sie es wie folgt aufrufen:</span><span class="sxs-lookup"><span data-stu-id="fd442-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call 'GET', '/v1.0/me', access_token, { '$select': 'displayName' }
```

<span data-ttu-id="fd442-111">Sie werden später darauf aufbauen, wenn Sie weitere Features von Microsoft Graph in die APP implementieren.</span><span class="sxs-lookup"><span data-stu-id="fd442-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="fd442-112">Abrufen von Kalenderereignissen von Outlook</span><span class="sxs-lookup"><span data-stu-id="fd442-112">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="fd442-113">Führen Sie in der CLI den folgenden Befehl aus, um einen neuen Controller hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="fd442-113">In your CLI, run the following command to add a new controller.</span></span>

    ```Shell
    rails generate controller Calendar index new
    ```

1. <span data-ttu-id="fd442-114">Fügen Sie die neue Route zu **./config/routes.RB**.</span><span class="sxs-lookup"><span data-stu-id="fd442-114">Add the new route to **./config/routes.rb**.</span></span>

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. <span data-ttu-id="fd442-115">Fügen Sie eine neue Methode zur Graph-Helfer hinzu, um [eine Kalenderansicht zu erhalten](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span><span class="sxs-lookup"><span data-stu-id="fd442-115">Add a new method to the Graph helper to [get a calendar view](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span></span> <span data-ttu-id="fd442-116">Öffnen Sie **/App/Helpers/graph_helper. RB** , und fügen Sie dem Modul die folgende Methode hinzu `GraphHelper` .</span><span class="sxs-lookup"><span data-stu-id="fd442-116">Open **./app/helpers/graph_helper.rb** and add the following method to the `GraphHelper` module.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    <span data-ttu-id="fd442-117">Überlegen Sie sich, was dieser Code macht.</span><span class="sxs-lookup"><span data-stu-id="fd442-117">Consider what this code is doing.</span></span>

    - <span data-ttu-id="fd442-118">Die URL, die aufgerufen wird, lautet `/v1.0/me/calendarview`.</span><span class="sxs-lookup"><span data-stu-id="fd442-118">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="fd442-119">Die `Prefer: outlook.timezone` Kopfzeile bewirkt, dass die Anfangs-und Endzeiten in den Ergebnissen an die Zeitzone des Benutzers angepasst werden.</span><span class="sxs-lookup"><span data-stu-id="fd442-119">The `Prefer: outlook.timezone` header causes the start and end times in the results to be adjusted to the user's time zone.</span></span>
        - <span data-ttu-id="fd442-120">Die `startDateTime` `endDateTime` Parameter und legen den Anfang und das Ende der Ansicht fest.</span><span class="sxs-lookup"><span data-stu-id="fd442-120">The `startDateTime` and `endDateTime` parameters set the start and end of the view.</span></span>
        - <span data-ttu-id="fd442-121">Der `$select` Parameter schränkt die für die einzelnen Ereignisse zurückgegebenen Felder auf diejenigen ein, die von der Ansicht tatsächlich verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="fd442-121">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
        - <span data-ttu-id="fd442-122">Der `$orderby` Parameter sortiert die Ergebnisse nach dem Startzeitpunkt.</span><span class="sxs-lookup"><span data-stu-id="fd442-122">The `$orderby` parameter sorts the results by start time.</span></span>
        - <span data-ttu-id="fd442-123">Der `$top` Parameter schränkt die Ergebnisse auf 50-Ereignisse ein.</span><span class="sxs-lookup"><span data-stu-id="fd442-123">The `$top` parameter limits the results to 50 events.</span></span>
    - <span data-ttu-id="fd442-124">Für eine erfolgreiche Antwort gibt es das Array von Elementen zurück, die im `value` Schlüssel enthalten sind.</span><span class="sxs-lookup"><span data-stu-id="fd442-124">For a successful response, it returns the array of items contained in the `value` key.</span></span>

1. <span data-ttu-id="fd442-125">Fügen Sie der Graph-Hilfe eine neue Methode hinzu, um einen [IANA-Zeitzonenbezeichner](https://www.iana.org/time-zones) basierend auf einem Windows-Zeitzonennamen zu suchen.</span><span class="sxs-lookup"><span data-stu-id="fd442-125">Add a new method to the Graph helper to lookup an [IANA time zone identifier](https://www.iana.org/time-zones) based on a Windows time zone name.</span></span> <span data-ttu-id="fd442-126">Dies ist erforderlich, da Microsoft Graph Zeitzonen als Windows-Zeitzonennamen zurückgeben kann, und die Ruby- **DateTime** -Klasse erfordert IANA-Zeitzonenbezeichner.</span><span class="sxs-lookup"><span data-stu-id="fd442-126">This is necessary because Microsoft Graph can return time zones as Windows time zone names, and the Ruby **DateTime** class requires IANA time zone identifiers.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. <span data-ttu-id="fd442-127">Öffnen Sie **/App/Controllers/calendar_controller. RB** , und ersetzen Sie den gesamten Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="fd442-127">Open **./app/controllers/calendar_controller.rb** and replace its entire contents with the following.</span></span>

    ```ruby
    # Calendar controller
    class CalendarController < ApplicationController
      include GraphHelper

      def index
        # Get the IANA identifier of the user's time zone
        time_zone = get_iana_from_windows(user_timezone)

        # Calculate the start and end of week in the user's time zone
        start_datetime = Date.today.beginning_of_week(:sunday).in_time_zone(time_zone).to_time
        end_datetime = start_datetime.advance(:days => 7)

        @events = get_calendar_view access_token, start_datetime, end_datetime, user_timezone || []
        render json: @events
      rescue RuntimeError => e
        @errors = [
          {
            :message => 'Microsoft Graph returned an error getting events.',
            :debug => e
          }
        ]
      end
    end
    ```

1. <span data-ttu-id="fd442-128">Starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="fd442-128">Restart the server.</span></span> <span data-ttu-id="fd442-129">Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** .</span><span class="sxs-lookup"><span data-stu-id="fd442-129">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="fd442-130">Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="fd442-130">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="fd442-131">Anzeigen der Ergebnisse</span><span class="sxs-lookup"><span data-stu-id="fd442-131">Display the results</span></span>

<span data-ttu-id="fd442-132">Nun können Sie HTML hinzufügen, um die Ergebnisse auf eine benutzerfreundlichere Weise anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="fd442-132">Now you can add HTML to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="fd442-133">Öffnen Sie **./app/views/Calendar/index.html. Erb** , und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="fd442-133">Open **./app/views/calendar/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    <span data-ttu-id="fd442-134">Dadurch wird eine Ereignissammlung durchlaufen und jedem Ereignis wird jeweils eine Tabellenzeile hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="fd442-134">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="fd442-135">Entfernen Sie die- `render json: @events` Reihe aus der `index` Aktion in **./app/Controllers/calendar_controller. RB**.</span><span class="sxs-lookup"><span data-stu-id="fd442-135">Remove the `render json: @events` line from the `index` action in **./app/controllers/calendar_controller.rb**.</span></span>

1. <span data-ttu-id="fd442-136">Aktualisieren Sie die Seite, und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.</span><span class="sxs-lookup"><span data-stu-id="fd442-136">Refresh the page and the app should now render a table of events.</span></span>

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
