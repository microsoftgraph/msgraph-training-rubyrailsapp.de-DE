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

Nehmen Sie sich einen Moment Zeit, um zu überprüfen, was dieser Code tut. Es stellt eine einfache Get-Anforderung über `httparty` das gem an den angeforderten Endpunkt. Er sendet das Zugriffstoken in der `Authorization` Kopfzeile und enthält alle übergebenen Abfrageparameter.

Um beispielsweise die `make_api_call` -Methode zu verwenden, um einen get `https://graph.microsoft.com/v1.0/me?$select=displayName`-to durchführen zu können, können Sie es wie folgt aufrufen:

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

Sie werden später darauf aufbauen, wenn Sie weitere Features von Microsoft Graph in die APP implementieren.

## <a name="get-calendar-events-from-outlook"></a>Abrufen von Kalenderereignissen von Outlook

1. Führen Sie in der CLI den folgenden Befehl aus, um einen neuen Controller hinzuzufügen.

    ```Shell
    rails generate controller Calendar index
    ```

1. Fügen Sie die neue Route zu **./config/routes.RB**.

    ```ruby
    get 'calendar', to: 'calendar#index'
    ```

1. Fügen Sie der Graph-Hilfe eine neue Methode hinzu, um [die Ereignisse des Benutzers aufzulisten](/graph/api/user-list-events?view=graph-rest-1.0). Öffnen Sie **/App/Helpers/graph_helper. RB** , und fügen Sie dem `GraphHelper` Modul die folgende Methode hinzu.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    Überlegen Sie sich, was dieser Code macht.

    - Die URL, die aufgerufen wird, lautet `/v1.0/me/events`.
    - Der `$select` Parameter schränkt die zurückgegebenen Felder für die einzelnen Ereignisse auf diejenigen ein, die von unserer Ansicht tatsächlich verwendet werden.
    - Der `$orderby` Parameter sortiert die Ergebnisse nach dem Datum und der Uhrzeit, zu der Sie erstellt wurden, wobei das letzte Element zuerst angezeigt wird.
    - Für eine erfolgreiche Antwort gibt es das Array von Elementen zurück, die `value` im Schlüssel enthalten sind.

1. Öffnen Sie **/App/Controllers/calendar_controller. RB** , und ersetzen Sie den gesamten Inhalt durch Folgendes.

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

1. Starten Sie den Server neu. Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** . Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.

## <a name="display-the-results"></a>Anzeigen der Ergebnisse

Nun können Sie HTML hinzufügen, um die Ergebnisse auf eine benutzerfreundlichere Weise anzuzeigen.

1. Öffnen Sie **./app/views/Calendar/Index.html.Erb** , und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    Dadurch wird eine Ereignissammlung durchlaufen und jedem Ereignis wird jeweils eine Tabellenzeile hinzugefügt.

1. Entfernen Sie `render json: @events` die-Reihe `index` aus der Aktion in **./app/Controllers/calendar_controller. RB**.

1. Aktualisieren Sie die Seite, und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
