---
title: Mettre à jour la version IoT Edge sur les appareils - Azure IoT Edge | Microsoft Docs
description: Guide pratique pour mettre à jour des appareils IoT Edge afin qu’ils exécutent les dernières versions du démon de sécurité et le runtime IoT Edge
keywords: ''
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 12/17/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: seodec18
ms.openlocfilehash: dfad3199ba3a9cd2f3bca55be50760ddde676e70
ms.sourcegitcommit: b767a6a118bca386ac6de93ea38f1cc457bb3e4e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2018
ms.locfileid: "53558190"
---
# <a name="update-the-iot-edge-security-daemon-and-runtime"></a>Mettre à jour le runtime et le démon de sécurité IoT Edge

À mesure que le service IoT Edge publie de nouvelles versions, vous souhaitez mettre à jour vos appareils IoT Edge afin qu’ils bénéficient des dernières fonctionnalités et améliorations de la sécurité. Cet article fournit des informations sur la façon de mettre à jour vos appareils IoT Edge quand une nouvelle version est disponible. 

Deux composants d’un appareil IoT Edge doivent être mis à jour si vous souhaitez passer à une version plus récente. Le premier est le démon de sécurité qui s’exécute sur l’appareil et démarre les modules du runtime au démarrage de l’appareil. Le démon de sécurité ne peut être mis à jour qu’à partir de l’appareil lui-même. Le second composant est le runtime, constitué des modules de l’agent IoT Edge et du hub IoT Edge. Selon la façon dont vous structurez votre déploiement, le runtime peut être mis à jour à partir de l’appareil ou à distance. 

>[!IMPORTANT]
>Si vous exécutez Azure IoT Edge sur un appareil Windows, ne mettez pas à jour vers la version 1.0.5 si l’une de ces affirmations s’applique à votre appareil : 
>* Vous n’avez pas mis à niveau votre appareil vers Windows build 17763. La version 1.0.5 d’IoT Edge ne prend pas en charge les builds Windows antérieures à 17763.
>* Vous exécutez des modules Java ou Node.js sur votre appareil Windows. Ignorez la version 1.0.5 même si vous avez mis à jour votre appareil Windows vers la dernière build. 
>
>Pour plus d’informations sur la version 1.0.5 d’IoT Edge, voir les [notes de publication 1.0.5](https://github.com/Azure/azure-iotedge/releases/tag/1.0.5). Pour plus d’informations sur la façon d’empêcher la mise à jour de vos outils de développement vers la dernière version, voir le [blog du développeur IoT](https://aka.ms/dev-win-iot-edge-module).


Pour rechercher la dernière version d’Azure IoT Edge, consultez [Versions d’Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases).

## <a name="update-the-security-daemon"></a>Mettre à jour le démon de sécurité

Le démon de sécurité IoT Edge est un composant natif qui doit être mis à jour à l’aide du gestionnaire de package sur l’appareil IoT Edge. 

Vérifiez la version du démon de sécurité qui s’exécute sur votre appareil à l’aide de la commande `iotedge version`. 

### <a name="linux-devices"></a>Appareils Linux

Sur les appareils Linux, utilisez apt-get ou votre gestionnaire de package approprié pour mettre à jour le démon de sécurité. 

```bash
apt-get update
apt-get install libiothsm iotedge
```

### <a name="windows-devices"></a>Appareils Windows

Sur les appareils Windows, utilisez le script PowerShell pour désinstaller, puis réinstaller le démon de sécurité. Le script d’installation extrait automatiquement la dernière version du démon de sécurité. 

Désinstallez le démon de sécurité dans une session PowerShell d’administrateur. 

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
Uninstall-SecurityDaemon
```

Le fait d’exécuter la commande `Uninstall-SecurityDaemon` sans aucun paramètre supprime le démon de sécurité de votre appareil, ainsi que les deux images de conteneur du runtime. Le fichier config.yaml est conservé sur l’appareil, de même que les données du moteur de conteneur Moby. Le fait de conserver la configuration signifie que vous n’avez pas à fournir de nouveau la chaîne de connexion ou les informations du service Device Provisioning pour votre appareil lors du processus d’installation. 

Réinstallez le démon de sécurité selon que votre appareil IoT Edge utilise des conteneurs Windows ou des conteneurs Linux. Remplacez l’expression **\<Windows or Linux\>** par l’un des systèmes d’exploitation de conteneur. Utilisez l’indicateur **-ExistingConfig** pour pointer vers le fichier config.yaml existant sur votre appareil. 

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
Install-SecurityDaemon -ExistingConfig -ContainerOS <Windows or Linux>
```

Si vous voulez installer une version spécifique du démon de sécurité, téléchargez le fichier iotedged-windows.zip approprié dans les [publications IoT Edge](https://github.com/Azure/azure-iotedge/releases). Utilisez ensuite le paramètre `-OfflineInstallationPath` pour pointer vers l’emplacement du fichier. Pour plus d’informations, voir [Installation hors connexion](how-to-install-iot-edge-windows.md#offline-installation).

## <a name="update-the-runtime-containers"></a>Mettre à jour les conteneurs du runtime

La façon dont vous mettez à jour les conteneurs de l’agent IoT Edge et du hub IoT Edge diffère selon que vous utilisez des étiquettes évolutives (par exemple, 1.0) ou des étiquettes spécifiques (par exemple, 1.0.2) dans votre déploiement. 

Vérifiez la version des modules de l’agent IoT Edge et du hub IoT Edge sur votre appareil à l’aide des commandes `iotedge logs edgeAgent` ou `iotedge logs edgeHub`. 

  ![Rechercher la version du conteneur dans les journaux](./media/how-to-update-iot-edge/container-version.png)

### <a name="understand-iot-edge-tags"></a>Comprendre les étiquettes IoT Edge

Les images de l’agent IoT Edge et du hub IoT Edge sont marquées avec la version IoT Edge à laquelle elles sont associées. Il existe deux façons d’utiliser des étiquettes avec les images de runtime : 

* **Étiquettes évolutives** : utilisez uniquement les deux premières valeurs du numéro de version pour obtenir la dernière image qui correspond à ces chiffres. Par exemple, 1.0 est mis à jour à chaque nouvelle version pour pointer vers la dernière version 1.0.x. Si le runtime du conteneur sur votre appareil IoT Edge réextrait l’image, les modules de runtime sont mis à jour vers la dernière version. Cette approche est conseillée à des fins de développement. Les déploiements à partir du portail Azure adoptent par défaut des étiquettes évolutives. 
* **Étiquettes spécifiques** : utilisez les trois valeurs du numéro de version pour définir explicitement la version de l’image. Par exemple, la version 1.0.2 ne change pas après sa publication initiale. Vous pouvez déclarer un nouveau numéro de version dans le manifeste de déploiement quand vous êtes prêt à effectuer une mise à jour. Cette approche est conseillée à des fins de production.

### <a name="update-a-rolling-tag-image"></a>Mettre à jour une image avec des étiquettes évolutives

Si vous utilisez des étiquettes évolutives dans votre déploiement (par exemple, mcr.microsoft.com/azureiotedge-hub:**1.0**), vous devez forcer le runtime du conteneur sur votre appareil à extraire la dernière version de l’image. 

Supprimez la version locale de l’image de votre appareil IoT Edge. Sur les ordinateurs Windows, la désinstallation du démon de sécurité entraîne également la suppression des images du runtime, ce qui permet de sauter cette étape. 

```cmd/sh
docker rmi mcr.microsoft.com/azureiotedge-hub:1.0
docker rmi mcr.microsoft.com/azureiotedge-agent:1.0
```

Vous devrez peut-être utiliser l’indicateur `-f` (forcer) pour supprimer les images. 

Le service IoT Edge extrait les dernières versions des images de runtime et les démarre automatiquement sur votre appareil à nouveau. 

### <a name="update-a-specific-tag-image"></a>Mettre à jour une image avec des étiquettes spécifiques

Si vous utilisez des étiquettes spécifiques dans votre déploiement (par exemple, mcr.microsoft.com/azureiotedge-hub:**1.0.2**), il vous suffit de mettre à jour l’étiquette dans votre manifeste de déploiement et d’appliquer les modifications à votre appareil. 

Dans le portail Azure, les images de déploiement de runtime sont déclarées dans la section **Configurer les paramètres avancés du runtime Edge**. 

[Configurer les paramètres avancés du runtime Edge](./media/how-to-update-iot-edge/configure-runtime.png)

Dans un manifeste de déploiement JSON, mettez à jour les images de modules dans la section **systemModules**. 

```json
"systemModules": {
  "edgeAgent": {
    "type": "docker",
    "settings": {
      "image": "mcr.microsoft.com/azureiotedge-agent:1.0.2",
      "createOptions": ""
    }
  },
  "edgeHub": {
    "type": "docker",
    "status": "running",
    "restartPolicy": "always",
    "settings": {
      "image": "mcr.microsoft.com/azureiotedge-hub:1.0.2",
      "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"5671/tcp\":[{\"HostPort\":\"5671\"}], \"8883/tcp\":[{\"HostPort\":\"8883\"}],\"443/tcp\":[{\"HostPort\":\"443\"}]}}}"
    }
  }
},
```

## <a name="next-steps"></a>Étapes suivantes

Déterminez les dernières [versions d’Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases).

Restez informé des dernières mises à jour et annonces en consultant le [blog Internet of Things](https://azure.microsoft.com/blog/topics/internet-of-things/) 