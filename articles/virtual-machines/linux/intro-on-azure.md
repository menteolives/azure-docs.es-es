---
title: "Introducción a Linux en Azure | Microsoft Docs"
description: "Aprenda a utilizar máquinas virtuales de Linux en Azure."
services: virtual-machines-linux
documentationcenter: python
author: szarkos
manager: timlt
editor: 
tags: azure-resource-manager,azure-service-management
ms.assetid: b13bf305-87bf-4df3-815e-e8f6337aa6ea
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 02/02/2017
ms.author: szark
translationtype: Human Translation
ms.sourcegitcommit: eeb56316b337c90cc83455be11917674eba898a3
ms.openlocfilehash: eaac5300292e328bcff9ddf5447bea0e53075179
ms.lasthandoff: 04/03/2017


---
# <a name="introduction-to-linux-on-azure"></a>Introducción a Linux en Azure
En este tema se ofrece información general acerca de algunos aspectos relacionados con el uso de las máquinas virtuales con Linux en la nube de Azure. La implementación de una máquina virtual con Linux es un proceso sencillo cuando se usa una imagen de la galería.

## <a name="authentication-usernames-passwords-and-ssh-keys"></a>Autenticación: Nombres de usuario, contraseñas y claves SSH.
Al crear una máquina virtual Linux con el Portal de Azure clásico, se le pedirá que facilite un nombre de usuario, una contraseña o una clave pública SSH. La elección de un nombre de usuario para implementar una máquina virtual Linux en Azure está sujeta a la siguiente limitación: no se admiten los nombres de cuentas del sistema (UID <100) ya existentes en la máquina virtual, como por ejemplo, "root".

* Consulte [Creación de una máquina virtual que ejecuta Linux](quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
* Consulte [Utilización de SSH con Linux en Azure](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="obtaining-superuser-privileges-using-sudo"></a>Obtención de privilegios de superusuario con el uso de `sudo`
La cuenta de usuario especificada durante la implementación de la instancia de máquina virtual en Azure es una cuenta con privilegios. El Agente de Linux de Azure configura esta cuenta para elevar privilegios a root (cuenta de superusuario) con la utilidad `sudo` . Después de iniciar sesión con esta cuenta de usuario, podrá ejecutar comandos como root con el uso del la sintaxis de comando

    # sudo <COMMAND>

También puede obtener un shell root con **sudo -s**.

* Consulte [Uso de privilegios raíz en máquinas virtuales con Linux en Azure](use-root-privileges.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="firewall-configuration"></a>Configuración del firewall
Azure ofrece un filtro de paquetes de entrada que restringe la conectividad a los puertos especificados en el Portal de Azure clásico. De forma predeterminada, el único puerto permitido es SSH. Puede abrir el acceso a puertos adicionales de la máquina virtual con Linux con la configuración de puntos de conexión en el Portal de Azure clásico:

* Consulte [Configuración de puntos de conexión en una máquina virtual](../windows/classic/setup-endpoints.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).

Las imágenes de Linux de la Galería de Azure no habilitan el firewall *iptables* de forma predeterminada. Si lo desea, el firewall puede configurarse para proporcionar filtrado adicional.

## <a name="hostname-changes"></a>Cambios del nombre de host
Al implementar inicialmente una instancia de una imagen de Linux, se le pide que facilite un nombre de host para la máquina virtual. Cuando la máquina virtual está en ejecución, este nombre de host se publica en los servidores DNS de la plataforma, a fin de que varias máquinas virtuales conectadas entre sí puedan realizar búsquedas de direcciones IP con el uso de nombres de host.

Si desea realizar cambios en el nombre de host después de la implementación de una máquina virtual, use el comando

    # sudo hostname <newname>

El Agente de Linux de Azure incluye la funcionalidad para detectar automáticamente este cambio de nombre y configurar correctamente la máquina virtual para que mantenga este cambio y publica este cambio en los servidores DNS de la plataforma.

* [Guía de usuario del Agente de Linux de Azure](../windows/agent-user-guide.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="cloud-init"></a>Cloud-Init
Las imágenes de **Ubuntu** y **CoreOS** usan cloud-init en Azure, que proporciona funcionalidades adicionales para arrancar una máquina virtual.

* [Inyección de datos personalizados](../windows/classic/inject-custom-data.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)
* [Custom Data and Cloud-Init on Microsoft Azure (Datos personalizados y cloud-init en Microsoft Azure)](https://azure.microsoft.com/blog/2014/04/21/custom-data-and-cloud-init-on-windows-azure/)
* [Create Azure Swap Partitions Using Cloud-Init (Creación de particiones de intercambio de Azure con cloud-init)](https://wiki.ubuntu.com/AzureSwapPartitions)
* [Uso de CoreOS en Azure](https://coreos.com/os/docs/latest/booting-on-azure.html)

## <a name="virtual-machine-image-capture"></a>Captura de imagen de máquina virtual
Azure ofrece la capacidad de capturar el estado de una máquina virtual existente en una imagen que posteriormente puede usarse para implementar instancias adicionales de máquina virtual. El Agente de Linux de Azure puede usarse para la reversión de la personalización realizada durante el proceso de aprovisionamiento. Puede seguir los siguientes pasos para capturar una máquina virtual como una imagen:

1. Ejecute **waagent -deprovision** para deshacer la personalización del aprovisionamiento. O ejecute **waagent -deprovision+user** para eliminar opcionalmente la cuenta de usuario especificada durante el aprovisionamiento y todos los datos asociados.
2. Cierre o apague la máquina virtual.
3. Haga clic en *Capturar* en el Portal de Azure clásico o use Powershell o las herramientas de la CLI para capturar la máquina virtual como una imagen.
   
   * Consulte: [Captura de una máquina virtual Linux para usar como plantilla](classic/capture-image.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json).

## <a name="attaching-disks"></a>Acoplamiento de discos
Las máquinas virtuales disponen de un *disco de recursos* local y temporal acoplado. Debido a que los datos de un disco de recursos podrían no resistir los diversos reinicios, muchas veces los usan aplicaciones y procesos que se ejecutan en la máquina virtual para un almacenamiento de datos transitorio y **temporal** . También se usan para almacenar archivos de intercambio o de paginación para el sistema operativo.

En Linux, el disco de recursos se administra generalmente mediante el agente Linux de Azure y se monta automáticamente en **/mnt/resource** (o **/mnt** en las imágenes de Ubuntu).

> [!NOTE]
> Tenga en cuenta que el disco de recursos es un disco **temporal** , que debe eliminarse y reformatearse cuando se reinicia la máquina virtual.
> 
> 

En Linux el kernel debe poner al disco de datos el nombre `/dev/sdc`y los usuarios tendrán que particionar ese recurso, darle formato y montarlo. Esto se trata paso a paso en el tutorial: [Acoplamiento de un disco de datos a una máquina virtual](../windows/classic/attach-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json).

* **Consulte también:** [Configuración del software RAID en Linux](configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) y  & [Configuración del LVM en una máquina virtual Linux en Azure](configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).


