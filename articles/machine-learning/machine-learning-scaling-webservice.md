---
title: "Ampliación de la simultaneidad en servicios web de Azure Machine Learning | Microsoft Docs"
description: "Aprenda a ampliar la simultaneidad en los servicios web de Azure Machine Learning incorporando puntos de conexión adicionales."
services: machine-learning
documentationcenter: 
author: neerajkh
manager: srikants
editor: cgronlun
keywords: "aprendizaje automático de azure, servicios web, operacionalización, escalado, punto de conexión, simultaneidad"
ms.assetid: c2c51d7f-fd2d-4f03-bc51-bf47e6969296
ms.service: machine-learning
ms.devlang: NA
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/23/2017
ms.author: neerajkh
translationtype: Human Translation
ms.sourcegitcommit: 197ebd6e37066cb4463d540284ec3f3b074d95e1
ms.openlocfilehash: 013354515d841003c912ac0338690dd975a79ef7
ms.lasthandoff: 03/31/2017


---
# <a name="scaling-an-azure-machine-learning-web-service-by-adding-additional-endpoints"></a>Escalado de un servicio web de Azure Machine Learning mediante la incorporación de puntos de conexión adicionales
> [!NOTE]
> En este tema se describen técnicas que se aplican a un servicio web Machine Learning **clásico**. 
> 
> 

De manera predeterminada, cada servicio web publicado está configurado para admitir 20 solicitudes simultáneas y 200 solicitudes simultáneas como máximo. Aunque el Portal de Azure clásico ofrece una manera de establecer este valor, Azure Machine Learning optimiza automáticamente este valor con el fin de brindar el mejor rendimiento para el servicio web, por lo que se omite el valor del portal. 

Si tiene previsto llamar a la API con una carga mayor que un máximo de 200 llamadas simultáneas, debe crear varios puntos de conexión en el mismo servicio web. Acto seguido, podrá distribuir aleatoriamente la carga entre todos ellos.

El escalado de un servicio web es una tarea común. Algunos de los motivos para ello son admitir más de 200 solicitudes simultáneas, aumentar la disponibilidad a través de varios puntos de conexión u ofrecer puntos de conexión independientes para el servicio web. Puede aumentar la escala agregando más puntos de conexión para el mismo servicio web a través del [Portal de Azure clásico](https://manage.windowsazure.com/) o del portal [Servicio web Azure Machine Learning](https://services.azureml.net/).

Para obtener más información sobre la incorporación de puntos de conexión nuevos, consulte [Creación de puntos de conexión](machine-learning-create-endpoint.md).

Tenga en cuenta que usar un recuento de simultaneidad alto puede ser perjudicial si no se llama a la API con una tasa elevada en consecuencia. Puede que vea tiempos de espera esporádicos o picos de latencia si pone una carga relativamente baja en una API configurada para una carga elevada.

Las API sincrónicas se usan normalmente en situaciones en las que se desea una latencia baja. Latencia aquí implica el tiempo que tarda la API en completar una solicitud, sin contar los retrasos de red. Supongamos que tiene una API con una latencia de 50 ms. Para utilizar totalmente la capacidad disponible con nivel de limitación alto y un número máximo de llamadas simultáneas de 20, debe llamar a esta API 20 * 1000/50 = 400 veces por segundo. Al ampliar esto aún más, un número máximo de llamadas simultáneas de 200 le permitirá llamar a la API 4000 veces por segundo, si presuponemos que la latencia es de 50 ms.

<!--Image references-->
[1]: ./media/machine-learning-scaling-webservice/machlearn-1.png
[2]: ./media/machine-learning-scaling-webservice/machlearn-2.png

