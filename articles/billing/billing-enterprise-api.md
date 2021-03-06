---
title: "API de facturación de Azure Enterprise | Microsoft Docs"
description: "Obtenga información acerca de las API de informes que permiten a los clientes de Azure Enterprise extraer datos de consumo mediante programación."
services: 
documentationcenter: 
author: aedwin
manager: aedwin
editor: 
tags: billing
ms.assetid: 3e817b43-0696-400c-a02e-47b7817f9b77
ms.service: billing
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: billing
ms.date: 04/25/2017
ms.author: aedwin
ms.translationtype: Human Translation
ms.sourcegitcommit: f6006d5e83ad74f386ca23fe52879bfbc9394c0f
ms.openlocfilehash: 343b71e28adfd32295b837a40ecf64083341b972
ms.contentlocale: es-es
ms.lasthandoff: 05/03/2017


---
# <a name="overview-of-reporting-apis-for-enterprise-customers-preview"></a>Información general de API de informes para clientes de Enterprise (versión preliminar)
Las API de informes permiten a los clientes de Azure Enterprise extraer datos de facturación y consumo mediante programación en las herramientas de análisis de datos preferidas. 

## <a name="enabling-data-access-to-the-api"></a>Habilitación del acceso de datos a la API
* **Generar/recuperar clave de API**: inicie sesión en el portal de Enterprise y siga el tutorial en Ayuda - API de informes. La primera sección de este artículo de ayuda explica cómo generar/recuperar la clave de API para la inscripción especificada.
* **Pasar claves en la API**: la clave de API tiene que pasarse para cada llamada para la autenticación y autorización. La siguiente propiedad tiene que ser los encabezados HTTP.

|Clave de encabezado de solicitud | Valor|
|-|-|
|Autorización| Especifique el valor con este formato: **bearer {API_KEY}** <br/> Ejemplo: bearer eyr....09|

## <a name="consumption-apis"></a>API de consumo
Hay disponible un punto de conexión de Swagger [aquí](https://consumption.azure.com/v1/swagger) para la API descrita a continuación que debe habilitar una introspección sencilla de la API y la capacidad de generar SDK de cliente con [AutoRest](https://github.com/Azure/AutoRest) o [Swagger CodeGen](http://swagger.io/swagger-codegen/). Los datos a partir del 1 de mayo de 2014 están disponibles a través de esta API. 

* **Saldos y resumen**: la [API de saldos y resumen](billing-enterprise-api-balance-summary.md) ofrece un resumen mensual de información sobre saldos, nuevas compras, gastos de servicios en Azure Marketplace, ajustes y gastos de uso por encima del límite.

* **Detalles de uso**: la [API de detalles de uso](billing-enterprise-api-usage-detail.md) ofrece un desglose diario de cantidades consumidas y gastos estimados por una inscripción. El resultado también incluye información sobre instancias, medidores y departamentos. La API se puede consultar por período de facturación o por una fecha de inicio y finalización especificada. 

* **Gasto en la tienda Marketplace**: la [API de gastos en la tienda Marketplace](billing-enterprise-api-marketplace-storecharge.md) devuelve el desglose de los gastos de Marketplace basado en el uso por día para el período de facturación o las fechas de inicio y finalización especificadas (no se incluyen las cuotas de una vez).

* **Hoja de precios**: la [API de hoja de precios](billing-enterprise-api-pricesheet.md) proporciona el tipo aplicable de cada medidor para la inscripción y el período de facturación determinados. 

## <a name="helper-apis"></a>API de ayuda
 **Enumerar períodos de facturación**: la [API de períodos de facturación](billing-enterprise-api-billing-periods.md) devuelve una lista de períodos de facturación que tienen datos de consumo para la inscripción especificada en orden cronológico inverso. Cada período contiene una propiedad que señala a la ruta de la API para los cuatro conjuntos de datos: BalanceSummary, UsageDetails, Marketplace Charges y PriceSheet.


## <a name="api-response-codes"></a>Códigos de respuesta de la API  
|Código de estado de respuesta|Message|Descripción|
|-|-|-|
|200| OK|Sin errores|
|401| No autorizado| Clave de API no encontrada, no válida, expirada, etc.|
|404| No disponible| Punto de conexión de informe no encontrado|
|400| Bad Request| Parámetros no válidos: intervalos de fechas, números EA, etc.|
|500| Error de servidor| Error inesperado al procesar la solicitud| 










