---
title: Exemple - Utiliser un sous-réseau approuvé pour les interfaces réseau de machine virtuelle
description: Cet exemple de définition de stratégie exige que les interfaces réseau utilisent un sous-réseau approuvé.
services: azure-policy
author: DCtheGeek
manager: carmonm
ms.service: azure-policy
ms.topic: sample
ms.date: 01/23/2019
ms.author: dacoulte
ms.openlocfilehash: 854dbb6d3743c0e8af8a267a425286624453b688
ms.sourcegitcommit: fcb674cc4e43ac5e4583e0098d06af7b398bd9a9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/18/2019
ms.locfileid: "56341094"
---
# <a name="sample---use-approved-subnet-for-vm-network-interfaces"></a>Exemple - Utiliser un sous-réseau approuvé pour les interfaces réseau de machine virtuelle

Cette stratégie exige que les interfaces réseau utilisent un sous-réseau approuvé. Spécifiez l’identifiant du sous-réseau approuvé.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-template"></a>Exemple de modèle

[!code-json[main](../../../../policy-templates/samples/Network/vm-creation-in-approved-subnet/azurepolicy.json "Use approved subnet for VM network interfaces")]

Vous pouvez déployer ce modèle en utilisant le [portail Azure](#deploy-with-the-portal), avec [PowerShell](#deploy-with-powershell) ou avec [Azure CLI](#deploy-with-azure-cli).

## <a name="deploy-with-the-portal"></a>Déployer avec le portail

[![Déployer sur Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/?feature.customportal=false&microsoft_azure_policy=true&microsoft_azure_policy_policyinsights=true&feature.microsoft_azure_security_policy=true&microsoft_azure_marketplace_policy=true#blade/Microsoft_Azure_Policy/CreatePolicyDefinitionBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-policy%2Fmaster%2Fsamples%2FNetwork%2Fvm-creation-in-approved-subnet%2Fazurepolicy.json)

## <a name="deploy-with-powershell"></a>Déployer avec PowerShell

[!INCLUDE [sample-powershell-install](../../../../includes/sample-powershell-install-no-ssh-az.md)]

```azurepowershell-interactive
$definition = New-AzPolicyDefinition -Name "vm-creation-in-approved-subnet" -DisplayName "Use approved subnet for VM network interfaces" -description "This policy enforces VM network interfaces to use subnet" -Policy 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/Network/vm-creation-in-approved-subnet/azurepolicy.rules.json' -Parameter 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/Network/vm-creation-in-approved-subnet/azurepolicy.parameters.json' -Mode All
$definition
$assignment = New-AzPolicyAssignment -Name <assignmentname> -Scope <scope>  -subnetId <Subnet Id> -PolicyDefinition $definition
$assignment
```

### <a name="clean-up-powershell-deployment"></a>Nettoyer un déploiement PowerShell

Exécutez la commande suivante pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="deploy-with-azure-cli"></a>Déploiement avec l’interface de ligne de commande Azure

[!INCLUDE [sample-cli-install](../../../../includes/sample-cli-install.md)]

```azurecli-interactive
az policy definition create --name 'vm-creation-in-approved-subnet' --display-name 'Use approved subnet for VM network interfaces' --description 'This policy enforces VM network interfaces to use subnet' --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/Network/vm-creation-in-approved-subnet/azurepolicy.rules.json' --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/Network/vm-creation-in-approved-subnet/azurepolicy.parameters.json' --mode All

az policy assignment create --name <assignmentname> --scope <scope> --policy "vm-creation-in-approved-subnet"
```

### <a name="clean-up-azure-cli-deployment"></a>Nettoyer un déploiement Azure CLI

Exécutez la commande suivante pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées.

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Étapes suivantes

- Consulter d’autres exemples dans [Exemples Azure Policy](index.md)