---
ms.openlocfilehash: 5b1a776c28b6f9218c713dde68f45e571ebfd999
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942158"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="77657-101">Empiece por crear una ASP.NET web principal.</span><span class="sxs-lookup"><span data-stu-id="77657-101">Start by creating an ASP.NET Core web app.</span></span>

1. <span data-ttu-id="77657-102">Abra la interfaz de línea de comandos (CLI) en un directorio donde quiera crear el proyecto.</span><span class="sxs-lookup"><span data-stu-id="77657-102">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="77657-103">Ejecute el comando siguiente.</span><span class="sxs-lookup"><span data-stu-id="77657-103">Run the following command.</span></span>

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. <span data-ttu-id="77657-104">Una vez creado el proyecto, compruebe que funciona cambiando el directorio actual al directorio **GraphTutorial** y ejecutando el siguiente comando en la CLI.</span><span class="sxs-lookup"><span data-stu-id="77657-104">Once the project is created, verify that it works by changing the current directory to the **GraphTutorial** directory and running the following command in your CLI.</span></span>

    ```Shell
    dotnet run
    ```

1. <span data-ttu-id="77657-105">Abra el explorador y vaya a `https://localhost:5001` .</span><span class="sxs-lookup"><span data-stu-id="77657-105">Open your browser and browse to `https://localhost:5001`.</span></span> <span data-ttu-id="77657-106">Si todo funciona, debería ver una página ASP.NET principal predeterminada.</span><span class="sxs-lookup"><span data-stu-id="77657-106">If everything is working, you should see a default ASP.NET Core page.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="77657-107">Si recibe una advertencia de que el certificado de **localhost** no es de confianza, puede usar la CLI de .NET Core para instalar y confiar en el certificado de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="77657-107">If you receive a warning that the certificate for **localhost** is un-trusted you can use the .NET Core CLI to install and trust the development certificate.</span></span> <span data-ttu-id="77657-108">Consulta [Aplicar HTTPS en ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) para obtener instrucciones para sistemas operativos específicos.</span><span class="sxs-lookup"><span data-stu-id="77657-108">See [Enforce HTTPS in ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) for instructions for specific operating systems.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="77657-109">Agregar paquetes NuGet</span><span class="sxs-lookup"><span data-stu-id="77657-109">Add NuGet packages</span></span>

<span data-ttu-id="77657-110">Antes de seguir adelante, instala algunos paquetes NuGet adicionales que usarás más adelante.</span><span class="sxs-lookup"><span data-stu-id="77657-110">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="77657-111">[Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) para solicitar y administrar tokens de acceso.</span><span class="sxs-lookup"><span data-stu-id="77657-111">[Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) for requesting and managing access tokens.</span></span>
- <span data-ttu-id="77657-112">[Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) para agregar el SDK de Microsoft Graph mediante inserción de dependencias.</span><span class="sxs-lookup"><span data-stu-id="77657-112">[Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) for adding the Microsoft Graph SDK via dependency injection.</span></span>
- <span data-ttu-id="77657-113">[Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) para la interfaz de usuario de inicio y de salida.</span><span class="sxs-lookup"><span data-stu-id="77657-113">[Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) for sign-in and sign-out UI.</span></span>
- <span data-ttu-id="77657-114">[TimeZoneConverter para](https://github.com/mj1856/TimeZoneConverter) controlar identificadores de zona horaria multiplataforma.</span><span class="sxs-lookup"><span data-stu-id="77657-114">[TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter) for handling time zoned identifiers cross-platform.</span></span>

1. <span data-ttu-id="77657-115">Ejecute los siguientes comandos en la CLI para instalar las dependencias.</span><span class="sxs-lookup"><span data-stu-id="77657-115">Run the following commands in your CLI to install the dependencies.</span></span>

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.5.1
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.5.1
    dotnet add package Microsoft.Identity.Web.UI --version 1.5.1
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a><span data-ttu-id="77657-116">Diseñar la aplicación</span><span class="sxs-lookup"><span data-stu-id="77657-116">Design the app</span></span>

<span data-ttu-id="77657-117">En esta sección crearás la estructura básica de la interfaz de usuario de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="77657-117">In this section you will create the basic UI structure of the application.</span></span>

### <a name="implement-alert-extension-methods"></a><span data-ttu-id="77657-118">Implementar métodos de extensión de alerta</span><span class="sxs-lookup"><span data-stu-id="77657-118">Implement alert extension methods</span></span>

<span data-ttu-id="77657-119">En esta sección creará métodos de extensión para el `IActionResult` tipo devuelto por las vistas de controlador.</span><span class="sxs-lookup"><span data-stu-id="77657-119">In this section you will create extension methods for the `IActionResult` type returned by controller views.</span></span> <span data-ttu-id="77657-120">Esta extensión permitirá pasar mensajes temporales de error o de éxito a la vista.</span><span class="sxs-lookup"><span data-stu-id="77657-120">This extension will enable passing temporary error or success messages to the view.</span></span>

> [!TIP]
> <span data-ttu-id="77657-121">Puede usar cualquier editor de texto para editar los archivos de origen de este tutorial.</span><span class="sxs-lookup"><span data-stu-id="77657-121">You can use any text editor to edit the source files for this tutorial.</span></span> <span data-ttu-id="77657-122">Sin embargo, [Visual Studio código](https://code.visualstudio.com/) proporciona características adicionales, como depuración e IntelliSense.</span><span class="sxs-lookup"><span data-stu-id="77657-122">However, [Visual Studio Code](https://code.visualstudio.com/) provides additional features, such as debugging and Intellisense.</span></span>

1. <span data-ttu-id="77657-123">Cree un nuevo directorio en el **directorio GraphTutorial** denominado **Alerts**.</span><span class="sxs-lookup"><span data-stu-id="77657-123">Create a new directory in the **GraphTutorial** directory named **Alerts**.</span></span>

1. <span data-ttu-id="77657-124">Crea un nuevo archivo denominado **WithAlertResult.cs** en el directorio **./Alerts** y agrega el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="77657-124">Create a new file named **WithAlertResult.cs** in the **./Alerts** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. <span data-ttu-id="77657-125">Crea un nuevo archivo denominado **AlertExtensions.cs** en el directorio **./Alerts** y agrega el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="77657-125">Create a new file named **AlertExtensions.cs** in the **./Alerts** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a><span data-ttu-id="77657-126">Implementar métodos de extensión de datos de usuario</span><span class="sxs-lookup"><span data-stu-id="77657-126">Implement user data extension methods</span></span>

<span data-ttu-id="77657-127">En esta sección creará métodos de extensión para el `ClaimsPrincipal` objeto generado por la plataforma Microsoft Identity.</span><span class="sxs-lookup"><span data-stu-id="77657-127">In this section you will create extension methods for the `ClaimsPrincipal` object generated by the Microsoft Identity platform.</span></span> <span data-ttu-id="77657-128">Esto le permitirá ampliar la identidad de usuario existente con datos de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="77657-128">This will allow you to extend the existing user identity with data from Microsoft Graph.</span></span>

> [!NOTE]
> <span data-ttu-id="77657-129">Este código es solo un marcador de posición por ahora, lo completará en una sección posterior.</span><span class="sxs-lookup"><span data-stu-id="77657-129">This code is just a placeholder for now, you will complete it in a later section.</span></span>

1. <span data-ttu-id="77657-130">Cree un nuevo directorio en el **directorio GraphTutorial** denominado **Graph**.</span><span class="sxs-lookup"><span data-stu-id="77657-130">Create a new directory in the **GraphTutorial** directory named **Graph**.</span></span>

1. <span data-ttu-id="77657-131">Cree un archivo denominado **GraphClaimsPrincipalExtensions.cs** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="77657-131">Create a new file named **GraphClaimsPrincipalExtensions.cs** and add the following code.</span></span>

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

### <a name="create-views"></a><span data-ttu-id="77657-132">Crear vistas</span><span class="sxs-lookup"><span data-stu-id="77657-132">Create views</span></span>

<span data-ttu-id="77657-133">En esta sección, implementará las vistas Desenlazadas para la aplicación.</span><span class="sxs-lookup"><span data-stu-id="77657-133">In this section you will implement the Razor views for the application.</span></span>

1. <span data-ttu-id="77657-134">Agregue un nuevo archivo denominado **_LoginPartial.cshtml** en el directorio **./Views/Shared** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="77657-134">Add a new file named **_LoginPartial.cshtml** in the **./Views/Shared** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. <span data-ttu-id="77657-135">Agregue un nuevo archivo denominado **_AlertPartial.cshtml** en el directorio **./Views/Shared** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="77657-135">Add a new file named **_AlertPartial.cshtml** in the **./Views/Shared** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. <span data-ttu-id="77657-136">Abra el **archivo ./Views/Shared/_Layout.cshtml** y reemplace todo su contenido por el código siguiente para actualizar el diseño global de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="77657-136">Open the **./Views/Shared/_Layout.cshtml** file, and replace its entire contents with the following code to update the global layout of the app.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. <span data-ttu-id="77657-137">Abra **./wwwroot/css/site.css** y agregue el siguiente código en la parte inferior del archivo.</span><span class="sxs-lookup"><span data-stu-id="77657-137">Open **./wwwroot/css/site.css** and add the following code at the bottom of the file.</span></span>

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. <span data-ttu-id="77657-138">Abra el **archivo ./Views/Home/index.cshtml** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="77657-138">Open the **./Views/Home/index.cshtml** file and replace its contents with the following.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. <span data-ttu-id="77657-139">Cree un nuevo directorio en el directorio **./wwwroot** denominado **img**.</span><span class="sxs-lookup"><span data-stu-id="77657-139">Create a new directory in the **./wwwroot** directory named **img**.</span></span> <span data-ttu-id="77657-140">Agregue un archivo de imagen que elija con el nombre **no-profile-photo.png** en este directorio.</span><span class="sxs-lookup"><span data-stu-id="77657-140">Add an image file of your choosing named **no-profile-photo.png** in this directory.</span></span> <span data-ttu-id="77657-141">Esta imagen se usará como foto del usuario cuando el usuario no tenga ninguna foto en Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="77657-141">This image will be used as the user's photo when the user has no photo in Microsoft Graph.</span></span>

    > [!TIP]
    > <span data-ttu-id="77657-142">Puede descargar la imagen usada en estas capturas de pantalla desde [GitHub.](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png)</span><span class="sxs-lookup"><span data-stu-id="77657-142">You can download the image used in these screenshots from [GitHub](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png).</span></span>

1. <span data-ttu-id="77657-143">Guarde todos los cambios y reinicie el servidor ( `dotnet run` ).</span><span class="sxs-lookup"><span data-stu-id="77657-143">Save all of your changes and restart the server (`dotnet run`).</span></span> <span data-ttu-id="77657-144">Ahora, la aplicación debería tener un aspecto muy diferente.</span><span class="sxs-lookup"><span data-stu-id="77657-144">Now, the app should look very different.</span></span>

    ![Captura de pantalla de la página principal rediseñada](./images/create-app-01.png)
