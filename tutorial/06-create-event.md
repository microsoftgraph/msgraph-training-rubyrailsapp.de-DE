<!-- markdownlint-disable MD002 MD041 -->

In diesem Abschnitt können Sie die Möglichkeit zum Erstellen von Ereignissen im Kalender des Benutzers hinzufügen.

1. Öffnen Sie **/App/Helpers/graph_helper. RB** , und fügen Sie die folgende Methode zur **Graph** -Klasse hinzu.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="CreateEventSnippet":::

1. Öffnen Sie **./app/Controllers/calendar_controller** , und fügen Sie der **CalendarController** -Klasse die folgende Route hinzu.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/calendar_controller.rb" id="CreateEventRouteSnippet":::

1. Öffnen Sie **./config/routes.RB** , und fügen Sie die neue Route hinzu.

    ```ruby
    post 'calendar/new', :to => 'calendar#create'
    ```

1. Öffnen Sie **./app/views/Calendar/new.html. Erb** , und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/new.html.erb" id="NewEventFormSnippet":::

1. Speichern Sie Ihre Änderungen, und aktualisieren Sie die app. Klicken Sie auf der Seite **Kalender** auf die Schaltfläche **Neues Ereignis** . Füllen Sie das Formular aus, und wählen Sie **Erstellen** aus, um ein neues Ereignis zu erstellen.

    ![Screenshot des neuen Ereignis Formulars](images/create-event-01.png)
