---
title: "Cómo depurar una aplicación web de Node.js en el Servicio de aplicaciones de Azure"
description: "Aprenda a depurar una aplicación web de Node.js en el Servicio de aplicaciones de Azure."
tags: azure-portal
services: app-service\web
documentationcenter: nodejs
author: rmcmurray
manager: erikre
editor: 
ms.assetid: a48f906c-1a3e-43bc-ae84-7d2dde175b15
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 04/25/2017
ms.author: robmcm
translationtype: Human Translation
ms.sourcegitcommit: a087df444c5c88ee1dbcf8eb18abf883549a9024
ms.openlocfilehash: 29cb2faa2e2d6cb9f45242794d88d3c8f881539d
ms.lasthandoff: 03/15/2017


---
# <a name="how-to-debug-a-nodejs-web-app-in-azure-app-service"></a>Cómo depurar una aplicación web de Node.js en el Servicio de aplicaciones de Azure
Azure proporciona diagnósticos integrados para ayudar a depurar aplicaciones Node.js hospedadas en aplicaciones web del [Servicio de aplicaciones de Azure](http://go.microsoft.com/fwlink/?LinkId=529714) . En este artículo aprenderá a habilitar el registro de stdout y stderr, mostrar información de error en el explorador y a descargar y ver los archivos de registro.

El diagnóstico de las aplicaciones Node.js hospedadas en Azure lo proporciona [IISNode]. Si bien este artículo analiza la configuración más común para recopilar información de diagnóstico, no proporciona una referencia completa para trabajar con IISNode. Para obtener más información sobre el trabajo con IISNode, consulte el archivo [léame de IISNode] en GitHub.

<a id="enablelogging"></a>

## <a name="enable-logging"></a>Habilitación del registro
De manera predeterminada, una aplicación web del Servicio de aplicaciones solo captura información de diagnóstico sobre las implementaciones, como cuando se realiza la implementación de un sitio web con Git. Esta información es útil si está teniendo problemas durante la implementación, como un error al instalar un módulo al que se hace referencia en **package.json**o si va a usar un script de implementación personalizado.

Para habilitar el registro de las secuencias stdout y stderr, debe crear un archivo **IISNode.yml** en la raíz de su aplicación Node.js y agregar lo siguiente:

    loggingEnabled: true

Esto permite el registro de stderr y stdout desde su aplicación Node.js.

El archivo **IISNode.yml** puede usarse también para controlar si los errores descriptivos o del desarrollador se devuelven al explorador cuando se produce un error. Para habilitar los errores del desarrollador, agregue la siguiente línea al archivo **IISNode.yml** :

    devErrorsEnabled: true

Después de que se habilita esta opción, IISNode devolverá los últimos 64 K de información que se enviaron a stderr en vez de un error descriptivo como "se produjo un error de servidor interno".

> [!NOTE]
> Si bien devErrorsEnabled es útil para diagnosticar los problemas que se producen durante el desarrollo, su habilitación en un entorno de producción puede tener como resultado que se envíen errores de desarrollo a los usuarios finales.
> 
> 

Si el archivo **IISNode.yml** no existía ya en su aplicación, debe reiniciar la aplicación web después de publicar la aplicación actualizada. Si solamente va a cambiar la configuración de un archivo **IISNode.yml** existente que ya se publicó anteriormente, no es necesario reiniciar.

> [!NOTE]
> Si aplicación web se creó usando las herramientas de línea de comandos de Azure o los cmdlets de Azure PowerShell, automáticamente se crea un archivo predeterminado **IISNode.yml** .
> 
> 

Para reiniciar la aplicación web, selecciónela en el [Portal de Azure](https://portal.azure.com)y después haga clic en el botón **REINICIAR** :

![Botón de reinicio][restart-button]

Si las herramientas de línea de comandos de Azure están instaladas en su entorno de desarrollo, puede usar el siguiente comando para reiniciar la aplicación web:

    azure site restart [sitename]

> [!NOTE]
> Si bien loggingEnabled y devErrorsEnabled son las opciones de configuración de IISNode.yml de uso más común para capturar información de diagnóstico, se puede usar IISNode.yml para configurar diversas opciones para su entorno de hospedaje. Para una lista completa de las opciones de configuración, consulte el archivo [iisnode_schema.xml](https://github.com/tjanczuk/iisnode/blob/master/src/config/iisnode_schema.xml).
> 
> 

<a id="viewlogs"></a>

## <a name="accessing-logs"></a>Acceso a los registros
Se puede tener acceso a los registros de diagnósticos de tres formas; mediante el protocolo de transferencia de archivos (FTP), la descarga de un archivo Zip o como una secuencia en directo actualizada del registro (lo que se conoce también como cola). La descarga del archivo Zip de los archivos de registro o la visualización de la secuencia en directo requieren las herramientas de línea de comandos de Azure. Estas se pueden instalar usando el siguiente comando:

    npm install azure-cli -g

Una vez instaladas, se puede tener acceso a ellas mediante el comando 'azure'. Primero se deben configurar las herramientas de línea de comandos para usar su suscripción de Azure. Para obtener información sobre cómo llevar a cabo esta tarea, consulte la sección **Descarga e importación de la configuración de publicación** del artículo [Uso de las herramientas de línea de comandos de Azure](../xplat-cli-connect.md) .

### <a name="ftp"></a>FTP
Para tener acceso a la información de diagnóstico a través de FTP, visite el [Portal de Azure](https://portal.azure.com), seleccione su aplicación web y luego seleccione el **PANEL**. En la sección de **vínculos rápidos**, los vínculos **FTP DIAGNOSTIC LOGS** y **FTPS DIAGNOSTIC LOGS** proporcionan acceso a los registros usando el protocolo FTP.

> [!NOTE]
> Si no ha configurado antes el nombre de usuario y la contraseña de FTP o la implementación, puede hacerlo desde la página de administración **Inicio rápido** al seleccionar **Configurar las credenciales de implementación**.
> 
> 

La URL de FTP devuelta en el panel es para el directorio **LogFiles** , que contendrá los siguientes subdirectorios:

* [Deployment Method](web-sites-deploy.md) : si usa un método de implementación como Git, se creará un directorio del mismo nombre que contendrá información relacionada con las implementaciones.
* nodejs: La información de Stdout y stderr capturada desde todas las instancias de su aplicación (cuando loggingEnabled es verdadero).

### <a name="zip-archive"></a>Archivo Zip
Para descargar un archivo Zip de los registros de diagnóstico, use el siguiente comando de las herramientas de línea de comandos de Azure:

    azure site log download [sitename]

De esta manera se descargará un archivo **diagnostics.zip** en el directorio actual. Este archivo contiene la siguiente estructura de directorios:

* deployments: Un registro de información sobre las implementaciones de su aplicación
* LogFiles
  
  * [Deployment Method](web-sites-deploy.md) : si usa un método de implementación como Git, se creará un directorio del mismo nombre que contendrá información relacionada con las implementaciones.
  * nodejs: La información de Stdout y stderr capturada desde todas las instancias de su aplicación (cuando loggingEnabled es verdadero).

### <a name="live-stream-tail"></a>Secuencia en directo (cola)
Para ver una secuencia en directo de la información de registro de diagnóstico, use el siguiente comando en las herramientas de línea de comandos de Azure:

    azure site log tail [sitename]

De esta manera se devolverá una secuencia de eventos de registro que se actualizan a medida que se producen en el servidor. Esta secuencia devolverá información de implementación además de información de stdout y stderr (cuando loggingEnabled es verdadero).

<a id="nextsteps"></a>

## <a name="next-steps"></a>Pasos siguientes
En este artículo aprendió a habilitar y a tener acceso a información de diagnóstico en Azure. Si bien esta información es útil para comprender los problemas que se producen con su aplicación, puede indicar un problema con un módulo que está usando o que la versión de Node.js que usan las aplicaciones web del Servicio de aplicaciones es diferente a la usada en su entorno de implementación.

Para obtener información sobre el trabajo con módulos en Azure, consulte [Uso de módulos Node.js con aplicaciones de Azure](../nodejs-use-node-modules-azure-apps.md).

Para obtener información sobre la especificación de una versión de Node.js para su aplicación, consulte [Especificación de una versión de Node.js en una aplicación de Azure].

Para obtener más información, consulte también el [Centro para desarrolladores de Node.js](/develop/nodejs/).

## <a name="whats-changed"></a>Lo que ha cambiado
* Para obtener una guía del cambio de Sitios web a Servicio de aplicaciones, consulte: [Servicio de aplicaciones de Azure y su impacto en los servicios de Azure existentes](http://go.microsoft.com/fwlink/?LinkId=529714)

> [!NOTE]
> Si desea empezar a trabajar con el Servicio de aplicaciones de Azure antes de inscribirse para abrir una cuenta de Azure, vaya a [Prueba del Servicio de aplicaciones](https://azure.microsoft.com/try/app-service/), donde podrá crear inmediatamente una aplicación web de inicio de corta duración en el Servicio de aplicaciones. No es necesario proporcionar ninguna tarjeta de crédito ni asumir ningún compromiso.
> 
> 

[IISNode]: https://github.com/tjanczuk/iisnode
[léame de IISNode]: https://github.com/tjanczuk/iisnode#readme
[How to Use The Azure Command-Line Interface]:../cli-install-nodejs.md
[Using Node.js Modules with Azure Applications]: ../nodejs-use-node-modules-azure-apps.md
[Especificación de una versión de Node.js en una aplicación de Azure]: ../nodejs-specify-node-version-azure-apps.md

[restart-button]: ./media/web-sites-nodejs-debug/restartbutton.png


