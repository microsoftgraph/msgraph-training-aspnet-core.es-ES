---
ms.openlocfilehash: 5b1a776c28b6f9218c713dde68f45e571ebfd999
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942158"
---
<!-- markdownlint-disable MD002 MD041 -->

Empiece por crear una ASP.NET web principal.

1. Abra la interfaz de línea de comandos (CLI) en un directorio donde quiera crear el proyecto. Ejecute el comando siguiente.

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. Una vez creado el proyecto, compruebe que funciona cambiando el directorio actual al directorio **GraphTutorial** y ejecutando el siguiente comando en la CLI.

    ```Shell
    dotnet run
    ```

1. Abra el explorador y vaya a `https://localhost:5001` . Si todo funciona, debería ver una página ASP.NET principal predeterminada.

> [!IMPORTANT]
> Si recibe una advertencia de que el certificado de **localhost** no es de confianza, puede usar la CLI de .NET Core para instalar y confiar en el certificado de desarrollo. Consulta [Aplicar HTTPS en ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) para obtener instrucciones para sistemas operativos específicos.

## <a name="add-nuget-packages"></a>Agregar paquetes NuGet

Antes de seguir adelante, instala algunos paquetes NuGet adicionales que usarás más adelante.

- [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) para solicitar y administrar tokens de acceso.
- [Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) para agregar el SDK de Microsoft Graph mediante inserción de dependencias.
- [Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) para la interfaz de usuario de inicio y de salida.
- [TimeZoneConverter para](https://github.com/mj1856/TimeZoneConverter) controlar identificadores de zona horaria multiplataforma.

1. Ejecute los siguientes comandos en la CLI para instalar las dependencias.

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.5.1
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.5.1
    dotnet add package Microsoft.Identity.Web.UI --version 1.5.1
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a>Diseñar la aplicación

En esta sección crearás la estructura básica de la interfaz de usuario de la aplicación.

### <a name="implement-alert-extension-methods"></a>Implementar métodos de extensión de alerta

En esta sección creará métodos de extensión para el `IActionResult` tipo devuelto por las vistas de controlador. Esta extensión permitirá pasar mensajes temporales de error o de éxito a la vista.

> [!TIP]
> Puede usar cualquier editor de texto para editar los archivos de origen de este tutorial. Sin embargo, [Visual Studio código](https://code.visualstudio.com/) proporciona características adicionales, como depuración e IntelliSense.

1. Cree un nuevo directorio en el **directorio GraphTutorial** denominado **Alerts**.

1. Crea un nuevo archivo denominado **WithAlertResult.cs** en el directorio **./Alerts** y agrega el siguiente código.

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. Crea un nuevo archivo denominado **AlertExtensions.cs** en el directorio **./Alerts** y agrega el siguiente código.

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a>Implementar métodos de extensión de datos de usuario

En esta sección creará métodos de extensión para el `ClaimsPrincipal` objeto generado por la plataforma Microsoft Identity. Esto le permitirá ampliar la identidad de usuario existente con datos de Microsoft Graph.

> [!NOTE]
> Este código es solo un marcador de posición por ahora, lo completará en una sección posterior.

1. Cree un nuevo directorio en el **directorio GraphTutorial** denominado **Graph**.

1. Cree un archivo denominado **GraphClaimsPrincipalExtensions.cs** y agregue el siguiente código.

    ```csharp
    using System.Security.Claims;

    namespace GraphTutorial
    {
        public static class GraphClaimTypes {
            public const string DisplayName ="graph_name";
            public const string Email = "graph_email";
            public const string Photo = "graph_photo";
            public const string TimeZone = "graph_timezone";
            public const string DateTimeFormat = "graph_datetimeformat";
        }

        // Helper methods to access Graph user data stored in
        // the claims principal
        public static class GraphClaimsPrincipalExtensions
        {
            public static string GetUserGraphDisplayName(this ClaimsPrincipal claimsPrincipal)
            {
                return "Adele Vance";
            }

            public static string GetUserGraphEmail(this ClaimsPrincipal claimsPrincipal)
            {
                return "adelev@contoso.com";
            }

            public static string GetUserGraphPhoto(this ClaimsPrincipal claimsPrincipal)
            {
                return "/img/no-profile-photo.png";
            }
        }
    }
    ```

### <a name="create-views"></a>Crear vistas

En esta sección, implementará las vistas Desenlazadas para la aplicación.

1. Agregue un nuevo archivo denominado **_LoginPartial.cshtml** en el directorio **./Views/Shared** y agregue el siguiente código.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. Agregue un nuevo archivo denominado **_AlertPartial.cshtml** en el directorio **./Views/Shared** y agregue el siguiente código.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. Abra el **archivo ./Views/Shared/_Layout.cshtml** y reemplace todo su contenido por el código siguiente para actualizar el diseño global de la aplicación.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. Abra **./wwwroot/css/site.css** y agregue el siguiente código en la parte inferior del archivo.

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. Abra el **archivo ./Views/Home/index.cshtml** y reemplace su contenido por lo siguiente.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. Cree un nuevo directorio en el directorio **./wwwroot** denominado **img**. Agregue un archivo de imagen que elija con el nombre **no-profile-photo.png** en este directorio. Esta imagen se usará como foto del usuario cuando el usuario no tenga ninguna foto en Microsoft Graph.

    > [!TIP]
    > Puede descargar la imagen usada en estas capturas de pantalla desde [GitHub.](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png)

1. Guarde todos los cambios y reinicie el servidor ( `dotnet run` ). Ahora, la aplicación debería tener un aspecto muy diferente.

    ![Captura de pantalla de la página principal rediseñada](./images/create-app-01.png)
