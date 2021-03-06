---
title: Présentation du verrouillage des ressources
description: Découvrez les options de verrouillage permettant de protéger les ressources au moment d’affecter un blueprint.
services: blueprints
author: DCtheGeek
ms.author: dacoulte
ms.date: 01/23/2019
ms.topic: conceptual
ms.service: blueprints
manager: carmonm
ms.custom: seodec18
ms.openlocfilehash: 2e281896d45ada8010f24a1f18265a8cdd523d31
ms.sourcegitcommit: a65b424bdfa019a42f36f1ce7eee9844e493f293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/04/2019
ms.locfileid: "55696982"
---
# <a name="understand-resource-locking-in-azure-blueprints"></a>Comprendre le verrouillage de ressources dans les blueprints Azure

La création d’environnements cohérents à l’échelle n’est vraiment utile que s’il existe un mécanisme pour gérer cette cohérence. Cet article explique le fonctionnement du verrouillage de ressources dans les blueprints Azure.

## <a name="locking-modes-and-states"></a>Modes et états de verrouillage

Le mode de verrouillage s’applique à l’attribution de blueprint et offre trois options : **Ne pas verrouiller**, **Lecture seule** ou **Ne pas supprimer**. Le mode de verrouillage est configuré durant le déploiement d’artefact au cours d’une attribution de blueprint. Un autre mode de verrouillage peut être défini en mettant à jour l’attribution de blueprint.
Les modes de verrouillage ne peuvent cependant pas être modifiés en dehors des blueprints.

Les ressources créées par des artefacts dans une attribution de blueprint ont quatre états possibles : **Non verrouillé**, **Lecture seule**, **Modification/suppression impossible** ou **Suppression impossible**. Chaque type d’artefact peut être en état **Non verrouillé**. Le tableau suivant permet de déterminer l’état d’une ressource :

|Mode|Type de ressource d’artefact|État|Description|
|-|-|-|-|
|Ne pas verrouiller|*|Non verrouillé|Les ressources ne sont pas protégées par des blueprints. Cet état est également utilisé pour des ressources ajoutées à un artéfact de groupe de ressources **En lecture seule** ou **Ne pas supprimer** à partir de l’extérieur d’une attribution de blueprint.|
|Lecture seule|Groupe de ressources|Modification/suppression impossible|Le groupe de ressources est en lecture seule et les balises sur le groupe de ressources ne peuvent pas être modifiées. Des ressources **Non verrouillées** peuvent être ajoutées, déplacées, modifiées ou supprimées dans ce groupe de ressources.|
|Lecture seule|Groupe de non-ressources|Lecture seule|La ressource ne peut pas être modifiée de quelque manière que ce soit (aucune modification possible et suppression impossible).|
|Ne pas supprimer|*|Suppression impossible|Les ressources peuvent être modifiées mais pas supprimées. Des ressources **Non verrouillées** peuvent être ajoutées, déplacées, modifiées ou supprimées dans ce groupe de ressources.|

## <a name="overriding-locking-states"></a>Neutralisation des états de verrouillage

Il est généralement possible pour une personne disposant d’un [contrôle d’accès en fonction du rôle](../../../role-based-access-control/overview.md) (RBAC) approprié sur l’abonnement, tel que le rôle « Propriétaire », d’être autorisée à modifier ou supprimer n’importe quelle ressource. Cet accès n’est pas possible quand les blueprints appliquent un verrouillage dans le cadre d’une affectation déployée. Si l’attribution a été définie avec l’option **Lecture seule** ou **Ne pas supprimer**, même le propriétaire de l’abonnement ne peut pas effectuer l’action bloquée sur la ressource protégée.

Cette mesure de sécurité assure la cohérence du blueprint défini et protège l’environnement pour la création duquel il a été conçu contre toute modification ou suppression accidentelle ou programmatique.

## <a name="removing-locking-states"></a>Suppression des états de verrouillage

S’il devient nécessaire de modifier ou supprimer une ressource protégée par une attribution, il existe deux manières de procéder.

- En mettant à jour de l’attribution de blueprint en la définissant sur un mode de verrouillage **Ne pas verrouiller**
- Supprimer l’attribution de blueprint

Une fois l’affectation supprimée, les verrous créés par les blueprints sont supprimés. Cependant, la ressource est conservée et doit être supprimée selon des procédés normaux.

## <a name="how-blueprint-locks-work"></a>Fonctionnement des verrous de blueprint

Une action de refus de type [Refuser les attributions](../../../role-based-access-control/deny-assignments.md) de contrôle d’accès en fonction du rôle (RBAC) est appliquée aux ressources d’artefact lors de l’attribution d’un blueprint si cette attribution a sélectionné l’option **Lecture seule** ou **Ne pas supprimer**. L’action de refus est ajoutée par l’identité managée de l’attribution de blueprint, et ne peut être supprimée des ressources d’artefacts que par cette même identité managée. Cette mesure de sécurité a pour effet d’appliquer le mécanisme de verrouillage et d’empêcher la suppression du verrou du blueprint en dehors de blueprints.

> [!IMPORTANT]
> Azure Resource Manager met en cache les détails des affectations de rôles pendant 30 minutes au maximum. Par conséquent, une action de refus de type Refuser les attributions sur des ressources de blueprint risque de ne pas être immédiatement effective. Pendant cette période de temps, il peut être possible de supprimer une ressource destinée à être protégée par des verrous de blueprint.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur le [cycle de vie des blueprints](lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](parameters.md)
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](sequencing-order.md)
- Découvrir comment [mettre à jour des affectations existantes](../how-to/update-existing-assignments.md)
- Résoudre les problèmes durant l’affectation d’un blueprint en suivant les étapes de [dépannage général](../troubleshoot/general.md)