---
title: Télécharger les outils Azure Stack à partir de GitHub | Microsoft Docs
description: Découvrez comment télécharger les outils nécessaires pour utiliser Azure Stack.
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: PowerShell
ms.topic: article
ms.date: 01/14/2019
ms.author: mabrigg
ms.reviewer: thoroet
ms.lastreviewed: 01/14/2019
ms.openlocfilehash: 05dd3b292f90964c6af21890aaeafab9849a09ed
ms.sourcegitcommit: 898b2936e3d6d3a8366cfcccc0fccfdb0fc781b4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/30/2019
ms.locfileid: "55242975"
---
# <a name="download-azure-stack-tools-from-github"></a>Télécharger les outils Azure Stack à partir de GitHub

*S’applique à : systèmes intégrés Azure Stack et Kit de développement Azure Stack*

**AzureStack-Tools** est un [référentiel GitHub](https://github.com/Azure/AzureStack-Tools) qui héberge des modules PowerShell permettant de gérer et déployer des ressources sur Azure Stack. Si vous prévoyez d’établir la connectivité VPN, vous pouvez télécharger ces modules PowerShell avec le Kit de développement Azure Stack ou sur un client externe Windows. Pour obtenir ces outils, clonez le référentiel GitHub ou téléchargez le dossier **AzureStack-Tools** en exécutant le script suivant :

```PowerShell
# Change directory to the root directory. 
cd \

# Download the tools archive.
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 
invoke-webrequest `
  https://github.com/Azure/AzureStack-Tools/archive/master.zip `
  -OutFile master.zip

# Expand the downloaded files.
expand-archive master.zip `
  -DestinationPath . `
  -Force

# Change to the tools directory.
cd AzureStack-Tools-master

```

## <a name="functionality-provided-by-the-modules"></a>Fonctionnalités fournies par les modules

Le dépôt **AzureStack-Tools** contient des modules PowerShell qui prennent en charge les fonctionnalités suivantes pour Azure Stack :  

| Fonctionnalités | Description | Qui peut utiliser ce module ? |
| --- | --- | --- |
| [Fonctionnalités de cloud](user/azure-stack-validate-templates.md) | Utilisez ce module pour obtenir les fonctionnalités cloud d’un cloud. Par exemple, en utilisant ce module, vous pouvez obtenir les fonctionnalités de cloud telles que la version d’API et les ressources Azure Resource Manager. Vous pouvez également obtenir les extensions de machine virtuelle pour Azure Stack et les clouds Azure à l’aide de ce module. | Les opérateurs et les utilisateurs de cloud |
| [Stratégie Resource Manager pour Azure Stack](user/azure-stack-policy-module.md) | Utilisez ce module pour configurer un abonnement Azure ou un groupe de ressources Azure avec la même gestion des versions et la même disponibilité de service qu’Azure Stack. | Les opérateurs et les utilisateurs de cloud |
| [Inscription auprès d’Azure](azure-stack-register.md) | Utilisez ce module pour inscrire votre instance de Kit de développement auprès d’Azure. Une fois l’inscription effectuée, vous pouvez télécharger les éléments de Place de Marché à partir d’Azure et les utiliser dans Azure Stack. | Les opérateurs de cloud |
| [Déploiement Azure Stack](azure-stack-run-powershell-script.md) | Utilisez ce module pour préparer l’ordinateur hôte Azure Stack au déploiement et au redéploiement à l’aide de l’image de disque dur virtuel (VHD) Azure Stack. | Les opérateurs de cloud|
| [Connexion à Azure Stack](azure-stack-connect-powershell.md) | Ce module permet de configurer la connectivité VPN avec Azure Stack. | Les opérateurs et les utilisateurs de cloud |
| [Validateur de modèle](user/azure-stack-validate-templates.md) | Utilisez ce module pour vérifier si un modèle existant ou nouveau peut être déployé sur Azure Stack. | Les opérateurs et les utilisateurs de cloud|


## <a name="next-steps"></a>Étapes suivantes
* [Configurez l’environnement PowerShell de l’utilisateur Azure Stack.](user/azure-stack-powershell-configure-user.md)   
* [Se connecter au Kit de développement Azure Stack par le biais d’un réseau privé virtuel](azure-stack-connect-azure-stack.md)  
