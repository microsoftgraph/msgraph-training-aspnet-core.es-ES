---
ms.openlocfilehash: 73ba9c271c9c6675ffbb22ce4f800ffb652c715d
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942137"
---
<!-- markdownlint-disable MD002 MD041 -->

Este tutorial le enseña a crear una aplicación web de ASP.NET Core que use la API de Microsoft Graph para recuperar información de calendario para un usuario.

> [!TIP]
> Si prefiere descargar el tutorial completo, puede descargar o clonar el [repositorio de GitHub.](https://github.com/microsoftgraph/msgraph-training-aspnet-core) Consulta el archivo README en la carpeta **de demostración** para obtener instrucciones sobre cómo configurar la aplicación con un identificador y un secreto de aplicación.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar este tutorial, debes tener [instalado el SDK de .NET Core](https://dotnet.microsoft.com/download) en el equipo de desarrollo. Si no tiene el SDK, visite el vínculo anterior para ver las opciones de descarga.

También debe tener una cuenta personal de Microsoft con un buzón en Outlook.com o una cuenta de Microsoft para el trabajo o la escuela. Si no tienes una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:

- Puede registrarse [para obtener una nueva cuenta personal de Microsoft.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)
- Puede registrarse en el Programa de desarrolladores de [Office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.

> [!NOTE]
> Este tutorial se escribió con .NET Core SDK versión 5.0.102. Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.

## <a name="feedback"></a>Comentarios

Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnet-core).
