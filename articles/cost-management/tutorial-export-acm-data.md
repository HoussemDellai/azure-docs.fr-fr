---
title: 'Tutoriel : Créer et gérer des données exportées depuis Azure Cost Management | Microsoft Docs'
description: Cet article vous montre comment vous pouvez créer et gérer des données Azure Cost Management pour les utiliser dans des systèmes externes.
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 02/05/2019
ms.topic: tutorial
ms.service: cost-management
manager: dougeby
ms.custom: seodec18
ms.openlocfilehash: a7c503fba534b72323472fa58b14188bc412003c
ms.sourcegitcommit: 39397603c8534d3d0623ae4efbeca153df8ed791
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/12/2019
ms.locfileid: "56100686"
---
# <a name="tutorial-create-and-manage-exported-data"></a>Tutoriel : Créer et gérer des données exportées

Si vous avez lu le tutoriel Analyse du coût, vous êtes familiarisé avec le téléchargement manuel de vos données Cost Management. Cependant, vous pouvez créer une tâche récurrente qui exporte automatiquement sur une base quotidienne, hebdomadaire ou mensuelle vos données Cost Management dans un stockage Azure. Les données exportées sont au format CSV, et elles contiennent toutes les informations collectées par Cost Management. Vous pouvez ensuite utiliser les données exportées dans Stockage Azure avec des systèmes externes et les combiner avec vos propres données personnalisées. Vous pouvez aussi utiliser vos données exportées dans un système externe, comme un tableau de bord ou un autre système financier.

Les exemples de ce tutoriel montrent comment exporter vos données de gestion des coûts, puis comment vérifier que les données ont été exportées correctement.

Ce tutoriel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une exportation quotidienne
> * Vérifier que les données sont collectées

## <a name="prerequisites"></a>Prérequis
L’exportation des données est disponible pour divers types de comptes Azure, notamment pour les clients [Contrat Entreprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/). Pour voir la liste complète des types de comptes pris en charge, consultez [Comprendre les données Cost Management](understand-cost-mgt-data.md). Les autorisations Azure suivantes sont prises en charge par abonnement chaque pour l’exportation de données par utilisateur et par groupe :

- Propriétaire : peut créer, modifier ou supprimer des exportations planifiées pour un abonnement.
- Contributeur : peut créer, modifier ou supprimer ses propres exportations planifiées. Peut modifier le nom d’exportations planifiées créées par d’autres utilisateurs.
- Lecteur : peut planifier des exportations pour lesquelles il dispose des autorisations adéquates.

Pour les comptes Stockage Azure :
- Des autorisations d’écriture sont requises pour la modification du compte de stockage configuré, quelles que soient les autorisations sur l’exportation.
- Votre compte de stockage Azure doit être configuré pour le stockage d’objets blob ou de fichiers.

## <a name="sign-in-to-azure"></a>Connexion à Azure
Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com/).

## <a name="create-a-daily-export"></a>Créer une exportation quotidienne

Gestion des coûts + Facturation &gt; Gestion des coûts &gt; sélectionnez un abonnement ou un groupe de ressources dans un abonnement &gt; Exporter &gt; **Ajouter**.

Tapez un nom pour l’exportation et sélectionnez l’option « Exportation quotidienne des coûts en cumul mensuel à ce jour ». Cliquez sur **Suivant**.

![Exemple de nouvelle exportation indiquant le type d’exportation](./media/tutorial-export-acm-data/basics_exports.png)

Spécifiez l’abonnement pour votre compte de stockage Azure, puis sélectionnez votre compte de stockage.  Spécifiez le conteneur de stockage et le chemin du répertoire que vous souhaitez utiliser pour le fichier d’exportation.  Cliquez sur **Suivant**.

![Exemple de nouvelle exportation indiquant les détails du compte de stockage](./media/tutorial-export-acm-data/storage_exports.png)

Vérifiez vos informations d’exportation, puis cliquez sur **Créer**.

Votre nouvelle exportation apparaît dans la liste des exportations. Par défaut, les nouvelles exportations sont activées. Si vous voulez désactiver ou supprimer une exportation planifiée, cliquez sur n’importe quel élément de la liste, puis cliquez sur **Désactiver** ou sur **Supprimer**.

Initialement, l’exportation peut s’exécuter au bout d’une ou deux heures. Jusqu’à quatre heures peuvent cependant être nécessaires avant que les données apparaissent dans les fichiers exportés.

### <a name="export-schedule"></a>Planification d’exportation

Les exportations planifiées dépendent de l’heure et du jour de la semaine de la création initiale des exportations. Quand vous créez une exportation planifiée, chacune de ses occurrences suivantes s’exécute à la même heure de la journée. Par exemple, vous créez une exportation quotidienne à 13h00. L’exportation suivante s’exécute à 13h00 le lendemain. L’heure actuelle affecte tous les autres types d’exportation de la même manière. Les exportations s’exécutent toujours à la même heure de la journée que celle à laquelle vous les avez initialement créées. Autre exemple : vous créez une exportation hebdomadaire à 16h00 le lundi. L’exportation suivante s’exécute à 16h00 le lundi suivant. *Les données exportées sont disponibles dans un délai de quatre heures après l’heure d’exécution.*

Chaque exportation crée un fichier, ce qui signifie que les exportations antérieures ne sont pas écrasées.

Il existe trois types d’options d’exportation :

**Exportation quotidienne des coûts en cumul mensuel à ce jour** : l’exportation initiale s’exécute immédiatement. Les exportations suivantes s’exécutent le lendemain à la même heure que l’exportation initiale. Les dernières données sont ajoutées aux exportations quotidiennes précédentes.

**Exportation hebdomadaire des coûts pour les 7 derniers jours** : l’exportation initiale s’exécute immédiatement. Les exportations suivantes s’exécutent à la même heure et le même jour que l’exportation initiale. Les coûts correspondent aux sept derniers jours.

**Personnalisé** : permet de planifier des exportations hebdomadaires et mensuelles avec des options de cumul hebdomadaire ou mensuel à ce jour. *L’exportation initiale s’exécute immédiatement.*

![Nouvelle exportation - Onglet de base montrant une sélection d’exportation en cumul hebdomadaire personnalisée](./media/tutorial-export-acm-data/tutorial-export-schedule-weekly-week-to-date.png)

## <a name="verify-that-data-is-collected"></a>Vérifier que les données sont collectées

Vous pouvez facilement vérifier que vos données Cost Management sont collectées et visualiser le fichier CSV exporté avec l’Explorateur Stockage Azure.

Dans la liste des exportations, cliquez sur le nom du compte de stockage. Dans la page du compte de stockage, cliquez sur Ouvrir dans l’Explorateur. Si vous voyez une boîte de confirmation, cliquez sur **Oui** pour ouvrir le fichier dans l’Explorateur Stockage Azure.

![Page de compte de stockage affichant des exemples d’informations et un lien pour les ouvrir dans l’Explorateur](./media/tutorial-export-acm-data/storage-account-page.png)

Dans l’Explorateur Stockage, accédez au conteneur que vous voulez ouvrir, puis sélectionnez le dossier correspondant au mois en cours. Une liste de fichiers CSV s’affiche. Sélectionnez un fichier, puis cliquez sur **Ouvrir**.

![Exemples d’informations affichées dans l’Explorateur de stockage](./media/tutorial-export-acm-data/storage-explorer.png)

Le fichier s’ouvre avec le programme ou l’application configuré pour ouvrir les fichiers avec l’extension CSV. Voici un exemple dans Excel.

![Exemples de données CSV exportées affichées dans Excel](./media/tutorial-export-acm-data/example-export-data.png)

## <a name="access-exported-data-from-other-systems"></a>Accéder à des données exportées à partir d’autres systèmes

Un des objectifs de l’exportation de vos données Cost Management est d’accéder à ces données à partir de systèmes externes. Vous pouvez utiliser un système de tableau de bord ou un autre système financier. Ces systèmes peuvent grandement varier : montrer un exemple ne serait donc pas pratique.  Vous pouvez cependant découvrir comment accéder à vos données à partir de vos applications dans [Introduction à Stockage Azure](../storage/common/storage-introduction.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à :

> [!div class="checklist"]
> * Créer une exportation quotidienne
> * Vérifier que les données sont collectées

Passez au tutoriel suivant pour optimiser et améliorer l’efficacité en identifiant les ressources inactives et sous-utilisées.

> [!div class="nextstepaction"]
> [Consulter des recommandations d’optimisation et agir en fonction](tutorial-acm-opt-recommendations.md)
