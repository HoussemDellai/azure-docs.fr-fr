---
title: Créer une image personnalisée Azure DevTest Labs à partir d’un fichier de disque dur virtuel | Microsoft Docs
description: Apprenez à créer une image personnalisée dans Azure DevTest Labs à partir d’un fichier de disque dur virtuel à l’aide du portail Azure
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: b795bc61-7c28-40e6-82fc-96d629ee0568
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: a961565815ca0d89dc98a8d6a3e14b338b649398
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/11/2018
ms.locfileid: "38687806"
---
# <a name="create-a-custom-image-from-a-vhd-file"></a>Créer une image personnalisée à partir d’un fichier de disque dur virtuel

[!INCLUDE [devtest-lab-create-custom-image-from-vhd-selector](../../includes/devtest-lab-create-custom-image-from-vhd-selector.md)]

[!INCLUDE [devtest-lab-custom-image-definition](../../includes/devtest-lab-custom-image-definition.md)]

[!INCLUDE [devtest-lab-upload-vhd-options](../../includes/devtest-lab-upload-vhd-options.md)]

## <a name="step-by-step-instructions"></a>Instructions pas à pas

La procédure suivante décrit comment créer une image personnalisée à partir d’un fichier de disque dur virtuel avec le portail Azure :

1. Connectez-vous au [Portail Azure](http://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.

1. Sélectionnez le laboratoire souhaité dans la liste des laboratoires.  

1. Dans le volet principal du lab, sélectionnez **Configuration et stratégies**. 

1. Dans le volet **Configuration et stratégies**, sélectionnez **Images personnalisées**.

1. Dans le volet **Images personnalisées**, sélectionnez **+Ajouter**.

    ![Ajouter une image personnalisée](./media/devtest-lab-create-template/add-custom-image.png)

1. Entrez le nom de l’image personnalisée. Ce nom s’affiche dans la liste des images de base quand vous créez une machine virtuelle.

1. Entrez la description de l’image personnalisée. Cette description s’affiche dans la liste des images de base quand vous créez une machine virtuelle.

1. Pour **Type de système d’exploitation**, sélectionnez **Windows** ou **Linux**.

    - Si vous sélectionnez **Windows**, cochez la case pour indiquer si *sysprep* a été exécuté sur la machine. 
    - Si vous sélectionnez **Linux**, cochez la case pour indiquer si *deprovision* a été exécuté sur la machine. 

1. Sélectionnez un **disque dur virtuel** dans le menu déroulant. Il s’agit du disque dur virtuel qui sera utilisé pour créer la nouvelle image personnalisée. Si nécessaire, sélectionnez pour **Upload a VHD using PowerShell** (Télécharger un disque dur virtuel à l’aide de PowerShell).

1. Vous pouvez également entrer un nom de plan, une offre de plan et un éditeur de plan si l’image utilisée pour créer l’image personnalisée n’est pas une image sous licence (publiée par Microsoft).

   - **Nom du plan :** entrez le nom de l’image de la Place de marché (SKU) à partir de laquelle cette image personnalisée est créée. 
   - **Offre du plan :** entrez le produit (offre) de l’image de la Place de marché à partir de laquelle cette image personnalisée est créée. 
   - **Éditeur du plan :** entrez l’éditeur de l’image de la Place de marché à partir de laquelle cette image personnalisée est créée.

   > [!NOTE]
   > Si l’image que vous utilisez pour créer une image personnalisée n’est **pas** une image sous licence, ces champs sont vides et peuvent être renseignés si vous le choisissez. Si l’image **est** une image sous licence, les champs sont renseignés automatiquement avec les informations du plan. Si vous essayez de les modifier, un message d’avertissement s’affiche.
   >
   >

1. Cliquez sur **OK** pour créer l’image personnalisée.

Après quelques minutes, l’image personnalisée est créée et stockée dans le compte de stockage du laboratoire. Lorsqu’un utilisateur du laboratoire souhaite créer une nouvelle machine virtuelle, l’image est disponible dans la liste des images de base.

![Image personnalisée disponible dans la liste des images de base](./media/devtest-lab-create-template/custom-image-available-as-base.png)


[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="related-blog-posts"></a>Billets de blog connexes

- [Custom images or formulas? (Images personnalisées ou formules ?)](https://blogs.msdn.microsoft.com/devtestlab/2016/04/06/custom-images-or-formulas/)
- [Copying Custom Images between Azure DevTest Labs (Copie d’images personnalisées entre plusieurs Azure DevTest Labs)](http://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs#copying-custom-images-between-azure-devtest-labs)

## <a name="next-steps"></a>Étapes suivantes

- [Ajout d’une machine virtuelle à votre laboratoire](./devtest-lab-add-vm.md)
