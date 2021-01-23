---
ms.openlocfilehash: a024fb533c552563da6c9179301e16a2e1d09d5f
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942165"
---
<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD. Esto es necesario para obtener el token de acceso OAuth necesario para llamar a la API de Microsoft Graph. En este paso configurará la [biblioteca Microsoft.Identity.Web.](https://www.nuget.org/packages/Microsoft.Identity.Web/)

> [!IMPORTANT]
> Para evitar almacenar el identificador de aplicación y el secreto en el origen, usará el Administrador de secretos [de .NET](/aspnet/core/security/app-secrets) para almacenar estos valores. El Administrador de secretos es solo con fines de desarrollo, las aplicaciones de producción deben usar un administrador de secretos de confianza para almacenar secretos.

1. Abra **./appsettings.jsy** reemplace su contenido por lo siguiente.

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. Abra la CLI en el directorio donde se encuentra **GraphTutorial.csproj** y ejecute los siguientes comandos, sustituyendo con su identificador de aplicación desde Azure Portal y con el secreto de `YOUR_APP_ID` `YOUR_APP_SECRET` aplicación.

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a>Implementar el inicio de sesión

Empiece por agregar los servicios de la plataforma de Microsoft Identity a la aplicación.

1. Cree un archivo denominado **GraphConstants.cs** en el directorio **./Graph** y agregue el siguiente código.

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. Abra el **archivo ./Startup.cs** y agregue las siguientes `using` instrucciones en la parte superior del archivo.

    ```csharp
    using Microsoft.AspNetCore.Authentication.OpenIdConnect;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc.Authorization;
    using Microsoft.Identity.Web;
    using Microsoft.Identity.Web.UI;
    using Microsoft.IdentityModel.Protocols.OpenIdConnect;
    using Microsoft.Graph;
    using System.Net;
    using System.Net.Http.Headers;
    ```

1. Reemplace la función `ConfigureServices` existente por lo siguiente.

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services
            // Use OpenId authentication
            .AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
            // Specify this is a web app and needs auth code flow
            .AddMicrosoftIdentityWebApp(Configuration)
            // Add ability to call web API (Graph)
            // and get access tokens
            .EnableTokenAcquisitionToCallDownstreamApi(options => {
                Configuration.Bind("AzureAd", options);
            }, GraphConstants.Scopes)
            // Use in-memory token cache
            // See https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization
            .AddInMemoryTokenCaches();

        // Require authentication
        services.AddControllersWithViews(options =>
        {
            var policy = new AuthorizationPolicyBuilder()
                .RequireAuthenticatedUser()
                .Build();
            options.Filters.Add(new AuthorizeFilter(policy));
        })
        // Add the Microsoft Identity UI pages for signin/out
        .AddMicrosoftIdentityUI();
    }
    ```

1. En la `Configure` función, agregue la siguiente línea por encima de la `app.UseAuthorization();` línea.

    ```csharp
    app.UseAuthentication();
    ```

1. Abra **./Controllers/HomeController.cs** y reemplace su contenido por lo siguiente.

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using System.Diagnostics;
    using System.Threading.Tasks;

    namespace GraphTutorial.Controllers
    {
        public class HomeController : Controller
        {
            ITokenAcquisition _tokenAcquisition;
            private readonly ILogger<HomeController> _logger;

            // Get the ITokenAcquisition interface via
            // dependency injection
            public HomeController(
                ITokenAcquisition tokenAcquisition,
                ILogger<HomeController> logger)
            {
                _tokenAcquisition = tokenAcquisition;
                _logger = logger;
            }

            public async Task<IActionResult> Index()
            {
                // TEMPORARY
                // Get the token and display it
                try
                {
                    string token = await _tokenAcquisition
                        .GetAccessTokenForUserAsync(GraphConstants.Scopes);
                    return View().WithInfo("Token acquired", token);
                }
                catch (MicrosoftIdentityWebChallengeUserException)
                {
                    return Challenge();
                }
            }

            public IActionResult Privacy()
            {
                return View();
            }

            [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
            public IActionResult Error()
            {
                return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
            }

            [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
            [AllowAnonymous]
            public IActionResult ErrorWithMessage(string message, string debug)
            {
                return View("Index").WithError(message, debug);
            }
        }
    }
    ```

1. Guarde los cambios e inicie el proyecto. Inicie sesión con su cuenta de Microsoft.

1. Examine la solicitud de consentimiento. La lista de permisos corresponde a la lista de ámbitos de permisos configurados en **./Graph/GraphConstants.cs**.

    - **Mantenga el acceso a los datos** a los que le ha concedido acceso: ( ) MSAL solicita este permiso para recuperar `offline_access` tokens de actualización.
    - **Inicie sesión y lea su perfil:** ( ) Este permiso permite a la aplicación obtener el perfil y la foto de perfil del usuario que ha iniciado `User.Read` sesión.
    - **Lea la configuración del buzón:** ( ) Este permiso permite a la aplicación leer la configuración del buzón del usuario, incluido el formato de zona `MailboxSettings.Read` horaria y hora.
    - Tener acceso completo a los **calendarios:** ( ) Este permiso permite a la aplicación leer eventos en el calendario del usuario, agregar nuevos eventos y modificar los `Calendars.ReadWrite` existentes.

    ![Captura de pantalla del aviso de consentimiento de la plataforma de identidad de Microsoft](./images/add-aad-auth-03.png)

    Para obtener más información sobre el consentimiento, consulte [Descripción de las experiencias de consentimiento de aplicaciones de Azure AD.](/azure/active-directory/develop/application-consent-experience)

1. Consentimiento para los permisos solicitados. El explorador redirige a la aplicación y muestra el token.

### <a name="get-user-details"></a>Obtener detalles del usuario

Una vez que el usuario haya iniciado sesión, puede obtener su información de Microsoft Graph.

1. Abra **./Graph/GraphClaimsPrincipalExtensions.cs** y reemplace todo su contenido por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. Abra **./Startup.cs** y reemplace la línea `.AddMicrosoftIdentityWebApp(Configuration)` existente por el siguiente código.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    Ten en cuenta lo que hace este código.

    - Agrega un controlador de eventos para el `OnTokenValidated` evento.
        - Usa la interfaz `ITokenAcquisition` para obtener un token de acceso.
        - Llama a Microsoft Graph para obtener el perfil y la foto del usuario.
        - Agrega la información de Graph a la identidad del usuario.

1. Agregue la siguiente llamada de función después de `EnableTokenAcquisitionToCallDownstreamApi` la llamada y antes de la `AddInMemoryTokenCaches` llamada.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    Esto hará que **graphServiceClient** autenticado esté disponible para los controladores a través de la inserción de dependencias.

1. Abra **./Controllers/HomeController.cs** y reemplace la `Index` función por lo siguiente.

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. Quite todas las referencias `ITokenAcquisition` a la clase **HomeController.**

1. Guarde los cambios, inicie la aplicación y realice el proceso de inicio de sesión. Debería volver a la página principal, pero la interfaz de usuario debe cambiar para indicar que ha iniciado sesión.

    ![Captura de pantalla de la página principal después de iniciar sesión](./images/add-aad-auth-01.png)

1. Haz clic en el avatar del usuario en la esquina superior derecha para acceder al vínculo **Cerrar** sesión. Al **hacer clic en** Cerrar sesión, se restablece la sesión y se vuelve a la página principal.

    ![Captura de pantalla del menú desplegable con el vínculo Cerrar sesión](./images/add-aad-auth-02.png)

> [!TIP]
> Si no ve el nombre de usuario en la página principal y falta el nombre y el correo electrónico del cuadro desplegable de avatares de uso después de realizar estos cambios, vuelva a cerrar sesión y cerrar sesión.

## <a name="storing-and-refreshing-tokens"></a>Almacenar y actualizar tokens

En este momento, la aplicación tiene un token de acceso, que se envía en el `Authorization` encabezado de las llamadas API. Este es el token que permite que la aplicación acceda a Microsoft Graph en nombre del usuario.

Sin embargo, este token es de corta duración. El token expira una hora después de su emisión. Aquí es donde el token de actualización resulta útil. El token de actualización permite a la aplicación solicitar un nuevo token de acceso sin necesidad de que el usuario vuelva a iniciar sesión.

Dado que la aplicación usa la biblioteca Microsoft.Identity.Web, no es necesario implementar ningún almacenamiento de tokens ni lógica de actualización.

La aplicación usa la memoria caché de tokens en memoria, lo que es suficiente para las aplicaciones que no necesitan conservar tokens cuando se reinicia la aplicación. En su lugar, las aplicaciones de producción pueden usar las opciones de caché [distribuida](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) en la biblioteca Microsoft.Identity.Web.

El `GetAccessTokenForUserAsync` método controla la expiración del token y la actualización automáticamente. Primero comprueba el token almacenado en caché y, si no ha expirado, lo devuelve. Si ha expirado, usa el token de actualización en caché para obtener uno nuevo.

**GraphServiceClient que obtienen** los controladores a través de la inserción de dependencias se configurará previamente con un proveedor de autenticación `GetAccessTokenForUserAsync` que lo use automáticamente.
