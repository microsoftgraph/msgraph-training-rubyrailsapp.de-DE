<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung integrieren Sie Microsoft Graph in die Anwendung. Für diese Anwendung verwenden Sie den [httparty](https://github.com/jnunemaker/httparty) -Edelstein, um Aufrufe von Microsoft Graph zu tätigen.

## <a name="create-a-graph-helper"></a>Erstellen einer Graph-Hilfe

Erstellen Sie eine Hilfsfunktion, um alle Ihre API-Aufrufe zu verwalten. Führen Sie den folgenden Befehl in der CLI aus, um den Helfer zu generieren.

```Shell
rails generate helper Graph
```

Öffnen Sie die neu `./app/helpers/graph_helper.rb` erstellte Datei, und ersetzen Sie den Inhalt durch Folgendes.

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

Nehmen Sie sich einen Moment Zeit, um zu sehen, was dieser Code macht. Sie stellt eine einfache Get-Anforderung über `httparty` das gem an den angeforderten Endpunkt. Sie sendet das Zugriffstoken in der `Authorization` Kopfzeile und enthält alle übergebenen Abfrageparameter.

Wenn Sie beispielsweise die `make_api_call` Methode zum Abrufen von `https://graph.microsoft.com/v1.0/me?$select=displayName`verwenden möchten, können Sie Sie wie folgt aufrufen:

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

Sie werden später darauf aufbauen, wenn Sie mehr Microsoft Graph-Features in die APP implementieren.

## <a name="get-calendar-events-from-outlook"></a>Abrufen von Kalenderereignissen aus Outlook

Beginnen wir mit dem Hinzufügen der Möglichkeit, Ereignisse im Kalender des Benutzers anzuzeigen. Führen Sie in der CLI den folgenden Befehl aus, um einen neuen Controller hinzuzufügen.

```Shell
rails generate controller Calendar index
```

Nachdem die Route verfügbar ist, aktualisieren Sie den Link **Kalender** in der navbar, `./app/view/layouts/application.html.erb` um Sie zu verwenden. Ersetzen Sie die `<a class="nav-link" href="#">Calendar</a>` -Reihe durch Folgendes.

```html
<%= link_to "Calendar", {:controller => :calendar, :action => :index}, class: "nav-link#{' active' if controller.controller_name == 'calendar'}" %>
```

Fügen Sie der Graph-Hilfe eine neue Methode hinzu, um [die Ereignisse des Benutzers](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events)aufzulisten. Öffnen `./app/helpers/graph_helper.rb` Sie das `GraphHelper` Modul, und fügen Sie die folgende Methode hinzu.

```ruby
def get_calendar_events(token)
  get_events_url = '/v1.0/me/events'

  query = {
    '$select': 'subject,organizer,start,end',
    '$orderby': 'createdDateTime DESC'
  }

  response = make_api_call get_events_url, token, query

  raise response.parsed_response.to_s || "Request returned #{response.code}" unless response.code == 200
  response.parsed_response['value']
end
```

Überlegen Sie sich, was dieser Code tut.

- Die URL, die aufgerufen wird, `/v1.0/me/events`lautet.
- Der `$select` Parameter schränkt die für jedes Ereignis zurückgegebenen Felder auf diejenigen ein, die unsere Ansicht tatsächlich verwendet.
- Der `$orderby` Parameter sortiert die Ergebnisse nach dem Datum und der Uhrzeit, zu denen Sie erstellt wurden, wobei das neueste Element zuerst angezeigt wird.
- Für eine erfolgreiche Antwort gibt es das Array von Elementen zurück, die `value` im Schlüssel enthalten sind.

Jetzt können Sie dies testen. Öffnen `./app/controllers/calendar_controller.rb` und aktualisieren Sie `index` die Aktion, um diese Methode aufzurufen und die Ergebnisse zu rendern.

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

Starten Sie den Server neu. Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** . Wenn alles funktioniert, sollte ein JSON-Dump von Ereignissen im Kalender des Benutzers angezeigt werden.

## <a name="display-the-results"></a>Anzeigen der Ergebnisse

Jetzt können Sie HTML und CSS hinzufügen, um die Ergebnisse benutzerfreundlicher anzuzeigen.

Öffnen `./app/views/calendar/index.html.erb` Sie den Inhalt, und ersetzen Sie ihn durch den folgenden.

```html
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    <% @events.each do |event| %>
      <tr>
        <td><%= event['organizer']['emailAddress']['name'] %></td>
        <td><%= event['subject'] %></td>
        <td><%= event['start']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
        <td><%= event['end']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

, Der eine Auflistung von Ereignissen durchläuft und jeweils eine Tabellenzeile hinzufügt. Entfernen Sie `render json: @events` die Leitung aus `index` der Aktion `./app/controllers/calendar_controller.rb` in, und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.

![Screenshot der Ereignistabelle](./images/add-msgraph-01.png)