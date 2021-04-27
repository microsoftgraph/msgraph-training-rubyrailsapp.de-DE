<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung werden Sie die Microsoft-Graph in die Anwendung integrieren. Für diese Anwendung verwenden Sie das [httparty-Gem,](https://github.com/jnunemaker/httparty) um Anrufe an Microsoft Graph.

## <a name="create-a-graph-helper"></a>Erstellen eines Graph Helfers

1. Erstellen Sie eine Hilfshilfe zum Verwalten aller Ihrer API-Aufrufe. Führen Sie den folgenden Befehl in Ihrer CLI aus, um den Hilfsbefehl zu generieren.

    ```Shell
    rails generate helper Graph
    ```

1. Öffnen **Sie ./app/helpers/graph_helper.rb,** und ersetzen Sie den Inhalt durch Folgendes.

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

Nehmen Sie sich einen Moment Zeit, um zu überprüfen, was dieser Code macht. Es wird eine einfache GET- oder POST-Anforderung über das `httparty` Gem an den angeforderten Endpunkt gesendet. Es sendet das Zugriffstoken in der Kopfzeile und enthält `Authorization` alle übergebenen Abfrageparameter.

Wenn Sie beispielsweise die Methode verwenden `make_api_call` möchten, um ein GET zu verwenden, können Sie sie `https://graph.microsoft.com/v1.0/me?$select=displayName` wie so aufrufen:

```ruby
make_api_call 'GET', '/v1.0/me', access_token, {}, { '$select': 'displayName' }
```

Sie bauen später darauf auf, wenn Sie weitere Microsoft Graph in der App implementieren.

## <a name="get-calendar-events-from-outlook"></a>Abrufen von Kalenderereignissen von Outlook

1. Führen Sie in Der CLI den folgenden Befehl aus, um einen neuen Controller hinzuzufügen.

    ```Shell
    rails generate controller Calendar index new
    ```

1. Fügen Sie die neue Route **zu ./config/routes.rb hinzu.**

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. Fügen Sie dem Hilfshilfs-Graph eine neue Methode hinzu, um [eine Kalenderansicht zu erhalten.](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0) Öffnen **Sie ./app/helpers/graph_helper.rb,** und fügen Sie dem Modul die folgende Methode `GraphHelper` hinzu.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    Überlegen Sie sich, was dieser Code macht.

    - Die URL, die aufgerufen wird, lautet `/v1.0/me/calendarview`.
        - Der `Prefer: outlook.timezone` Header bewirkt, dass die Start- und Endzeiten in den Ergebnissen an die Zeitzone des Benutzers angepasst werden.
        - Die `startDateTime` Parameter und legen den Anfang und das Ende der Ansicht `endDateTime` fest.
        - Der Parameter beschränkt die für jedes Ereignis zurückgegebenen Felder auf die Felder, die `$select` von der Ansicht tatsächlich verwendet werden.
        - Der `$orderby` Parameter sortiert die Ergebnisse nach Startzeit.
        - Der `$top` Parameter beschränkt die Ergebnisse auf 50 Ereignisse.
    - Für eine erfolgreiche Antwort gibt sie das Array von Elementen zurück, die im Schlüssel enthalten `value` sind.

1. Fügen Sie dem Hilfshilfs-Graph eine neue Methode hinzu, um eine [IANA-Zeitzonen-ID](https://www.iana.org/time-zones) basierend auf einem Windows zeitzonennamen zu suchen. Dies ist erforderlich, da Microsoft Graph Zeitzonen als Windows Zeitzonennamen zurückgeben kann und die Ruby **DateTime-Klasse** IANA-Zeitzonenbezeichner erfordert.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. Öffnen **Sie ./app/controllers/calendar_controller.rb,** und ersetzen Sie den gesamten Inhalt durch Folgendes.

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

1. Starten Sie den Server neu. Melden Sie sich an, **und** klicken Sie in der Navigationsleiste auf den Link Kalender. Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.

## <a name="display-the-results"></a>Anzeigen der Ergebnisse

Jetzt können Sie HTML hinzufügen, um die Ergebnisse benutzerfreundlicher anzeigen zu können.

1. Öffnen **Sie ./app/views/calendar/index.html.erb,** und ersetzen Sie dessen Inhalt durch Folgendes.

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    Dadurch wird eine Ereignissammlung durchlaufen und jedem Ereignis wird jeweils eine Tabellenzeile hinzugefügt.

1. Entfernen Sie `render json: @events` die Zeile aus der Aktion in `index` **./app/controllers/calendar_controller.rb**.

1. Aktualisieren Sie die Seite, und die App sollte nun eine Tabelle mit Ereignissen rendern.

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
