---
ms.openlocfilehash: a024fb533c552563da6c9179301e16a2e1d09d5f
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942165"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="dffea-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="dffea-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="dffea-102">Esto es necesario para obtener el token de acceso OAuth necesario para llamar a la API de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="dffea-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="dffea-103">En este paso configurará la [biblioteca Microsoft.Identity.Web.](https://www.nuget.org/packages/Microsoft.Identity.Web/)</span><span class="sxs-lookup"><span data-stu-id="dffea-103">In this step you will configure the [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) library.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="dffea-104">Para evitar almacenar el identificador de aplicación y el secreto en el origen, usará el Administrador de secretos [de .NET](/aspnet/core/security/app-secrets) para almacenar estos valores.</span><span class="sxs-lookup"><span data-stu-id="dffea-104">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="dffea-105">El Administrador de secretos es solo con fines de desarrollo, las aplicaciones de producción deben usar un administrador de secretos de confianza para almacenar secretos.</span><span class="sxs-lookup"><span data-stu-id="dffea-105">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

1. <span data-ttu-id="dffea-106">Abra **./appsettings.jsy** reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="dffea-106">Open **./appsettings.json** and replace its contents with the following.</span></span>

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. <span data-ttu-id="dffea-107">Abra la CLI en el directorio donde se encuentra **GraphTutorial.csproj** y ejecute los siguientes comandos, sustituyendo con su identificador de aplicación desde Azure Portal y con el secreto de `YOUR_APP_ID` `YOUR_APP_SECRET` aplicación.</span><span class="sxs-lookup"><span data-stu-id="dffea-107">Open your CLI in the directory where **GraphTutorial.csproj** is located, and run the following commands, substituting `YOUR_APP_ID` with your application ID from the Azure portal, and `YOUR_APP_SECRET` with your application secret.</span></span>

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="dffea-108">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="dffea-108">Implement sign-in</span></span>

<span data-ttu-id="dffea-109">Empiece por agregar los servicios de la plataforma de Microsoft Identity a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="dffea-109">Start by adding the Microsoft Identity platform services to the application.</span></span>

1. <span data-ttu-id="dffea-110">Cree un archivo denominado **GraphConstants.cs** en el directorio **./Graph** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="dffea-110">Create a new file named **GraphConstants.cs** in the **./Graph** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. <span data-ttu-id="dffea-111">Abra el **archivo ./Startup.cs** y agregue las siguientes `using` instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="dffea-111">Open the **./Startup.cs** file and add the following `using` statements to the top of the file.</span></span>

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

1. <span data-ttu-id="dffea-112">Reemplace la función `ConfigureServices` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="dffea-112">Replace the existing `ConfigureServices` function with the following.</span></span>

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

1. <span data-ttu-id="dffea-113">En la `Configure` función, agregue la siguiente línea por encima de la `app.UseAuthorization();` línea.</span><span class="sxs-lookup"><span data-stu-id="dffea-113">In the `Configure` function, add the following line above the `app.UseAuthorization();` line.</span></span>

    ```csharp
    app.UseAuthentication();
    ```

1. <span data-ttu-id="dffea-114">Abra **./Controllers/HomeController.cs** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="dffea-114">Open **./Controllers/HomeController.cs** and replace its contents with the following.</span></span>

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

1. <span data-ttu-id="dffea-115">Guarde los cambios e inicie el proyecto.</span><span class="sxs-lookup"><span data-stu-id="dffea-115">Save your changes and start the project.</span></span> <span data-ttu-id="dffea-116">Inicie sesión con su cuenta de Microsoft.</span><span class="sxs-lookup"><span data-stu-id="dffea-116">Login with your Microsoft account.</span></span>

1. <span data-ttu-id="dffea-117">Examine la solicitud de consentimiento.</span><span class="sxs-lookup"><span data-stu-id="dffea-117">Examine the consent prompt.</span></span> <span data-ttu-id="dffea-118">La lista de permisos corresponde a la lista de ámbitos de permisos configurados en **./Graph/GraphConstants.cs**.</span><span class="sxs-lookup"><span data-stu-id="dffea-118">The list of permissions correspond to list of permissions scopes configured in **./Graph/GraphConstants.cs**.</span></span>

    - <span data-ttu-id="dffea-119">**Mantenga el acceso a los datos** a los que le ha concedido acceso: ( ) MSAL solicita este permiso para recuperar `offline_access` tokens de actualización.</span><span class="sxs-lookup"><span data-stu-id="dffea-119">**Maintain access to data you have given it access to:** (`offline_access`) This permission is requested by MSAL in order to retrieve refresh tokens.</span></span>
    - <span data-ttu-id="dffea-120">**Inicie sesión y lea su perfil:** ( ) Este permiso permite a la aplicación obtener el perfil y la foto de perfil del usuario que ha iniciado `User.Read` sesión.</span><span class="sxs-lookup"><span data-stu-id="dffea-120">**Sign you in and read your profile:** (`User.Read`) This permission allows the app to get the logged-in user's profile and profile photo.</span></span>
    - <span data-ttu-id="dffea-121">**Lea la configuración del buzón:** ( ) Este permiso permite a la aplicación leer la configuración del buzón del usuario, incluido el formato de zona `MailboxSettings.Read` horaria y hora.</span><span class="sxs-lookup"><span data-stu-id="dffea-121">**Read your mailbox settings:** (`MailboxSettings.Read`) This permission allows the app to read the user's mailbox settings, including time zone and time format.</span></span>
    - <span data-ttu-id="dffea-122">Tener acceso completo a los **calendarios:** ( ) Este permiso permite a la aplicación leer eventos en el calendario del usuario, agregar nuevos eventos y modificar los `Calendars.ReadWrite` existentes.</span><span class="sxs-lookup"><span data-stu-id="dffea-122">**Have full access to your calendars:** (`Calendars.ReadWrite`) This permission allows the app to read events on the user's calendar, add new events, and modify existing ones.</span></span>

    ![Captura de pantalla del aviso de consentimiento de la plataforma de identidad de Microsoft](./images/add-aad-auth-03.png)

    <span data-ttu-id="dffea-124">Para obtener más información sobre el consentimiento, consulte [Descripción de las experiencias de consentimiento de aplicaciones de Azure AD.](/azure/active-directory/develop/application-consent-experience)</span><span class="sxs-lookup"><span data-stu-id="dffea-124">For more information regarding consent, see [Understanding Azure AD application consent experiences](/azure/active-directory/develop/application-consent-experience).</span></span>

1. <span data-ttu-id="dffea-125">Consentimiento para los permisos solicitados.</span><span class="sxs-lookup"><span data-stu-id="dffea-125">Consent to the requested permissions.</span></span> <span data-ttu-id="dffea-126">El explorador redirige a la aplicación y muestra el token.</span><span class="sxs-lookup"><span data-stu-id="dffea-126">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="dffea-127">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="dffea-127">Get user details</span></span>

<span data-ttu-id="dffea-128">Una vez que el usuario haya iniciado sesión, puede obtener su información de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="dffea-128">Once the user is logged in, you can get their information from Microsoft Graph.</span></span>

1. <span data-ttu-id="dffea-129">Abra **./Graph/GraphClaimsPrincipalExtensions.cs** y reemplace todo su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="dffea-129">Open **./Graph/GraphClaimsPrincipalExtensions.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. <span data-ttu-id="dffea-130">Abra **./Startup.cs** y reemplace la línea `.AddMicrosoftIdentityWebApp(Configuration)` existente por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="dffea-130">Open **./Startup.cs** and replace the existing `.AddMicrosoftIdentityWebApp(Configuration)` line with the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    <span data-ttu-id="dffea-131">Ten en cuenta lo que hace este código.</span><span class="sxs-lookup"><span data-stu-id="dffea-131">Consider what this code does.</span></span>

    - <span data-ttu-id="dffea-132">Agrega un controlador de eventos para el `OnTokenValidated` evento.</span><span class="sxs-lookup"><span data-stu-id="dffea-132">It adds an event handler for the `OnTokenValidated` event.</span></span>
        - <span data-ttu-id="dffea-133">Usa la interfaz `ITokenAcquisition` para obtener un token de acceso.</span><span class="sxs-lookup"><span data-stu-id="dffea-133">It uses the `ITokenAcquisition` interface to get an access token.</span></span>
        - <span data-ttu-id="dffea-134">Llama a Microsoft Graph para obtener el perfil y la foto del usuario.</span><span class="sxs-lookup"><span data-stu-id="dffea-134">It calls Microsoft Graph to get the user's profile and photo.</span></span>
        - <span data-ttu-id="dffea-135">Agrega la información de Graph a la identidad del usuario.</span><span class="sxs-lookup"><span data-stu-id="dffea-135">It adds the Graph information to the user's identity.</span></span>

1. <span data-ttu-id="dffea-136">Agregue la siguiente llamada de función después de `EnableTokenAcquisitionToCallDownstreamApi` la llamada y antes de la `AddInMemoryTokenCaches` llamada.</span><span class="sxs-lookup"><span data-stu-id="dffea-136">Add the following function call after the `EnableTokenAcquisitionToCallDownstreamApi` call and before the `AddInMemoryTokenCaches` call.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    <span data-ttu-id="dffea-137">Esto hará que **graphServiceClient** autenticado esté disponible para los controladores a través de la inserción de dependencias.</span><span class="sxs-lookup"><span data-stu-id="dffea-137">This will make an authenticated **GraphServiceClient** available to controllers via dependency injection.</span></span>

1. <span data-ttu-id="dffea-138">Abra **./Controllers/HomeController.cs** y reemplace la `Index` función por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="dffea-138">Open **./Controllers/HomeController.cs** and replace the `Index` function with the following.</span></span>

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. <span data-ttu-id="dffea-139">Quite todas las referencias `ITokenAcquisition` a la clase **HomeController.**</span><span class="sxs-lookup"><span data-stu-id="dffea-139">Remove all references to `ITokenAcquisition` in the **HomeController** class.</span></span>

1. <span data-ttu-id="dffea-140">Guarde los cambios, inicie la aplicación y realice el proceso de inicio de sesión.</span><span class="sxs-lookup"><span data-stu-id="dffea-140">Save your changes, start the app, and go through the sign-in process.</span></span> <span data-ttu-id="dffea-141">Debería volver a la página principal, pero la interfaz de usuario debe cambiar para indicar que ha iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="dffea-141">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Captura de pantalla de la página principal después de iniciar sesión](./images/add-aad-auth-01.png)

1. <span data-ttu-id="dffea-143">Haz clic en el avatar del usuario en la esquina superior derecha para acceder al vínculo **Cerrar** sesión.</span><span class="sxs-lookup"><span data-stu-id="dffea-143">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="dffea-144">Al **hacer clic en** Cerrar sesión, se restablece la sesión y se vuelve a la página principal.</span><span class="sxs-lookup"><span data-stu-id="dffea-144">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Captura de pantalla del menú desplegable con el vínculo Cerrar sesión](./images/add-aad-auth-02.png)

> [!TIP]
> <span data-ttu-id="dffea-146">Si no ve el nombre de usuario en la página principal y falta el nombre y el correo electrónico del cuadro desplegable de avatares de uso después de realizar estos cambios, vuelva a cerrar sesión y cerrar sesión.</span><span class="sxs-lookup"><span data-stu-id="dffea-146">If you do not see your user name on the home page and the use avatar dropdown is missing name and email after making these changes, sign out and sign back in.</span></span>

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="dffea-147">Almacenar y actualizar tokens</span><span class="sxs-lookup"><span data-stu-id="dffea-147">Storing and refreshing tokens</span></span>

<span data-ttu-id="dffea-148">En este momento, la aplicación tiene un token de acceso, que se envía en el `Authorization` encabezado de las llamadas API.</span><span class="sxs-lookup"><span data-stu-id="dffea-148">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="dffea-149">Este es el token que permite que la aplicación acceda a Microsoft Graph en nombre del usuario.</span><span class="sxs-lookup"><span data-stu-id="dffea-149">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="dffea-150">Sin embargo, este token es de corta duración.</span><span class="sxs-lookup"><span data-stu-id="dffea-150">However, this token is short-lived.</span></span> <span data-ttu-id="dffea-151">El token expira una hora después de su emisión.</span><span class="sxs-lookup"><span data-stu-id="dffea-151">The token expires an hour after it is issued.</span></span> <span data-ttu-id="dffea-152">Aquí es donde el token de actualización resulta útil.</span><span class="sxs-lookup"><span data-stu-id="dffea-152">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="dffea-153">El token de actualización permite a la aplicación solicitar un nuevo token de acceso sin necesidad de que el usuario vuelva a iniciar sesión.</span><span class="sxs-lookup"><span data-stu-id="dffea-153">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="dffea-154">Dado que la aplicación usa la biblioteca Microsoft.Identity.Web, no es necesario implementar ningún almacenamiento de tokens ni lógica de actualización.</span><span class="sxs-lookup"><span data-stu-id="dffea-154">Because the app is using the Microsoft.Identity.Web library, you do not have to implement any token storage or refresh logic.</span></span>

<span data-ttu-id="dffea-155">La aplicación usa la memoria caché de tokens en memoria, lo que es suficiente para las aplicaciones que no necesitan conservar tokens cuando se reinicia la aplicación.</span><span class="sxs-lookup"><span data-stu-id="dffea-155">The app uses the in-memory token cache, which is sufficient for apps that do not need to persist tokens when the app restarts.</span></span> <span data-ttu-id="dffea-156">En su lugar, las aplicaciones de producción pueden usar las opciones de caché [distribuida](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) en la biblioteca Microsoft.Identity.Web.</span><span class="sxs-lookup"><span data-stu-id="dffea-156">Production apps may instead use the [distributed cache options](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in the Microsoft.Identity.Web library.</span></span>

<span data-ttu-id="dffea-157">El `GetAccessTokenForUserAsync` método controla la expiración del token y la actualización automáticamente.</span><span class="sxs-lookup"><span data-stu-id="dffea-157">The `GetAccessTokenForUserAsync` method handles token expiration and refresh for you.</span></span> <span data-ttu-id="dffea-158">Primero comprueba el token almacenado en caché y, si no ha expirado, lo devuelve.</span><span class="sxs-lookup"><span data-stu-id="dffea-158">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="dffea-159">Si ha expirado, usa el token de actualización en caché para obtener uno nuevo.</span><span class="sxs-lookup"><span data-stu-id="dffea-159">If it is expired, it uses the cached refresh token to obtain a new one.</span></span>

<span data-ttu-id="dffea-160">**GraphServiceClient que obtienen** los controladores a través de la inserción de dependencias se configurará previamente con un proveedor de autenticación `GetAccessTokenForUserAsync` que lo use automáticamente.</span><span class="sxs-lookup"><span data-stu-id="dffea-160">The **GraphServiceClient** that controllers get via dependency injection will be pre-configured with an authentication provider that uses `GetAccessTokenForUserAsync` for you.</span></span>
