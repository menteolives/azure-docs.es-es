---
title: "Configuración del firewall de aplicaciones web en una instancia de Azure Application Gateway | Microsoft Docs"
description: "En este artículo se proporcionan instrucciones sobre cómo comenzar a usar el firewall de aplicaciones web en una puerta de enlace de aplicaciones nueva o existente."
documentationcenter: na
services: application-gateway
author: georgewallace
manager: timlt
editor: tysonn
ms.assetid: 670b9732-874b-43e6-843b-d2585c160982
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/22/2017
ms.author: gwallace
translationtype: Human Translation
ms.sourcegitcommit: 432752c895fca3721e78fb6eb17b5a3e5c4ca495
ms.openlocfilehash: 52b7728c3fc702e37f5c5fe3d6544117a11464e8
ms.lasthandoff: 03/30/2017


---
# <a name="configure-web-application-firewall-on-a-new-or-existing-application-gateway"></a>Configuración del firewall de aplicaciones web en una puerta de enlace de aplicaciones nueva o existente

> [!div class="op_single_selector"]
> * [Azure Portal](application-gateway-web-application-firewall-portal.md)
> * [PowerShell del Administrador de recursos de Azure](application-gateway-web-application-firewall-powershell.md)

El firewall de aplicaciones web (WAF) de Azure Application Gateway protege las aplicaciones web de ataques web comunes, como inyección de código SQL, ataques de scripts entre sitios y secuestros de sesiones.

Puerta de enlace de aplicaciones de Azure es un equilibrador de carga de nivel 7. Proporciona conmutación por error, solicitudes HTTP de enrutamiento de rendimiento entre distintos servidores, independientemente de que se encuentren en la nube o en una implementación local. La aplicación proporciona numerosas características de entrega de aplicaciones (ADC), entre las que se incluyen el equilibrio de carga HTTP, la afinidad de sesiones basada en cookies, la descarga SSL (capa de sockets seguros), los sondeos personalizados sobre el estado y la compatibilidad con sitios múltiples. Para obtener una lista completa de las características admitidas, consulte la información general sobre Application Gateway.

En el artículo siguiente se muestra cómo [agregar el firewall de aplicaciones web a una puerta de enlace de aplicaciones existente](#add-web-application-firewall-to-an-existing-application-gateway) y [crear una puerta de enlace de aplicaciones que usa el firewall de aplicaciones web](#create-an-application-gateway-with-web-application-firewall).

![imagen de escenario][scenario]

## <a name="waf-configuration-differences"></a>Diferencias de configuración de WAF

Si ha leído [Creación de una puerta de enlace de aplicaciones con PowerShell](application-gateway-create-gateway-arm.md), comprenderá los valores de SKU que se deben configurar al crear una puerta de enlace de aplicaciones. WAF proporciona una configuración adicional que se puede definir al configurar la SKU en una puerta de enlace de aplicaciones. No hay ningún otro cambio que deba realizar en la puerta de enlace de aplicaciones propiamente dicha.

**SKU**: una puerta de enlace de aplicaciones normal sin WAF admite los tamaños **Standard\_Small**, **Standard\_Medium** y **Standard\_Large**. Con la introducción de WAF, hay dos SKU adicionales, **WAF\_Medium** y **WAF\_Large**. No se admite WAF en puertas de enlace de aplicaciones de pequeño tamaño.

**Nivel**: los valores disponibles son **Estándar** o **WAF**. Cuando se usa el firewall de aplicaciones web, se debe seleccionar **WAF**.

**Modo** : este valor es el modo de WAF. Los valores permitidos son **Detección** y **Prevención**. Cuando WAF está configurado en modo de detección, todas las amenazas se almacenan en un archivo de registro. En modo de prevención, los eventos se siguen registrando pero el atacante recibe una respuesta 403 no autorizado de la puerta de enlace de aplicaciones.

## <a name="add-web-application-firewall-to-an-existing-application-gateway"></a>Adición del firewall de aplicaciones web a una puerta de enlace de aplicaciones existente

Asegúrese de que está usando la versión más reciente de Azure PowerShell. Hay más información disponible en [Uso de Windows PowerShell con Resource Manager](../powershell-azure-resource-manager.md).

### <a name="step-1"></a>Paso 1

Inicie sesión en la cuenta de Azure.

```powershell
Login-AzureRmAccount
```

### <a name="step-2"></a>Paso 2

Seleccione la suscripción que se usará en este escenario.

```powershell
Select-AzureRmSubscription -SubscriptionName "<Subscription name>"
```

### <a name="step-3"></a>Paso 3

Recupere la puerta de enlace a la que va a agregar el firewall de aplicaciones web.

```powershell
$gw = Get-AzureRmApplicationGateway -Name "AdatumGateway" -ResourceGroupName "MyResourceGroup"
```

### <a name="step-4"></a>Paso 4

Configure la SKU del firewall de aplicaciones web. Los tamaños disponibles son **WAF\_Large** y **WAF\_Medium**. Cuando se usa un firewall de aplicaciones web el nivel debe ser **WAF** y la capacidad se debe confirmar cuando se establece la SKU.

```powershell
$gw | Set-AzureRmApplicationGatewaySku -Name WAF_Large -Tier WAF -Capacity 2
```

### <a name="step-5"></a>Paso 5

Configure las opciones de WAF, tal como se define en el ejemplo siguiente:

Para el valor **WafMode** , los valores disponibles son Detección y Prevención.

```powershell
$gw | Set-AzureRmApplicationGatewayWebApplicationFirewallConfiguration -Enabled $true -FirewallMode Prevention
```

### <a name="step-6"></a>Paso 6

Actualice la puerta de enlace de aplicaciones con la configuración definida en el paso anterior.

```powershell
Set-AzureRmApplicationGateway -ApplicationGateway $gw
```

Este comando actualiza la puerta de enlace de aplicaciones con el firewall de aplicaciones web. Se recomienda consultar [Diagnósticos de Application Gateway](application-gateway-diagnostics.md) para entender cómo ver los registros de la puerta de enlace de aplicaciones. Debido a la naturaleza de la seguridad de WAF, los registros se deben revisar periódicamente para entender la postura de seguridad de las aplicaciones web.

## <a name="create-an-application-gateway-with-web-application-firewall"></a>Creación de una puerta de enlace de aplicaciones con el firewall de aplicaciones web

Los pasos siguientes le llevan por el proceso entero de creación de una puerta de enlace de aplicaciones con el firewall de aplicaciones web.

Asegúrese de que está usando la versión más reciente de Azure PowerShell. Hay más información disponible en [Uso de Windows PowerShell con Resource Manager](../powershell-azure-resource-manager.md).

### <a name="step-1"></a>Paso 1

Inicie sesión en Azure.

```powershell
Login-AzureRmAccount
```

Se le solicita que se autentique con sus credenciales.

### <a name="step-2"></a>Paso 2

Compruebe las suscripciones para la cuenta.

```powershell
Get-AzureRmSubscription
```

### <a name="step-3"></a>Paso 3

Elección de la suscripción de Azure que se va a usar.

```powershell
Select-AzureRmsubscription -SubscriptionName "<Subscription name>"
```

### <a name="step-4"></a>Paso 4

Cree un grupo de recursos (omita este paso si usa uno existente).

```powershell
New-AzureRmResourceGroup -Name appgw-rg -Location "West US"
```

El Administrador de recursos de Azure requiere que todos los grupos de recursos especifiquen una ubicación. Esta ubicación se utiliza como ubicación predeterminada para los recursos de ese grupo de recursos. Asegúrese de que todos los comandos para crear una puerta de enlace de aplicaciones usan el mismo grupo de recursos.

En el ejemplo anterior, creamos un grupo de recursos denominado "appgw-RG" y la ubicación "West US".

> [!NOTE]
> Si necesita configurar un sondeo personalizado para la puerta de enlace de aplicaciones, consulte [Creación de una puerta de enlace de aplicaciones con sondeos personalizados mediante PowerShell](application-gateway-create-probe-ps.md). Para más información, consulte [Información general sobre la supervisión de estado de la puerta de enlace de aplicaciones](application-gateway-probe-overview.md) .

### <a name="step-5"></a>Paso 5

Asigne un intervalo de direcciones para la subred de la puerta de enlace de aplicaciones propiamente dicha.

```powershell
$gwSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name 'appgwsubnet' -AddressPrefix 10.0.0.0/24
```

> [!NOTE]
> Una subred de una aplicación debe tener como mínimo 28 bits de máscara. Este valor deja 10 direcciones disponibles en la subred para instancias de Application Gateway. Con una subred más pequeña, puede que no sea capaz de agregar más instancias de su puerta de enlace de aplicaciones en el futuro.


### <a name="step-6"></a>Paso 6

Asigne un intervalo de direcciones para el grupo de direcciones de back-end.

```powershell
$nicSubnet = New-AzureRmVirtualNetworkSubnetConfig  -Name 'appsubnet' -AddressPrefix 10.0.2.0/24
```

### <a name="step-7"></a>Paso 7

Cree una red virtual con las subredes anteriores en el grupo de recursos creado en el paso: [Creación del grupo de recursos](#create-the-resource-group)

```powershell
$vnet = New-AzureRmvirtualNetwork -Name 'appgwvnet' -ResourceGroupName appgw-rg -Location "West US" -AddressPrefix 10.0.0.0/16 -Subnet $gwSubnet, $nicSubnet
```

### <a name="step-8"></a>Paso 8

Recupere el recurso de red virtual y los recursos de subred que se usarán en los pasos siguientes:

```powershell
$vnet = Get-AzureRmvirtualNetwork -Name 'appgwvnet' -ResourceGroupName appgw-rg
$gwSubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'appgwsubnet' -VirtualNetwork $vnet
$nicSubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'appsubnet' -VirtualNetwork $vnet
```

### <a name="step-9"></a>Paso 9:

Cree un recurso de IP pública para la puerta de enlace de aplicaciones. Esta dirección IP pública se usa en uno de los siguientes pasos:

```powershell
$publicip = New-AzureRmPublicIpAddress -ResourceGroupName appgw-rg -name 'appgwpip' -Location "West US" -AllocationMethod Dynamic
```

> [!IMPORTANT]
> Application Gateway no admite el uso de una dirección IP pública creada con una etiqueta de dominio definida. Solo se admite una dirección IP pública con una etiqueta de dominio creada dinámicamente. Si necesita un nombre DNS descriptivo para la puerta de enlace de aplicaciones, se recomienda usar un registro CNAME como alias.


### <a name="step-10"></a>Paso 10

Debe configurar todos los elementos de configuración antes de crear la puerta de enlace de aplicaciones de la aplicación. En los pasos siguientes, se crean los elementos de configuración necesarios para un recurso de puerta de enlace de aplicaciones.

Cree una configuración de IP de puerta de enlace de aplicaciones. Esta configuración determina la subred que usará Application Gateway. Cuando se inicia la Puerta de enlace de aplicaciones, elige una dirección IP de la subred configurada y redirige el tráfico de red a las direcciones IP en el grupo IP de back-end. Tenga en cuenta que cada instancia toma una dirección IP.

```powershell
$gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name 'gwconfig' -Subnet $gwSubnet
```

### <a name="step-11"></a>Paso 11

Configure el grupo de direcciones IP de back-end con las direcciones IP de los servidores web de back-end. Estas direcciones IP son las direcciones que reciben el tráfico de red procedente del punto de conexión de la IP del front-end. Reemplazará las siguientes direcciones IP para agregar sus propios puntos de conexión de dirección IP de la aplicación.

```powershell
$pool = New-AzureRmApplicationGatewayBackendAddressPool -Name 'pool01' -BackendIPAddresses 1.1.1.1, 2.2.2.2, 3.3.3.3
```

### <a name="step-12"></a>Paso 12

Cargue el certificado que se usará en los recursos del grupo de back-end habilitado para SSL.

```powershell
$authcert = New-AzureRmApplicationGatewayAuthenticationCertificate -Name 'whitelistcert1' -CertificateFile <full path to .cer file>
```

### <a name="step-13"></a>Paso 13

Configure los valores HTTP de back-end de la puerta de enlace de aplicaciones. Asigne el certificado cargado en el paso anterior a la configuración HTTP.

```powershell
$poolSetting = New-AzureRmApplicationGatewayBackendHttpSettings -Name 'setting01' -Port 443 -Protocol Https -CookieBasedAffinity Enabled -AuthenticationCertificates $authcert
```

### <a name="step-14"></a>Paso 14

Configure el puerto IP de front-end para el punto de conexión de IP pública. Este puerto es el puerto al que se conectan los usuarios finales.

```powershell
$fp = New-AzureRmApplicationGatewayFrontendPort -Name 'port01'  -Port 443
```

### <a name="step-15"></a>Paso 15

Cree una configuración IP de front-end. Esta configuración asigna una dirección IP pública o privada al front-end de la puerta de enlace de aplicaciones. El paso siguiente asocia la dirección IP pública del paso anterior con la configuración IP de front-end.

```powershell
$fipconfig = New-AzureRmApplicationGatewayFrontendIPConfig -Name 'fip01' -PublicIPAddress $publicip
```

### <a name="step-16"></a>Paso 16

Configure el certificado de la puerta de enlace de aplicaciones. Este certificado se usa para descifrar y volver a cifrar el tráfico de la puerta de enlace de aplicaciones.

```powershell
$cert = New-AzureRmApplicationGatewaySslCertificate -Name cert01 -CertificateFile <full path to .pfx file> -Password <password for certificate file>
```

### <a name="step-17"></a>Paso 17

Cree el agente de escucha HTTP para la puerta de enlace de aplicaciones. Asigne la configuración IP de front-end, el puerto y el certificado SSL que se usarán.

```powershell
$listener = New-AzureRmApplicationGatewayHttpListener -Name listener01 -Protocol Https -FrontendIPConfiguration $fipconfig -FrontendPort $fp -SslCertificate $cert
```

### <a name="step-18"></a>Paso 18

Cree una regla de enrutamiento del equilibrador de carga que configure el comportamiento del equilibrador de carga. En este ejemplo, se crea una regla básica de operaciones por turnos.

```powershell
$rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name 'rule01' -RuleType basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool
```

### <a name="step-19"></a>Paso 19

Configure el tamaño de la instancia de la puerta de enlace de aplicaciones.

```powershell
$sku = New-AzureRmApplicationGatewaySku -Name WAF_Medium -Tier WAF -Capacity 2
```

> [!NOTE]
> Puede elegir entre **WAF\_Medium** y **WAF\_Large**; el nivel cuando se usa WAF siempre es **WAF**. La capacidad es cualquier número entre 1 y 10.

### <a name="step-20"></a>Paso 20

Configure el modo de WAF. Los valores aceptables son **Prevención** y **Detección**.

```powershell
$config = New-AzureRmApplicationGatewayWebApplicationFirewallConfiguration -Enabled $true -FirewallMode "Prevention"
```

### <a name="step-21"></a>Paso 21

Cree una puerta de enlace de aplicaciones con todos los elementos de configuración de los pasos anteriores. En el ejemplo, la puerta de enlace de aplicaciones se denomina "appgwtest".

```powershell
$appgw = New-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Location "West US" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku -WebApplicationFirewallConfig $config -SslCertificates $cert -AuthenticationCertificates $authcert
```

> [!NOTE]
> Las puertas de enlace de la aplicación creadas con la configuración de firewall de aplicación web básica se configuran con CRS 3.0 para protección.

## <a name="get-application-gateway-dns-name"></a>Obtención del nombre DNS de una puerta de enlace de aplicaciones

Una vez creada la puerta de enlace, el siguiente paso es configurar el front-end para la comunicación. Cuando se utiliza una dirección IP pública, la puerta de enlace de aplicaciones requiere un nombre DNS asignado dinámicamente, que no es descriptivo. Para asegurarse de que los usuarios finales puedan llegar a la Application Gateway, se puede utilizar un registro CNAME para que apunte al punto de conexión público de la Application Gateway. [Configuración de un nombre de dominio personalizado en Azure](../cloud-services/cloud-services-custom-domain-name-portal.md). Para ello, recupere los detalles de la puerta de enlace de aplicaciones y su nombre DNS o IP asociados mediante el elemento PublicIPAddress asociado a la puerta de enlace de aplicaciones. El nombre DNS de la puerta de enlace de aplicaciones se debe utilizar para crear un registro CNAME, que apunta las dos aplicaciones web a este nombre DNS. No se recomienda el uso de registros A, ya que la VIP puede cambiar al reiniciarse la puerta de enlace de aplicaciones.

```powershell
Get-AzureRmPublicIpAddress -ResourceGroupName appgw-RG -Name publicIP01
```

```
Name                     : publicIP01
ResourceGroupName        : appgw-RG
Location                 : westus
Id                       : /subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/publicIPAddresses/publicIP01
Etag                     : W/"00000d5b-54ed-4907-bae8-99bd5766d0e5"
ResourceGuid             : 00000000-0000-0000-0000-000000000000
ProvisioningState        : Succeeded
Tags                     : 
PublicIpAllocationMethod : Dynamic
IpAddress                : xx.xx.xxx.xx
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : {
                                "Id": "/subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/applicationGateways/appgwtest/frontendIP
                            Configurations/frontend1"
                            }
DnsSettings              : {
                                "Fqdn": "00000000-0000-xxxx-xxxx-xxxxxxxxxxxx.cloudapp.net"
                            }
```

## <a name="next-steps"></a>Pasos siguientes

Aprenda a configurar el registro de diagnóstico para registrar los eventos que se detectan o impiden con el firewall de aplicaciones web en [Diagnósticos de Application Gateway](application-gateway-diagnostics.md).

[scenario]: ./media/application-gateway-web-application-firewall-powershell/scenario.png

