---
ms.openlocfilehash: c954903f38d48bcca4c534f4d0cfbe1605cbf7f6
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942151"
---
<!-- markdownlint-disable MD002 MD041 -->

En esta sección incorporará Microsoft Graph a la aplicación. Para esta aplicación, usará la biblioteca cliente de [Microsoft Graph para .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para realizar llamadas a Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obtener eventos de calendario de Outlook

Empiece por crear un controlador nuevo para las vistas de calendario.

1. Agrega un nuevo archivo denominado **CalendarController.cs** en el directorio **./Controllers** y agrega el siguiente código.

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using Microsoft.Graph;
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using TimeZoneConverter;

    namespace GraphTutorial.Controllers
    {
        public class CalendarController : Controller
        {
            private readonly GraphServiceClient _graphClient;
            private readonly ILogger<HomeController> _logger;

            public CalendarController(
                GraphServiceClient graphClient,
                ILogger<HomeController> logger)
            {
                _graphClient = graphClient;
                _logger = logger;
            }
        }
    }
    ```

1. Agregue las siguientes funciones a la `CalendarController` clase para obtener la vista de calendario del usuario.

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetCalendarViewSnippet":::

    Ten en cuenta lo que hace el `GetUserWeekCalendar` código.

    - Usa la zona horaria del usuario para obtener los valores de fecha y hora de inicio y finalización UTC de la semana.
    - Consulta la vista de calendario del usuario [para](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) obtener todos los eventos que se encuentra entre la fecha y hora de inicio y finalización. El uso de una vista de calendario en lugar de enumerar eventos [expande](/graph/api/user-list-events?view=graph-rest-1.0) los eventos periódicos y devuelve las repeticiones que se produzcan en la ventana de tiempo especificada.
    - Usa el encabezado `Prefer: outlook.timezone` para obtener resultados en la zona horaria del usuario.
    - Se usa `Select` para limitar los campos que regresan solo a los que usa la aplicación.
    - Se usa `OrderBy` para ordenar los resultados cronológicamente.
    - Usa una página `PageIterator` para a través de la colección de [eventos](/graph/sdks/paging). Esto controla el caso en el que el usuario tiene más eventos en su calendario que el tamaño de página solicitado.

1. Agregue la siguiente función a la `CalendarController` clase para implementar una vista temporal de los datos devueltos.

    ```csharp
    // Minimum permission scope needed for this view
    [AuthorizeForScopes(Scopes = new[] { "Calendars.Read" })]
    public async Task<IActionResult> Index()
    {
        try
        {
            var userTimeZone = TZConvert.GetTimeZoneInfo(
                User.GetUserGraphTimeZone());
            var startOfWeek = CalendarController.GetUtcStartOfWeekInTimeZone(
                DateTime.Today, userTimeZone);

            var events = await GetUserWeekCalendar(startOfWeek);

            // Return a JSON dump of events
            return new ContentResult {
                Content = _graphClient.HttpProvider.Serializer.SerializeObject(events),
                ContentType = "application/json"
            };
        }
        catch (ServiceException ex)
        {
            if (ex.InnerException is MicrosoftIdentityWebChallengeUserException)
            {
                throw;
            }

            return new ContentResult {
                Content = $"Error getting calendar view: {ex.Message}",
                ContentType = "text/plain"
            };
        }
    }
    ```

1. Inicie la aplicación, inicie sesión y haga clic en el vínculo **Calendario** de la barra de navegación. Si todo funciona, debería ver un volcado JSON de eventos en el calendario del usuario.

## <a name="display-the-results"></a>Mostrar los resultados

Ahora puede agregar una vista para mostrar los resultados de una manera más fácil de usar.

### <a name="create-view-models"></a>Crear modelos de vista

1. Cree un archivo denominado **CalendarViewEvent.cs** en el directorio **./Models** y agregue el siguiente código.

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewEvent.cs" id="CalendarViewEventSnippet":::

1. Cree un archivo denominado **DailyViewModel.cs** en el directorio **./Models** y agregue el siguiente código.

    :::code language="csharp" source="../demo/GraphTutorial/Models/DailyViewModel.cs" id="DailyViewModelSnippet":::

1. Cree un archivo denominado **CalendarViewModel.cs** en el directorio **./Models** y agregue el siguiente código.

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewModel.cs" id="CalendarViewModelSnippet":::

### <a name="create-views"></a>Crear vistas

1. Cree un directorio denominado **Calendario** en el directorio **./Views.**

1. Cree un archivo denominado **_DailyEventsPartial.cshtml** en el directorio **./Views/Calendar** y agregue el siguiente código.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/_DailyEventsPartial.cshtml" id="DailyEventsPartialSnippet":::

1. Cree un nuevo archivo denominado **Index.cshtml** en el directorio **./Views/Calendar** y agregue el siguiente código.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/Index.cshtml" id="CalendarIndexSnippet":::

### <a name="update-calendar-controller"></a>Actualizar controlador de calendario

1. Abra **./Controllers/CalendarController.cs** y reemplace la función `Index` existente por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="IndexSnippet":::

1. Inicia la aplicación, inicia sesión y haz clic en el **vínculo** Calendario. La aplicación ahora debe representar una tabla de eventos.

    ![Captura de pantalla de la tabla de eventos](./images/add-msgraph-01.png)
