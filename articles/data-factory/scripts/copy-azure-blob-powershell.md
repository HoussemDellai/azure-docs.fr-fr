---
title: 'Script PowerShell : Copier des données dans le cloud à l’aide d’Azure Data Factory | Microsoft Docs'
description: Ce script PowerShell copie des données d’un emplacement dans un Stockage Blob Azure vers un autre dans le même Stockage Blob Azure.
services: data-factory
author: linda33wj
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 09/12/2017
ms.author: jingwang
ms.openlocfilehash: e36887e755d772a408fd31d05ed10a9a7c678bd9
ms.sourcegitcommit: 25936232821e1e5a88843136044eb71e28911928
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54021507"
---
# <a name="use-powershell-to-create-a-data-factory-pipeline-to-copy-data-in-the-cloud"></a>Utiliser PowerShell pour créer un pipeline de fabrique de données afin de copier des données dans le cloud

Cet exemple de script PowerShell crée dans Azure Data Factory un pipeline qui copie les données d’un emplacement vers un autre emplacement dans un stockage Blob Azure.

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="prerequisites"></a>Prérequis
* **Compte Stockage Azure**. Vous utilisez le stockage Blob à la fois comme magasins de données **source** et **récepteur**. Si vous ne possédez pas de compte de stockage Azure, consultez [Créer un compte de stockage](../../storage/common/storage-quickstart-create-account.md) pour en créer un. 
* Créez un **conteneur d’objets blob** dans le stockage Blob, créez un **dossier** d’entrée dans le conteneur et chargez des fichiers sur le dossier. Vous pouvez utiliser des outils tels que l’[Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) pour vous connecter au stockage Blob Azure, créer un conteneur d’objets blob, charger le fichier d’entrée et vérifier le fichier de sortie.

## <a name="sample-script"></a>Exemple de script

> [!IMPORTANT]
> Ce script crée des fichiers JSON qui définissent des entités Data Factory (service lié, jeu de données et pipeline) sur votre disque dur dans le dossier c:\.

[!code-powershell[main](../../../powershell_scripts/data-factory/copy-from-azure-blob-to-blob/copy-from-azure-blob-to-blob.ps1 "Copy from Blob Storage -> Blob Storage")]


## <a name="clean-up-deployment"></a>Nettoyer le déploiement

Après avoir exécuté l’exemple de script, vous pouvez utiliser la commande suivante pour supprimer le groupe de ressources et toutes les ressources associées :

```powershell
Remove-AzureRmResourceGroup -ResourceGroupName $resourceGroupName
```
Pour supprimer la fabrique de données du groupe de ressources, exécutez la commande suivante : 

```powershell
Remove-AzureRmDataFactoryV2 -Name $dataFactoryName -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes : 

| Commande | Notes |
|---|---|
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [Set-AzureRmDataFactoryV2](/powershell/module/azurerm.datafactoryv2/set-azurermdatafactoryv2) | Créer une fabrique de données. |
| [Set-AzureRmDataFactoryV2LinkedService](/powershell/module/azurerm.datafactoryv2/Set-azurermdatafactoryv2linkedservice) | Crée un service lié dans la fabrique de données. Un service lié rattache une banque de données ou une ressource de calcul à une fabrique de données. |
| [Set-AzureRmDataFactoryV2Dataset](/powershell/module/azurerm.datafactoryv2/Set-azurermdatafactoryv2dataset) | Crée un jeu de données dans la fabrique de données. Un jeu de données représente les entrées/sorties pour une activité dans un pipeline. | 
| [Set-AzureRmDataFactoryV2Pipeline](/powershell/module/azurerm.datafactoryv2/Set-azurermdatafactoryv2pipeline) | Crée un pipeline dans la fabrique de données. Un pipeline contient une ou plusieurs activités effectuant une opération donnée. Dans ce pipeline, l’activité de copie a pour effet de copier les données d’un emplacement vers un autre dans le Stockage Blob Azure. |
| [Invoke-AzureRmDataFactoryV2Pipeline](/powershell/module/azurerm.datafactoryv2/Invoke-azurermdatafactoryv2pipeline) | Crée une exécution pour le pipeline. En d’autres termes, exécute le pipeline. |
| [Get-AzureRmDataFactoryV2ActivityRun](/powershell/module/azurerm.datafactoryv2/get-azurermdatafactoryv2activityrun) | Obtient les détails de l’exécution de l’activité dans le pipeline. 
| [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |
|||

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure PowerShell, consultez la [documentation Azure PowerShell](https://docs.microsoft.com/powershell/).

Des exemples supplémentaires de scripts PowerShell pour Azure Data Factory sont à votre disposition dans [Exemples PowerShell pour Azure Data Factory](../samples-powershell.md).
