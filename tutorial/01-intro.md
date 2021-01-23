---
ms.openlocfilehash: 73ba9c271c9c6675ffbb22ce4f800ffb652c715d
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942137"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="14f80-101">Este tutorial le enseña a crear una aplicación web de ASP.NET Core que use la API de Microsoft Graph para recuperar información de calendario para un usuario.</span><span class="sxs-lookup"><span data-stu-id="14f80-101">This tutorial teaches you how to build an ASP.NET Core web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="14f80-102">Si prefiere descargar el tutorial completo, puede descargar o clonar el [repositorio de GitHub.](https://github.com/microsoftgraph/msgraph-training-aspnet-core)</span><span class="sxs-lookup"><span data-stu-id="14f80-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnet-core).</span></span> <span data-ttu-id="14f80-103">Consulta el archivo README en la carpeta **de demostración** para obtener instrucciones sobre cómo configurar la aplicación con un identificador y un secreto de aplicación.</span><span class="sxs-lookup"><span data-stu-id="14f80-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="14f80-104">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="14f80-104">Prerequisites</span></span>

<span data-ttu-id="14f80-105">Antes de comenzar este tutorial, debes tener [instalado el SDK de .NET Core](https://dotnet.microsoft.com/download) en el equipo de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="14f80-105">Before you start this tutorial, you should have the [.NET Core SDK](https://dotnet.microsoft.com/download) installed on your development machine.</span></span> <span data-ttu-id="14f80-106">Si no tiene el SDK, visite el vínculo anterior para ver las opciones de descarga.</span><span class="sxs-lookup"><span data-stu-id="14f80-106">If you do not have the SDK, visit the previous link for download options.</span></span>

<span data-ttu-id="14f80-107">También debe tener una cuenta personal de Microsoft con un buzón en Outlook.com o una cuenta de Microsoft para el trabajo o la escuela.</span><span class="sxs-lookup"><span data-stu-id="14f80-107">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="14f80-108">Si no tienes una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:</span><span class="sxs-lookup"><span data-stu-id="14f80-108">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="14f80-109">Puede registrarse [para obtener una nueva cuenta personal de Microsoft.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)</span><span class="sxs-lookup"><span data-stu-id="14f80-109">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="14f80-110">Puede registrarse en el Programa de desarrolladores de [Office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.</span><span class="sxs-lookup"><span data-stu-id="14f80-110">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="14f80-111">Este tutorial se escribió con .NET Core SDK versión 5.0.102.</span><span class="sxs-lookup"><span data-stu-id="14f80-111">This tutorial was written with .NET Core SDK version 5.0.102.</span></span> <span data-ttu-id="14f80-112">Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.</span><span class="sxs-lookup"><span data-stu-id="14f80-112">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="14f80-113">Comentarios</span><span class="sxs-lookup"><span data-stu-id="14f80-113">Feedback</span></span>

<span data-ttu-id="14f80-114">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnet-core).</span><span class="sxs-lookup"><span data-stu-id="14f80-114">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnet-core).</span></span>
