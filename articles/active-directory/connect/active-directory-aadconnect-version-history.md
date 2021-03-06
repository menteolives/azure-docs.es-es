---
title: 'Azure AD Connect: historial de versiones | Microsoft Docs'
description: "En este artículo se muestran todas las versiones de Azure AD Connect y Sincronización de Azure AD"
services: active-directory
documentationcenter: 
author: billmath
manager: femila
editor: 
ms.assetid: ef2797d7-d440-4a9a-a648-db32ad137494
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/08/2017
ms.author: billmath
ms.translationtype: Human Translation
ms.sourcegitcommit: 17c4dc6a72328b613f31407aff8b6c9eacd70d9a
ms.openlocfilehash: 4703cd03db398d8dc59fb5f5c0cf71214c606bc8
ms.contentlocale: es-es
ms.lasthandoff: 05/16/2017


---
# <a name="azure-ad-connect-version-release-history"></a>Azure AD Connect: historial de versiones
El equipo de Azure Active Directory (Azure AD) actualiza periódicamente Azure AD Connect con nuevas características y funcionalidades. No todas las adiciones son aplicables a todas las audiencias.

Este artículo está diseñado para ayudarle a realizar un seguimiento de las versiones que se han publicado y comprender si necesita actualizar a la versión más reciente o no.

Lista de temas relacionados:


Tema. |  Detalles
--------- | --------- |
Pasos para actualizar desde Azure AD Connect | Diferentes métodos para [actualizar desde una versión anterior a la última](active-directory-aadconnect-upgrade-previous-version.md) versión de Azure AD Connect.
Permisos necesarios | Para más información sobre los permisos necesarios para aplicar una actualización, consulte [cuentas y permisos](./active-directory-aadconnect-accounts-permissions.md#upgrade).
Descargar| [Descarga de Azure AD Connect](http://go.microsoft.com/fwlink/?LinkId=615771).

## <a name="115240"></a>1.1.524.0
Fecha de publicación: mayo de 2017

> [!IMPORTANT]
> Se produjeron cambios en las reglas de sincronización y el esquema en esta compilación. El servicio de sincronización de Azure AD Connect activará pasos de importación completa y sincronización completa después de la actualización. A continuación, se describen los detalles de los cambios.
>
>

**Problemas corregidos:**

Sincronización de Azure AD Connect

* Se ha corregido un problema que hacía que se produjera la actualización automática en el servidor de Azure AD Connect incluso si el cliente había deshabilitado la característica mediante el cmdlet Set-ADSyncAutoUpgrade. Con esta corrección, el proceso de actualización automática en el servidor sigue comprobando periódicamente si hay que actualizar, pero el instalador descargado respeta la configuración de actualización automática.
* Durante la actualización local de DirSync, Azure AD Connect crea una cuenta de servicio de Azure AD que el conector de Azure AD va a usar para sincronizar con Azure AD. Una vez creada la cuenta, Azure AD Connect la usa para autenticarse con Azure AD. En ocasiones, la autenticación genera un error por problemas transitorios, lo que a su vez hace que la actualización local de DirSync produzca un *error al ejecutar la tarea de configuración de la sincronización de AAD AADSTS50034; en este error se indica que, para iniciar sesión en la aplicación, se debe agregar la cuenta al directorio xxx.onmicrosoft.com*. Para mejorar la resistencia de la actualización de DirSync, Azure AD Connect ahora reintenta el paso de autenticación.
* Había un problema con la compilación 443 que hacía que la actualización local de DirSync se realizara correctamente, pero no se creaban perfiles de ejecución necesarios para la sincronización de directorios. Se incluye lógica de recuperación en esta compilación de Azure AD Connect. Cuando el cliente actualiza a esta compilación, Azure AD Connect detecta que faltan perfiles de ejecución y los crea.
* Se corrigió un problema que hacía que el proceso de sincronización de contraseña no se iniciara con el id. de evento 6900 y el error *"Ya se agregó un elemento con la misma clave"*. Este problema se producía si actualizaba la configuración del filtrado por unidad organizativa para incluir la partición de configuración de AD. Para corregir este problema, ahora el proceso de sincronización de contraseña sincroniza los cambios de contraseña solo desde particiones de dominio de AD. Se omiten las particiones que no son de dominio, como la partición de configuración.
* Durante la instalación rápida, Azure AD Connect crea una cuenta de AD DS local que el conector de AD usará para comunicarse con la instancia local de AD. Antes, la cuenta se creaba con la marca PASSWD_NOTREQD establecida en el atributo user-Account-Control y se establecía una contraseña aleatoria en la cuenta. Ahora, Azure AD Connect quita explícitamente la marca PASSWD_NOTREQD una vez que se establece la contraseña en la cuenta.
* Se corrigió un problema que causaba un error de actualización de DirSync que indicaba que *se había producido un interbloqueo en SQL Server al intentar adquirir un bloqueo de aplicación* cuando se encontraba el atributo mailNickname en el esquema de AD local, pero no estaba enlazado a la clase de objeto de usuario de AD.
* Se ha corregido un problema que hacía que la característica Reescritura de dispositivos se deshabilitara automáticamente cuando un administrador estaba actualizando la configuración de Azure AD Connect Sync mediante el asistente de Azure AD Connect. Esto se debía a que el asistente realizaba una comprobación de requisitos previos para la configuración de Reescritura de dispositivos existente en la instancia local de AD y se producía un error en la comprobación. La solución consiste en omitir la comprobación si Reescritura de dispositivos ya se había habilitado antes.
* Para configurar el filtrado por unidad organizativa, puede usar el asistente de Azure AD Connect o Synchronization Service Manager. Anteriormente, si usaba al asistente de Azure AD Connect para configurar el filtrado por unidad organizativa, las unidades organizativas nuevas creadas después se incluían en la sincronización de directorios. Si no desea que se incluyan las unidades organizativas nuevas, debe configurar el filtrado por unidad organizativa mediante Synchronization Service Manager. Ahora, puede lograr el mismo comportamiento con el asistente de Azure AD Connect.
* Se ha corregido un problema que hacía que los procedimientos almacenados que Azure AD Connect necesitaba se crearan bajo el esquema del administrador que llevaba a cabo la instalación, en lugar de bajo el esquema dbo.
* Se ha corregido un problema que hacía que el atributo TrackingId devuelto por Azure AD se omitiera en los registros de eventos del servidor de AAD Connect. El problema se producía si Azure AD Connect recibía un mensaje de redirección de Azure AD y Azure AD Connect no se podía conectar al punto de conexión proporcionado. Los ingenieros de soporte técnico usan TrackingId para correlacionarlo con los registros en el servicio durante la solución de problemas.
* Cuando Azure AD Connect recibe un error LargeObject de Azure AD, Azure AD Connect genera un evento con el id. 6941 y el mensaje *"El objeto aprovisionado es demasiado grande. Recorte el número de valores de atributo de este objeto"*. Al mismo tiempo, Azure AD Connect también genera un evento que puede inducir a error, con el id. 6900 y el mensaje *"Microsoft.Online.Coexistence.ProvisionRetryException: No se puede comunicar con el servicio de Windows Azure Active Directory"*. Para minimizar las confusiones, Azure AD Connect ya no genera este último evento cuando se recibe el error LargeObject.
* Se ha corregido un problema que hacía que Synchronization Service Manager dejara de responder al intentar actualizar la configuración para un conector LDAP genérico.

**Nuevas características y mejoras:**

Sincronización de Azure AD Connect
* Cambios en las reglas de sincronización: se han implementado los siguientes cambios en reglas de sincronización:
  * Se ha actualizado el conjunto de reglas de sincronización predeterminado para que no se exporten los atributos **userCertificate** ni **userSMIMECertificate** si tienen más de 15 valores.
  * Ahora los atributos **employeeID** y **msExchBypassModerationLink** de AD están incluidos en el conjunto de reglas de sincronización predeterminado.
  * Se ha quitado el atributo **photo** de AD del conjunto de reglas de sincronización predeterminado.
  * Se ha agregado **preferredDataLocation** al esquema de metaverso y al del conector de AAD. Los clientes que deseen actualizar cualquiera de los atributos en Azure AD pueden implementar reglas de sincronización personalizadas para hacerlo. Para más información sobre el atributo, consulte la sección [sobre la habilitación de la sincronización de PreferredDataLocation en el artículo Azure AD Connect Sync: cómo realizar un cambio en la configuración predeterminada](active-directory-aadconnectsync-change-the-configuration.md#enable-synchronization-of-preferreddatalocation).
  * Agregue **userType** al esquema de metaverso y al del conector de AAD. Los clientes que deseen actualizar cualquiera de los atributos en Azure AD pueden implementar reglas de sincronización personalizadas para hacerlo.

* Ahora Azure AD Connect habilita automáticamente el uso del atributo ConsistencyGuid como atributo de delimitador de origen para objetos de la instancia local de AD. Además, Azure AD Connect rellena el atributo ConsistencyGuid con el valor del atributo objectGuid si está vacío. Esta característica solo es aplicable a nueva implementaciones. Para más información sobre esta característica, consulte la sección [Uso de msDS-ConsistencyGuid como sourceAnchor en el artículo Azure AD Connect: conceptos de diseño](active-directory-aadconnect-design-concepts.md#using-msds-consistencyguid-as-sourceanchor).
* Se ha agregado un nuevo cmdlet de solución de problemas, Invoke-ADSyncDiagnostics, para ayudar a diagnosticar problemas relacionados con la sincronización de hash de contraseña.
* Azure AD Connect es compatible ahora con la sincronización de objetos de carpeta pública habilitada para correo de una instancia local de AD a Azure AD. Puede habilitar la característica con el asistente de Azure AD Connect en Características opcionales.
* Azure AD Connect requiere cuentas de AD DS para sincronizar desde la instancia local de AD. Antes, si instalaba Azure AD Connect con el modo rápido, podía proporcionar las credenciales de una cuenta de administrador de organización y dejar que Azure AD Connect creara la cuenta de AD DS necesaria. Sin embargo, para la instalación personalizada y para agregar bosques a una implementación existente, tenía que proporcionar la cuenta de AD DS en su lugar. Ahora, también tiene la opción de proporcionar las credenciales de una cuenta de administrador de organización durante la instalación personalizada y dejar que Azure AD Connect cree la cuenta de AD DS necesaria.
* Azure AD Connect ahora es compatible con AOA de SQL. Debe habilitar SQL antes de instalar Azure AD Connect. Durante la instalación, Azure AD Connect detecta si la instancia de SQL proporcionada está habilitada para AOA de SQL o no. Si AOA de SQL está habilitada, Azure AD Connect averigua si AOA de SQL está configurada para usar la replicación sincrónica o asincrónica. Cuando configure la escucha de grupo de disponibilidad, se recomienda que establezca la propiedad RegisterAllProvidersIP en 0. Esto se debe a que Azure AD Connect actualmente usa SQL Native Client para conectarse a SQL y SQL Native Client no admite el uso de la propiedad MultiSubNetFailover.
* Si usa LocalDB como base de datos para el servidor de Azure AD Connect y ha alcanzado su límite de 10 GB de tamaño, el servicio de sincronización deja de iniciarse. Antes, tenía que realizar la operación ShrinkDatabase en LocalDB para recuperar el suficiente espacio de base de datos para iniciar el servicio de sincronización. Después, podía usar Synchronization Service Manager para eliminar el historial de ejecución para reclamar más espacio de base de datos. Ahora, puede usar el cmdlet Start-ADSyncPurgeRunHistory para purgar los datos del historial de LocalDB y recuperar espacio de base de datos. Además, este cmdlet admite un modo sin conexión (especificando el parámetro -offline) que se puede usar cuando no se está ejecutando el servicio de sincronización. Nota: El modo sin conexión solo se puede usar si el servicio de sincronización no se está ejecutando y la base de datos usada es LocalDB.
* Para reducir la cantidad de espacio de almacenamiento necesario, ahora Azure AD Connect comprime los detalles de errores de sincronización antes de almacenarlos en las bases de datos LocalDB y SQL. Al actualizar desde una versión anterior de Azure AD Connect a esta versión, Azure AD Connect realiza una compresión única de los detalles de errores de sincronización existentes.
* Antes, después de actualizar la configuración del filtrado por unidad organizativa, había que ejecutar manualmente la importación completa para asegurarse de que los objetos existentes quedaran correctamente incluidos o excluidos de la sincronización de directorios. Ahora, Azure AD Connect desencadena automáticamente la importación completa durante el próximo ciclo de sincronización. Además, la importación completa solo se aplica a los conectores de AD afectados por la actualización. Nota: Esta mejora es aplicable a las actualizaciones del filtrado por unidad organizativa realizadas con el asistente de Azure AD Connect únicamente. No es aplicable al filtrado por unidad organizativa realizado mediante Synchronization Service Manager.
* Antes, el filtrado basado en grupo solo admitía usuarios, grupos y objetos de contacto. Ahora, también admite objetos de equipo.
* Antes, podía eliminar datos de espacio conector sin deshabilitar el programador de Azure AD Connect Sync. Ahora, Synchronization Service Manager bloquea la eliminación de datos del espacio conector si detecta que el programador está habilitado. Además, se devuelve una advertencia para informar a los clientes acerca de la posible pérdida de datos si se eliminan los datos del espacio conector.
* Anteriormente, tenía que deshabilitar la transcripción de PowerShell para que el asistente de Azure AD Connect se ejecutara correctamente. Este problema se ha resuelto en parte. Puede habilitar la transcripción de PowerShell si usa el asistente de Azure AD Connect para administrar la configuración de sincronización. Debe deshabilitarla si usa el asistente de Azure AD Connect para administrar la configuración de AD FS.



## <a name="114860"></a>1.1.486.0
Fecha de publicación: abril de 2017

**Problemas corregidos:**
* Se ha corregido el problema según el cual Azure AD Connect no se instala correctamente en una versión localizada de Windows Server.

## <a name="114840"></a>1.1.484.0
Fecha de publicación: abril de 2017

**Problemas conocidos**:

* Esta versión de Azure AD Connect no se instalará correctamente si se cumplen las condiciones siguientes:
   1. Realiza una actualización de DirSync o una instalación nueva de Azure AD Connect.
   2. Utiliza una versión localizada de Windows Server, donde el nombre del grupo de administradores integrado en el servidor no es "Administradores".
   3. Está usando la instancia predeterminada de Local DB incluida en SQL Server 2012 Express que se instala con Azure AD Connect en lugar de proporcionar su propia versión completa de SQL Server.

**Problemas corregidos:**

Sincronización de Azure AD Connect
* Se ha corregido un problema según el cual el programador de sincronización omite todo el paso de sincronización si uno o varios conectores no tienen un perfil de ejecución para ese paso de sincronización. Por ejemplo, puede agregar manualmente un conector con Synchronization Service Manager sin crear un perfil de ejecución de importación diferencial para él. Esta revisión garantiza que el programador de sincronización sigue ejecutando la importación diferencial para los demás conectores.
* Se corrigió un problema según el cual el servicio de sincronización detiene inmediatamente el procesamiento de un perfil de ejecución cuando es encuentra un problema con uno de los pasos de ejecución. Esta revisión garantiza que el servicio de sincronización omite ese paso de ejecución y continúa procesando el resto. Por ejemplo, tiene un perfil de ejecución de importación diferencial para el conector AD con varios pasos de ejecución (uno para cada dominio de Active Directory local). El servicio de sincronización ejecutará la importación diferencial con los demás dominios de AD, incluso si uno de ellos tiene problemas de conectividad de red.
* Se ha corregido un problema que hace que la actualización del conector Azure AD se omita durante la actualización automática.
* Se corrigió un problema que hace que Azure AD Connect determine incorrectamente si el servidor es un controlador de dominio durante la instalación, que a su vez provoca un error de actualización en DirSync.
* Se ha corregido un problema que hace que la actualización en contexto de DirSync no cree ningún perfil de ejecución para el conector Azure AD.
* Se ha corregido un problema según el cual la interfaz de usuario de Synchronization Service Manager deja de responder al intentar configurar el conector LDAP genérico.

Administración de AD FS
* Se ha corregido un problema en el que se produce un error en el Asistente para Azure AD Connect si el nodo principal de AD FS se ha movido a otro servidor.

SSO de escritorio
* Se ha corregido un problema en el Asistente para Azure AD Connect en el que la pantalla de inicio de sesión no le permite habilitar la característica de SSO de escritorio si ha elegido la sincronización de contraseña como opción de inicio de sesión durante la instalación nueva.

**Nuevas características y mejoras:**

Sincronización de Azure AD Connect
* Sincronización de Azure AD Connect ahora admite el uso de la cuenta de servicio virtual, la cuenta de servicio administrado y la cuenta de servicio administrado de grupo como su cuenta de servicio. Esto se aplica únicamente a la nueva instalación de Azure AD Connect. Al instalar Azure AD Connect:
    * De forma predeterminada, el Asistente para Azure AD Connect creará una cuenta de servicio virtual y la utilizará como su cuenta de servicio.
    * Si realiza la instalación en un controlador de dominio, Azure AD Connect vuelve al comportamiento anterior donde se creará una cuenta de usuario de dominio y se utilizará como cuenta de servicio.
    * Puede invalidar el comportamiento predeterminado al proporcionar una de las opciones siguientes:
      * Una cuenta de servicio administrada de grupo
      * Una cuenta de servicio administrada
      * Una cuenta de usuario de dominio
      * Una cuenta de usuario local
* Anteriormente, si actualizaba a una nueva versión de Azure AD Connect que contenía la actualización de los conectores o cambios de las reglas de sincronización, Azure AD Connect desencadenaba un ciclo completo de sincronización. Ahora, Azure AD Connect desencadena selectivamente el paso de importación completa solo para los conectores con actualización y el paso de sincronización completa solo para los conectores con cambios de la regla de sincronización.
* Anteriormente, el umbral de eliminación de exportación solo se aplicaba a las exportaciones desencadenadas mediante el programador de sincronización. Ahora, la característica se extiende para incluir las exportaciones desencadenadas manualmente por el cliente con Synchronization Service Manager.
* En su inquilino de Azure AD, hay una configuración del servicio que indica si la característica de sincronización de contraseña está habilitada para el inquilino o no. Anteriormente, era fácil que la configuración de servicio no se configurara correctamente por Azure AD Connect cuando tenía un servidor provisional y activo. Ahora, Azure AD Connect intenta mantener la coherencia de la configuración del servicio solo con su servidor de Azure AD Connect activo.
* El Asistente para Azure AD Connect ahora detecta y devuelve una advertencia si la instancia de AD local no tiene la papelera de reciclaje de AD habilitada.
* Anteriormente, se agotaba el tiempo de espera de la exportación a Azure AD y se producía un error si el tamaño combinado de los objetos en el lote superaba cierto umbral. Ahora, el servicio de sincronización intentará volver a enviar los objetos en lotes más pequeños e independientes si se encuentra el problema.
* La aplicación de administración de claves del servicio de sincronización se quitó del menú Inicio de Windows. La administración de la clave de cifrado continuará admitiéndose a través de la interfaz de línea de comandos mediante miiskmu.exe. Para más información sobre cómo administrar la clave de cifrado, consulte el artículo [Abandonar la clave de cifrado de sincronización de Azure AD Connect](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-change-serviceacct-pass#abandoning-the-azure-ad-connect-sync-encryption-key).
* Anteriormente, si cambiaba la contraseña de la cuenta del servicio de sincronización de Azure AD Connect, el servicio de sincronización no se iniciaba correctamente hasta que hubiera abandonado la clave de cifrado y reinicializado la contraseña de la cuenta del servicio de sincronización de Azure AD Connect. Ahora, esto ya no es necesario.

SSO de escritorio

* El Asistente para Azure AD Connect ya no requiere que el puerto 9090 esté abierto en la red al configurar la autenticación de paso a través y SSO de escritorio. Solo se requiere el puerto 443. 

## <a name="114430"></a>1.1.443.0
Fecha de publicación: marzo de 2017

**Problemas corregidos:**

Sincronización de Azure AD Connect
* Se ha corregido un problema que hace el Asistente para Azure AD Connect deje de funcionar si el nombre para mostrar del conector de Azure AD no contiene el dominio inicial onmicrosoft.com que se asignan al inquilino de Azure AD.
* Se ha corregido un problema que hace que el Asistente para Azure AD Connect genere un error al realizar la conexión a la base de datos SQL cuando la contraseña de la cuenta de servicio de sincronización tenga caracteres especiales, como un apóstrofo, una coma y un espacio.
* Se corrigió un problema que hace que el error que indica que dimage tiene un delimitador que es diferente de la imagen se produzca en un servidor de Azure AD Connect en modo de almacenamiento provisional, una vez que ha excluido temporalmente un objeto AD de sincronización local de la sincronización y, luego, lo incluye de nuevo para realizar la sincronización.
* Se corrigió un problema que hace que el error que indica que el objeto que ha localizado DN es un fantasma se produzca en un servidor de Azure AD Connect en modo de almacenamiento provisional, una vez que ha excluido temporalmente un objeto AD de sincronización local de la sincronización y, luego, lo incluye de nuevo para realizar la sincronización.

Administración de AD FS
* Se ha corregido un problema donde el Asistente para Azure AD Connect no actualizar la configuración de AD FS y establece las notificaciones adecuadas en el usuario de confianza después de configurar el identificador de inicio de sesión alternativo.
* Se ha corregido un problema donde el Asistente para Azure AD Connect no puede administrar correctamente los servidores de AD FS cuyas cuentas de servicio se configuran mediante el formato userPrincipalName en lugar de sAMAccountName.

Autenticación de paso a través
* Se ha corregido un problema que hace que el Asistente para Azure AD Connect deje de funcionar si se selecciona la autenticación de paso a través, pero se produce un error de registro de su conector.
* Se ha corregido un problema que hace que el Asistente para Azure AD Connect pase por alto las comprobaciones de validación en el método de inicio de sesión seleccionado cuando se habilita la característica de SSO de escritorio.

Restablecimiento de contraseña
* Se ha corregido un problema que puede hacer que el servidor de Azure AAD Connect no intente volver a conectarse si un firewall o un proxy termina la conexión.

**Nuevas características y mejoras:**

Sincronización de Azure AD Connect
* El cmdlet Get-ADSyncScheduler ahora devuelve una nueva propiedad booleana denominada "SyncCycleInProgress". Si el valor devuelto es True, significa que hay un ciclo de sincronización programada en curso.
* La carpeta de destino para almacenar la instalación de Azure AD Connect y los registros de instalación se han movido desde %localappdata%\AADConnect a %programdata%\AADConnect para mejorar la accesibilidad a los archivos de registro.

Administración de AD FS
* Se agregó compatibilidad para actualizar el certificado SSL de granja de servidores de AD FS.
* Se agregó compatibilidad para administrar AD FS 2016.
* Ahora puede especificar gMSA existente (cuenta de servicio administrada de grupo) durante la instalación de AD FS.
* Ahora puede configurar SHA-256 como el algoritmo de hash de firma para usuarios de confianza de Azure AD.

Restablecimiento de contraseña
* Se han introducido mejoras para permitir que el producto funcione en entornos con reglas de firewall más estrictas.
* Se ha mejorado la confiabilidad de la conexión a Azure Service Bus.

## <a name="113800"></a>1.1.380.0
Fecha de publicación: diciembre de 2016

**Problema corregido:**

* Se ha corregido el problema por el cual falta la regla de notificación de issuerid para Active Directory Federation Services (AD FS) en esta compilación.

>[!NOTE]
>Esta compilación no está disponible para los clientes a través de la característica de actualización automática de Azure AD Connect.

## <a name="113710"></a>1.1.371.0
Fecha de publicación: diciembre de 2016

**Problema conocido:**

* Falta la regla de notificación de issuerid para AD FS en esta compilación. La regla de notificación de issuerid es necesaria si se está realizando una federación de varios dominios con Azure Active Directory (Azure AD). Si utiliza Azure AD Connect para administrar la implementación de AD FS local, la actualización a esta compilación quitará la regla de notificación de issuerid existente de la configuración de AD FS. Puede solucionar el problema agregando la regla de notificación de issuerid después de la instalación o actualización. Para más información sobre cómo agregar la regla de notificación de issuerid, consulte el artículo [Compatibilidad con varios dominios para la federación con Azure AD](active-directory-aadconnect-multiple-domains.md).

**Problema corregido:**

* Si el puerto 9090 no está abierto para las conexiones salientes, la instalación o actualización de Azure AD Connect producirá un error.

>[!NOTE]
>Esta compilación no está disponible para los clientes a través de la característica de actualización automática de Azure AD Connect.

## <a name="113700"></a>1.1.370.0
Fecha de publicación: diciembre de 2016

**Problemas conocidos**:

* Falta la regla de notificación de issuerid para AD FS en esta compilación. La regla de notificación de issuerid es necesaria si se está realizando una federación de varios dominios con Azure AD. Si utiliza Azure AD Connect para administrar la implementación de AD FS local, la actualización a esta compilación quitará la regla de notificación de issuerid existente de la configuración de AD FS. Puede solucionar el problema agregando la regla de notificación de issuerid después de la instalación o actualización. Para más información sobre cómo agregar la regla de notificación de issuerid, consulte el artículo [Compatibilidad con varios dominios para la federación con Azure AD](active-directory-aadconnect-multiple-domains.md).
* El puerto 9090 debe estar abierto para las conexiones salientes a fin de completar la instalación.

**Nuevas características:**

* Autenticación de paso a través (versión preliminar)

>[!NOTE]
>Esta compilación no está disponible para los clientes a través de la característica de actualización automática de Azure AD Connect.

## <a name="113430"></a>1.1.343.0
Fecha de publicación: noviembre de 2016

**Problema conocido:**

* Falta la regla de notificación de issuerid para AD FS en esta compilación. La regla de notificación de issuerid es necesaria si se está realizando una federación de varios dominios con Azure AD. Si utiliza Azure AD Connect para administrar la implementación de AD FS local, la actualización a esta compilación quitará la regla de notificación de issuerid existente de la configuración de AD FS. Puede solucionar el problema agregando la regla de notificación de issuerid después de la instalación o actualización. Para más información sobre cómo agregar la regla de notificación de issuerid, consulte el artículo [Compatibilidad con varios dominios para la federación con Azure AD](active-directory-aadconnect-multiple-domains.md).

**Problemas corregidos:**

* A veces, Azure AD Connect no se puede instalar porque no puede crear una cuenta de servicio local cuya contraseña cumpla el nivel de complejidad especificado por la directiva de contraseñas de la organización.
* Se corrigió un problema por el que las reglas de unión no se vuelven a evaluar si un objeto en el espacio del conector se convierte simultáneamente en un objeto fuera de ámbito para una regla de unión y dentro de ámbito para otra regla. Esto puede ocurrir si tiene dos o más reglas de unión cuyas condiciones de unión son mutuamente excluyentes.
* Se ha corregido un problema por el que no se procesan las reglas de sincronización entrantes (desde Azure AD) que no contienen reglas de unión si tienen valores de prioridad inferiores que aquellas que sí que las contienen.

**Mejoras:**

* Se agregó compatibilidad para la instalación de Azure AD Connect en Windows Server 2016 estándar o superior.
* Se agregó compatibilidad para utilizar SQL Server 2016 como la base de datos remota para Azure AD Connect.

## <a name="112810"></a>1.1.281.0
Fecha de publicación: agosto de 2016

**Problemas corregidos:**

* Los cambios en el intervalo de sincronización no se realizarán hasta que finalice el siguiente ciclo de sincronización.
* El Asistente de Azure AD Connect no acepta cuentas de Azure AD cuyo nombre de usuario comience con un guion bajo (\_).
* Si la contraseña contiene demasiados caracteres especiales, se producirá un error en el Asistente de Azure AD Connect al autenticar la cuenta de Azure AD. Se mostrará el mensaje de error: No se pueden validar las credenciales. Se produjo un error inesperado" .
* Al desinstalar el servidor de ensayo, se deshabilita la sincronización de contraseñas del inquilino de Azure AD y se produce un error de sincronización de contraseñas en el servidor activo.
* Se produce un error de sincronización de contraseñas en casos raros cuando no hay ningún valor de hash de contraseña almacenado en el usuario.
* Cuando se habilita el servidor de Azure AD Connect para el modo provisional, la escritura diferida de contraseñas se deshabilita temporalmente.
* El Asistente de Azure AD Connect no muestra la sincronización de contraseñas y la configuración de escritura diferida de contraseñas reales cuando el servidor está en modo provisional. Siempre se muestran como deshabilitadas.
* Los cambios de configuración en la sincronización de contraseñas y la escritura diferida de contraseñas no se almacenan en el Asistente de Azure AD Connect cuando el servidor está en modo provisional.

**Mejoras:**

* Se ha actualizado el cmdlet Start-ADSyncSyncCycle para indicar si se puede iniciar correctamente un nuevo ciclo de sincronización.
* Se ha agregado el cmdlet Stop-ADSyncSyncCycle para finalizar el ciclo de sincronización y la operación actualmente en curso.
* Se ha actualizado el cmdlet Stop-ADSyncScheduler para finalizar el ciclo de sincronización y la operación actualmente en curso.
* Al configurar [extensiones de directorio](active-directory-aadconnectsync-feature-directory-extensions.md) en el Asistente de Azure AD Connect, ahora se podrá seleccionar el atributo de Azure AD de tipo cadena Teletexto.

## <a name="111890"></a>1.1.189.0
Fecha de publicación: junio de 2016

**Problemas corregidos y mejoras:**

* Azure AD Connect ahora puede instalarse en un servidor conforme a FIPS.
  * Para la sincronización de contraseñas, consulte [Sincronización de contraseñas y FIPS](active-directory-aadconnectsync-implement-password-synchronization.md#password-synchronization-and-fips).
* Se ha corregido un problema por el cual un nombre NetBIOS no se pudo resolver en el FQDN en el conector de Active Directory.

## <a name="111800"></a>1.1.180.0
Fecha de publicación: mayo de 2016

**Nuevas características:**

* Le advierte y ayuda a comprobar los dominios, si no lo hizo antes de ejecutar Azure AD Connect.
* Se ha agregado compatibilidad con [Microsoft Cloud Germany](active-directory-aadconnect-instances.md#microsoft-cloud-germany).
* Se ha agregado compatibilidad con la versión más reciente de la infraestructura de [Microsoft Azure Government Cloud](active-directory-aadconnect-instances.md#microsoft-azure-government-cloud) con los nuevos requisitos de dirección URL.

**Problemas corregidos y mejoras:**

* Agrega el filtrado en el Editor de reglas de sincronización para facilitar la búsqueda de reglas de sincronización.
* Mejora el rendimiento en la eliminación de un espacio de conector.
* Se corrigió un problema cuando el mismo objeto se eliminaba y se agregaba en la misma ejecución (llamado eliminar o agregar).
* Una regla de sincronización deshabilitada ya no vuelve a habilitar objetos y atributos incluidos durante la actualización del esquema de directorio o una actualización.

## <a name="111300"></a>1.1.130.0
Fecha de publicación: abril de 2016

**Nuevas características:**

* Se ha agregado compatibilidad con atributos multivalor en las [extensiones de directorio](active-directory-aadconnectsync-feature-directory-extensions.md).
* Se ha agregado compatibilidad con más variaciones de configuración para que la [actualización automática](active-directory-aadconnect-feature-automatic-upgrade.md) se considere apta para la actualización.
* Se han agregado algunos cmdlets para el [programador personalizado](active-directory-aadconnectsync-feature-scheduler.md#custom-scheduler).

## <a name="111190"></a>1.1.119.0
Fecha de publicación: marzo de 2016

**Problemas corregidos:**

* Nos aseguramos de que la instalación rápida no se pueda usar en Windows Server 2008 (versión preliminar 2) porque la sincronización de contraseñas no es compatible con este sistema operativo.
* La actualización desde DirSync con una configuración de filtro personalizada no funcionó como se esperaba.
* Cuando se actualiza a una versión más reciente y no hay ningún cambio en la configuración, no se debe programar una importación o sincronización completa.

## <a name="111100"></a>1.1.110.0
Fecha de publicación: febrero de 2016

**Problemas corregidos:**

* La actualización desde versiones anteriores no funciona si la instalación no está en la carpeta predeterminada C:\Archivos de programa.
* Si efectúa la instalación y anula la selección de **Inicie el proceso de sincronización** al final del asistente para instalación, el programador no se habilitará al volver a ejecutar el asistente.
* El programador no funciona según lo previsto en los servidores en los que no se use el formato de fecha y hora US-en. Además, impedirá que `Get-ADSyncScheduler` devuelva las horas correctas.
* Si ha instalado una versión anterior de Azure AD Connect con AD FS como opción de inicio de sesión y actualización, no puede volver a ejecutar el asistente para instalación.

## <a name="111050"></a>1.1.105.0
Fecha de publicación: febrero de 2016

**Nuevas características:**

* [Automatic upgrade](active-directory-aadconnect-feature-automatic-upgrade.md) para los clientes de configuración rápida.
* Compatibilidad con el administrador global mediante Azure Multi-Factor Authentication y Privileged Identity Management en el asistente para la instalación.
  * Si utiliza Multi-Factor Authentication, debe permitir que el servidor proxy también permita el tráfico a https://secure.aadcdn.microsoftonline-p.com.
  * Debe agregar https://secure.aadcdn.microsoftonline-p.com a la lista de sitios de confianza para que Multi-Factor Authentication funcione correctamente.
* Permite cambiar el método de inicio de sesión del usuario después de la instalación inicial.
* Permita el [filtrado de dominios y unidades organizativas](active-directory-aadconnect-get-started-custom.md#domain-and-ou-filtering) en el Asistente para instalación. Esto también permite conectar con bosques donde no todos los dominios están disponibles.
* [Scheduler](active-directory-aadconnectsync-feature-scheduler.md) está integrado en el motor de sincronización.

**Características promocionadas desde la vista previa a GA:**

* [Escritura diferida de dispositivos](active-directory-aadconnect-feature-device-writeback.md)
* [Extensiones de directorio](active-directory-aadconnectsync-feature-directory-extensions.md)

**Nuevas características de la versión preliminar:**

* El nuevo intervalo de ciclo de sincronización predeterminado es de 30 minutos. Solía ser tres horas en todas las versiones anteriores. Agrega compatibilidad para cambiar el comportamiento del [programador](active-directory-aadconnectsync-feature-scheduler.md) .

**Problemas corregidos:**

* La página de comprobación de dominios DNS no siempre reconocía los dominios.
* Solicita las credenciales de administrador de dominio al configurar AD FS.
* El Asistente para instalación no reconoce las cuentas de AD locales si están ubicadas en un dominio con un árbol DNS diferente al dominio raíz.

## <a name="1091310"></a>1.0.9131.0
Fecha de publicación: diciembre de 2015

**Problemas corregidos:**

* La sincronización de contraseñas podría no funcionar al cambiar las contraseñas en Active Directory Domain Services (AD DS), pero funciona al establecer una contraseña.
* Con un servidor proxy, la autenticación en Azure AD puede producir errores durante la instalación o si se cancela una actualización en la página de configuración.
* La actualización desde una versión anterior de Azure AD Connect con una instalación completa de una instancia de SQL Server produce errores si no es administrador del sistema de SQL Server (SA).
* La actualización desde una versión anterior de Azure AD Connect con una instalación remota de SQL Server muestra el error "Unable to access the ADSync SQL database" (No se puede acceder a la base de datos SQL de ADSync).

## <a name="1091250"></a>1.0.9125.0
Fecha de publicación: noviembre de 2015

**Nuevas características:**

* Puede volver a configurar AD FS para la confianza de Azure AD.
* Puede actualizar el esquema de Active Directory y volver a generar reglas de sincronización.
* Puede deshabilitar una regla de sincronización.
* Puede definir "AuthoritativeNull" como un nuevo literal en una regla de sincronización.

**Nuevas características de la versión preliminar:**

* [Azure AD Connect Health para sincronización](../connect-health/active-directory-aadconnect-health-sync.md)
* Compatibilidad para sincronización de contraseñas de [Servicios de dominio de Azure AD](../active-directory-passwords-update-your-own-password.md) .

**Nuevo escenarios admitido:**

* Admite varias organizaciones de Exchange locales. Consulte [Implementaciones híbridas con varios bosques de Active Directory](https://technet.microsoft.com/library/jj873754.aspx) para más información.

**Problemas corregidos:**

* Problemas de sincronización de contraseñas:
  * Un objeto movido desde fuera de ámbito a dentro de ámbito no tendrá su contraseña sincronizada. Esto incluye tanto UO como el filtrado de atributos.
  * La selección de una nueva UO para incluir en sincronización no requiere una sincronización de contraseñas completa.
  * Cuando un usuario deshabilitado se habilita, la contraseña no se sincroniza.
  * La cola de reintentos de contraseña es infinita y el límite anterior de 5.000 objetos para retirar anterior se ha quitado.
<<<<<<< HEAD <<<<<<< HEAD
  * [Solución de problemas mejorada](active-directory-aadconnectsync-implement-password-synchronization.md#troubleshoot-password-synchronization).
=======
  * [Solución de problemas mejorada](active-directory-aadconnectsync-troubleshoot-password-synchronization.md).
>>>>>>> <a name="487b660b6d3bb5ce9e64b6fdbde2ae621cb91922"></a>487b660b6d3bb5ce9e64b6fdbde2ae621cb91922
=======
  * [Solución de problemas mejorada](active-directory-aadconnectsync-troubleshoot-password-synchronization.md).
>>>>>>> 4b2e846c2cd4615f4e4be7195899de11e3957c83
* No se puede conectar con Active Directory con el nivel funcional de bosque de Windows Server 2016.
* No se puede cambiar el grupo usado para el filtrado de grupo tras la instalación inicial.
* Ya no se crea un nuevo perfil de usuario en el servidor de Azure AD Connect para cada usuario al hacer un cambio de contraseña con la escritura diferida de contraseñas habilitada.
* No se pueden usar valores enteros largos en ámbitos de reglas de sincronización.
* La casilla de verificación "Reescritura de dispositivos" permanece deshabilitada si hay controladores de dominio inalcanzables.

## <a name="1086670"></a>1.0.8667.0
Fecha de publicación: agosto de 2015

**Nuevas características:**

* Ahora, el asistente para la instalación de Azure AD Connect se adapta a todos los idiomas de Windows Server.
* Se ha agregado compatibilidad con el desbloqueo de cuentas cuando se usa la administración de contraseñas de Azure AD.

**Problemas corregidos:**

* El asistente para la instalación de Azure AD Connect se bloquea si otro usuario continúa la instalación, y no la persona que la inició por primera vez.
* Si una desinstalación anterior de Azure AD Connect no desinstala limpiamente la sincronización de Azure AD Connect, no es posible volver a instalarlo.
* No se puede instalar Azure AD Connect mediante la instalación rápida si el usuario no está en el dominio raíz del bosque o si se usa una versión distinta del inglés de Active Directory.
* Si no se puede resolver el FQDN de la cuenta de usuario de Active Directory, se muestra un mensaje de error engañoso para indicar que no se pudo confirmar el esquema.
* Si se cambia la cuenta usada en el conector de Active Directory fuera del asistente, este producirá errores en ejecuciones posteriores.
* Azure AD Connect no se puede instalar en ocasiones en un controlador de dominio.
* No se puede habilitar y deshabilitar el "Modo provisional" si se han agregado atributos de extensión.
* La escritura diferida de contraseñas produce errores en alguna configuración debido a una contraseña incorrecta en el conector de Active Directory.
* No se puede actualizar la sincronización de directorios si se usa en el filtrado de atributos el nombre distintivo (DN).
* Uso excesivo de CPU al utilizar el restablecimiento de contraseña.

**Características de versión la preliminar eliminadas:**

* La característica en vista previa [Reescritura de usuarios](active-directory-aadconnect-feature-preview.md#user-writeback) se ha eliminado temporalmente a raíz de los comentarios de nuestros clientes de vista previa. Se volverá a agregar después de que se hayan examinado los comentarios proporcionados.

## <a name="1086410"></a>1.0.8641.0
Fecha de publicación: junio de 2015

**Versión inicial de Azure AD Connect.**

Ha cambiado el nombre de Azure AD Sync a Azure AD Connect.

**Nuevas características:**

* Instalación de la [configuración rápida](active-directory-aadconnect-get-started-express.md)
* Posibilidad de [configurar AD FS](active-directory-aadconnect-get-started-custom.md#configuring-federation-with-ad-fs)
* Posibilidad de [actualizar desde DirSync](active-directory-aadconnect-dirsync-upgrade-get-started.md)
* [Evitar eliminaciones accidentales](active-directory-aadconnectsync-feature-prevent-accidental-deletes.md)
* Se ha introducido el [modo de ensayo](active-directory-aadconnectsync-operations.md#staging-mode)

**Nuevas características de la versión preliminar:**

* [Reescritura de usuarios](active-directory-aadconnect-feature-preview.md#user-writeback)
* [Escritura diferida de grupos](active-directory-aadconnect-feature-preview.md#group-writeback)
* [Escritura diferida de dispositivos](active-directory-aadconnect-feature-device-writeback.md)
* [Extensiones de directorio](active-directory-aadconnect-feature-preview.md)

## <a name="104940501"></a>1.0.494.0501
Fecha de publicación: mayo de 2015

**Nuevo requisito**

* Ahora, Azure AD Sync requiere que se instale la versión 4.5.1 de .NET Framework.

**Problemas corregidos:**

* La escritura diferida de contraseñas de Azure AD produce errores de conectividad de Service Bus.

## <a name="104910413"></a>1.0.491.0413
Fecha de publicación: abril de 2015

**Problemas corregidos y mejoras:**

* El conector de Active Directory no procesa las eliminaciones correctamente si está habilitada la Papelera de reciclaje y hay varios dominios en el bosque.
* Se ha mejorado el rendimiento de las operaciones de importación para el conector de Azure Active Directory.
* Cuando un grupo superaba el límite de pertenencia (de forma predeterminada, el límite se establece en 50 000 objetos), el grupo se eliminaba de Azure Active Directory. Con el nuevo comportamiento, no se elimina el grupo, se produce un error y no se exportan los cambios de pertenencia.
* No se puede aprovisionar un nuevo objeto si una eliminación preconfigurada con el mismo DN ya está presente en el espacio del conector.
* Algunos objetos se marcan para que se sincronicen durante una sincronización delta, aunque no haya ningún cambio preconfigurado en el objeto.
* Forzar una sincronización de contraseñas también elimina la lista preferida de controladores de dominio.
* CSExportAnalyzer tiene problemas con algunos estados de objetos.

**Nuevas características:**

* Una combinación puede conectarse ahora a "CUALQUIER" tipo de objeto en la MV.

## <a name="104850222"></a>1.0.485.0222
Fecha de publicación: febrero de 2015

**Mejoras:**

* Rendimiento de importación mejorada.

**Problemas corregidos:**

* La sincronización de contraseñas respeta el atributo cloudFiltered que usa el filtrado de atributos. Los objetos filtrados ya no pertenecen al ámbito de la sincronización de contraseñas.
* En las raras ocasiones donde la topología tenía muchos controladores de dominio, la sincronización de contraseñas no funcionaba.
* Se ha habilitado en Azure AD/Intune "Servidor detenido" al importar desde el conector de Azure AD después de la administración de dispositivos.
* La unión de entidades de seguridad externas (FSP) desde varios dominios del mismo bosque produce un error de unión ambigua.

## <a name="104751202"></a>1.0.475.1202
Fecha de publicación: diciembre de 2014

**Nuevas características:**

* Ahora es posible realizar la sincronización de contraseñas con filtrado basado en atributos. Para más información, consulte el artículo sobre la [sincronización de contraseñas con filtrado](active-directory-aadconnectsync-configure-filtering.md).
* El atributo msDS-ExternalDirectoryObjectID se reescribe en Active Directory. Esta característica agrega compatibilidad para las aplicaciones de Office 365. Utiliza OAuth2 para acceder a los buzones de correo en línea y locales en una implementación híbrida de Exchange.

**Problemas de actualización corregidos:**

* Hay una versión más reciente del ayudante de inicio de sesión en el servidor.
* Se usó una ruta de acceso de instalación personalizada para instalar Azure AD Sync.
* Un criterio de combinación personalizado no válido bloquea la actualización.

**Otras correcciones:**

* Se han corregido las plantillas para Office Pro Plus.
* Se han corregido los problemas de instalación ocasionados por nombres de usuario que comienzan con un guión.
* Se ha corregido la pérdida de la configuración de sourceAnchor cuando se ejecuta al asistente para la instalación por una segunda vez.
* Se ha corregido el seguimiento ETW para la sincronización de contraseñas.

## <a name="104701023"></a>1.0.470.1023
Fecha de publicación: octubre de 2014

**Nuevas características:**

* Sincronización de contraseñas desde varias instancias de Active Directory locales a Azure AD.
* Interfaz de usuario de instalación localizada para todos los idiomas de Windows Server.

**Actualización de AADSync 1.0 GA**

Si ya tiene instalado Azure AD Sync, hay un paso adicional que debe seguir en caso de que haya cambiado alguna de las reglas de sincronización inmediata. Después de haber actualizado a la versión 1.0.470.1023, las reglas de sincronización que ha modificado se duplican. Para cada regla de sincronización modificada, haga lo siguiente:

1.  Busque la regla de sincronización modificada y anote los cambios.
* Elimine la regla de sincronización.
* Busque la nueva regla de sincronización creada por Azure AD Sync y vuelva a aplicar los cambios.

**Permisos para la cuenta de Active Directory**

Se deben conceder permisos adicionales a la cuenta de Active Directory para poder leer los hash de contraseña de Active Directory. Los permisos que se deben conceder se denominan "Replicating Directory Changes" (Replicar cambios de directorio) y "Replicating Directory Changes All" (Replicar todos los cambios de directorio). Ambos permisos son necesarios para poder leer los hash de contraseña.

## <a name="104190911"></a>1.0.419.0911
Fecha de publicación: septiembre de 2014

**Versión inicial de Azure AD Sync.**

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](active-directory-aadconnect.md).

