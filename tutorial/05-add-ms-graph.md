<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung werden Sie das Microsoft Graph in die Anwendung integrieren. Für diese Anwendung verwenden Sie das [httparty](https://github.com/jnunemaker/httparty) -gem, um Anrufe an Microsoft Graph zu tätigen.

## <a name="create-a-graph-helper"></a>Erstellen einer Graph-Hilfsfunktion

1. Erstellen Sie einen Helfer zum Verwalten aller API-Aufrufe. Führen Sie den folgenden Befehl in der CLI aus, um den Helfer zu generieren.

    ```Shell
    rails generate helper Graph
    ```

1. Öffnen Sie **/App/Helpers/graph_helper. RB** , und ersetzen Sie den Inhalt durch Folgendes.

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

Nehmen Sie sich einen Moment Zeit, um zu überprüfen, was dieser Code tut. Es wird eine einfache Get-oder Post-Anforderung über das `httparty` gem an den angeforderten Endpunkt gesendet. Er sendet das Zugriffstoken in der `Authorization` Kopfzeile und enthält alle übergebenen Abfrageparameter.

Um beispielsweise die `make_api_call` -Methode zu verwenden, um einen get-to durchführen zu können `https://graph.microsoft.com/v1.0/me?$select=displayName` , können Sie es wie folgt aufrufen:

```ruby
make_api_call 'GET', '/v1.0/me', access_token, { '$select': 'displayName' }
```

Sie werden später darauf aufbauen, wenn Sie weitere Features von Microsoft Graph in die APP implementieren.

## <a name="get-calendar-events-from-outlook"></a>Abrufen von Kalenderereignissen von Outlook

1. Führen Sie in der CLI den folgenden Befehl aus, um einen neuen Controller hinzuzufügen.

    ```Shell
    rails generate controller Calendar index new
    ```

1. Fügen Sie die neue Route zu **./config/routes.RB**.

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. Fügen Sie eine neue Methode zur Graph-Helfer hinzu, um [eine Kalenderansicht zu erhalten](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0). Öffnen Sie **/App/Helpers/graph_helper. RB** , und fügen Sie dem Modul die folgende Methode hinzu `GraphHelper` .

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    Überlegen Sie sich, was dieser Code macht.

    - Die URL, die aufgerufen wird, lautet `/v1.0/me/calendarview`.
        - Die `Prefer: outlook.timezone` Kopfzeile bewirkt, dass die Anfangs-und Endzeiten in den Ergebnissen an die Zeitzone des Benutzers angepasst werden.
        - Die `startDateTime` `endDateTime` Parameter und legen den Anfang und das Ende der Ansicht fest.
        - Der `$select` Parameter schränkt die für die einzelnen Ereignisse zurückgegebenen Felder auf diejenigen ein, die von der Ansicht tatsächlich verwendet werden.
        - Der `$orderby` Parameter sortiert die Ergebnisse nach dem Startzeitpunkt.
        - Der `$top` Parameter schränkt die Ergebnisse auf 50-Ereignisse ein.
    - Für eine erfolgreiche Antwort gibt es das Array von Elementen zurück, die im `value` Schlüssel enthalten sind.

1. Fügen Sie der Graph-Hilfe eine neue Methode hinzu, um einen [IANA-Zeitzonenbezeichner](https://www.iana.org/time-zones) basierend auf einem Windows-Zeitzonennamen zu suchen. Dies ist erforderlich, da Microsoft Graph Zeitzonen als Windows-Zeitzonennamen zurückgeben kann, und die Ruby- **DateTime** -Klasse erfordert IANA-Zeitzonenbezeichner.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. Öffnen Sie **/App/Controllers/calendar_controller. RB** , und ersetzen Sie den gesamten Inhalt durch Folgendes.

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

1. Starten Sie den Server neu. Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** . Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.

## <a name="display-the-results"></a>Anzeigen der Ergebnisse

Nun können Sie HTML hinzufügen, um die Ergebnisse auf eine benutzerfreundlichere Weise anzuzeigen.

1. Öffnen Sie **./app/views/Calendar/index.html. Erb** , und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    Dadurch wird eine Ereignissammlung durchlaufen und jedem Ereignis wird jeweils eine Tabellenzeile hinzugefügt.

1. Entfernen Sie die- `render json: @events` Reihe aus der `index` Aktion in **./app/Controllers/calendar_controller. RB**.

1. Aktualisieren Sie die Seite, und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
