---
title: Concepts techniques et modèles Durable Functions - Azure
description: Fournit des détails sur le fonctionnement de Durable Functions dans Azure pour activer les avantages de l’exécution du code avec état dans le cloud.
services: functions
author: kashimiz
manager: jeconnoc
keywords: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 12/06/2018
ms.author: azfuncdf
ms.openlocfilehash: 6eb08af9cdd19bc83d44d29874f6ac58b41ed8c8
ms.sourcegitcommit: f863ed1ba25ef3ec32bd188c28153044124cacbc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/15/2019
ms.locfileid: "56302034"
---
# <a name="durable-functions-patterns-and-technical-concepts"></a>Concepts techniques et modèles Durable Functions

*Fonctions durables* est une extension d[’Azure Functions](../functions-overview.md) et d[’Azure WebJobs](../../app-service/web-sites-create-web-jobs.md) qui vous permet d’écrire des fonctions avec état dans un environnement sans serveur. L’extension gère l’état, les points de contrôle et les redémarrages à votre place. Cet article fournit des informations plus détaillées sur les comportements de l’extension Durable Functions pour Azure Functions, et les modèles d’implémentation courants.

> [!NOTE]
> Fonctions durables est une extension avancée Azure Functions et ne convient pas à toutes les applications. Le reste de cet article suppose que vous maîtrisez parfaitement les concepts [Azure Functions](../functions-overview.md) et les défis qu’impose le développement d’applications sans serveur.

## <a name="patterns"></a>Modèles

Cette section décrit certains modèles d’application standard qui peuvent tirer parti de Durable Functions.

### <a name="chaining"></a>Modèle 1 : Chaînage de fonctions

*Chaînage de fonctions* fait référence au modèle d’exécution d’une séquence de fonctions dans un ordre particulier. La sortie d’une fonction doit souvent être appliquée à l’entrée d’une autre fonction.

![Diagramme de chaînage de fonctions](./media/durable-functions-concepts/function-chaining.png)

Fonctions durables vous permet d’implémenter ce modèle de manière concise dans le code.

#### <a name="c-script"></a>Script C#

```cs
public static async Task<object> Run(DurableOrchestrationContext context)
{
    try
    {
        var x = await context.CallActivityAsync<object>("F1");
        var y = await context.CallActivityAsync<object>("F2", x);
        var z = await context.CallActivityAsync<object>("F3", y);
        return  await context.CallActivityAsync<object>("F4", z);
    }
    catch (Exception)
    {
        // error handling/compensation goes here
    }
}
```

> [!NOTE]
> Il existe des différences subtiles d’écriture d’une fonction durable précompilée en C# par rapport à l’exemple de script C# présenté précédemment. Une fonction précompilée C# requiert que les paramètres durables soient décorés avec leurs attributs respectifs. Un exemple est l’attribut `[OrchestrationTrigger]` pour le paramètre `DurableOrchestrationContext`. Si les paramètres ne sont pas correctement décorés, le runtime ne peut pas injecter les variables dans la fonction et génère une erreur. Pour plus d’exemples, visitez l’[exemple](https://github.com/Azure/azure-functions-durable-extension/blob/master/samples).

#### <a name="javascript-functions-2x-only"></a>JavaScript (Functions 2.x uniquement)

```js
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const x = yield context.df.callActivity("F1");
    const y = yield context.df.callActivity("F2", x);
    const z = yield context.df.callActivity("F3", y);
    return yield context.df.callActivity("F4", z);
});
```

Les valeurs « F1 », « F2 », « F3 » et « F4 » représentent les noms des autres fonctions dans l’application de la fonction. Le flux de contrôle est implémenté à l’aide de constructions de code impératives normales. Autrement dit, le code s’exécute de haut en bas et peut impliquer une sémantique de flux contrôle de langage existante, notamment des instructions conditionnelles et des boucles.  Une logique de gestion des erreurs peut être incluse dans les blocs try/catch/finally.

Le paramètre `context` [DurableOrchestrationContext] \(.NET\) et l’objet `context.df` (JavaScript) fournissent des méthodes permettant d’appeler d’autres fonctions par nom, de passer des paramètres et de retourner la sortie d’une fonction. Chaque fois que le code appelle `await` (C#) ou `yield` (JavaScript), le framework Durable Functions *crée des points de contrôle* de la progression de l’instance de la fonction actuelle. En cas de recyclage du processus ou de la machine virtuelle au milieu de l’exécution, l’instance de la fonction reprend à partir de l’appel à `await` ou `yield` précédent. De plus amples informations sur ce comportement de redémarrage seront présentées ultérieurement.

> [!NOTE]
> L’objet `context` dans JavaScript représente le [contexte de fonction](../functions-reference-node.md#context-object) dans son ensemble, et non le type [DurableOrchestrationContext].

### <a name="fan-in-out"></a>Modèle 2 : Fan-out/fan-in

*Fan-out/fan-in* fait référence à un modèle qui exécute plusieurs fonctions en parallèle puis attend que toutes ces fonctions se terminent.  Un travail d’agrégation est souvent effectué sur les résultats retournés par les fonctions.

![Diagramme Fan-out/fan-in](./media/durable-functions-concepts/fan-out-fan-in.png)

Avec des fonctions normales, le processus fan-out peut être effectué en configurant la fonction afin qu’elle envoie plusieurs messages vers une file d’attente. Mais le processus fan-in est beaucoup plus difficile. Vous devez écrire du code pour effectuer le suivi lorsque les fonctions déclenchées en file d’attente se terminent, puis stocker les sorties des fonctions. L’extension Fonctions durables gère ce modèle avec un code relativement simple.

#### <a name="c-script"></a>Script C#

```cs
public static async Task Run(DurableOrchestrationContext context)
{
    var parallelTasks = new List<Task<int>>();

    // get a list of N work items to process in parallel
    object[] workBatch = await context.CallActivityAsync<object[]>("F1");
    for (int i = 0; i < workBatch.Length; i++)
    {
        Task<int> task = context.CallActivityAsync<int>("F2", workBatch[i]);
        parallelTasks.Add(task);
    }

    await Task.WhenAll(parallelTasks);

    // aggregate all N outputs and send result to F3
    int sum = parallelTasks.Sum(t => t.Result);
    await context.CallActivityAsync("F3", sum);
}
```

#### <a name="javascript-functions-2x-only"></a>JavaScript (Functions 2.x uniquement)

```js
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const parallelTasks = [];

    // get a list of N work items to process in parallel
    const workBatch = yield context.df.callActivity("F1");
    for (let i = 0; i < workBatch.length; i++) {
        parallelTasks.push(context.df.callActivity("F2", workBatch[i]));
    }

    yield context.df.Task.all(parallelTasks);

    // aggregate all N outputs and send result to F3
    const sum = parallelTasks.reduce((prev, curr) => prev + curr, 0);
    yield context.df.callActivity("F3", sum);
});
```

Le processus fan-out est réparti sur plusieurs instances de la fonction `F2`, puis suivi à l’aide d’une liste dynamique de tâches. L’API `Task.WhenAll` .NET ou l’API `context.df.Task.all` JavaScript est appelée pour attendre la fin de toutes les fonctions appelées. Les sorties de la fonction `F2` sont ensuite agrégées à partir de la liste de tâches dynamique puis transmises à la fonction `F3`.

La création automatique de points de contrôle qui se produit lors de l’appel à `await` ou `yield` sur `Task.WhenAll` ou `context.df.Task.all` garantit qu’un incident ou qu’un redémarrage survenu au milieu du processus ne nécessite aucun redémarrage d’une quelconque tâche déjà terminée.

### <a name="async-http"></a>Modèle 3 : API HTTP Async

Le troisième modèle concerne le problème de coordination de l’état des opérations de longue durée avec des clients externes. Une méthode courante pour implémenter ce modèle consiste à déclencher l’action de longue durée par un appel HTTP, puis à rediriger le client vers un point de terminaison d’état interrogeable pour savoir quand l’opération se termine.

![Diagramme de l’API HTTP](./media/durable-functions-concepts/async-http-api.png)

Fonctions durables fournit des API intégrées qui simplifient le code que vous écrivez pour interagir avec les exécutions de fonctions de longue durée. Les exemples de démarrage rapide ([C#](durable-functions-create-first-csharp.md), [JavaScript](quickstart-js-vscode.md)) montrent une commande REST simple permettant de démarrer de nouvelles instances de fonctions orchestrator. Lorsqu’une instance est démarrée, l’extension expose des API HTTP webhook qui interrogent l’état de la fonction d’orchestrateur. L’exemple suivant montre les commandes REST permettant de démarrer un orchestrateur et d’interroger son état. Pour plus de clarté, certains détails ont été retirés de l’exemple.

```
> curl -X POST https://myfunc.azurewebsites.net/orchestrators/DoWork -H "Content-Length: 0" -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/admin/extensions/DurableTaskExtension/b79baf67f717453ca9e86c5da21e03ec

{"id":"b79baf67f717453ca9e86c5da21e03ec", ...}

> curl https://myfunc.azurewebsites.net/admin/extensions/DurableTaskExtension/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/admin/extensions/DurableTaskExtension/b79baf67f717453ca9e86c5da21e03ec

{"runtimeStatus":"Running","lastUpdatedTime":"2017-03-16T21:20:47Z", ...}

> curl https://myfunc.azurewebsites.net/admin/extensions/DurableTaskExtension/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 200 OK
Content-Length: 175
Content-Type: application/json

{"runtimeStatus":"Completed","lastUpdatedTime":"2017-03-16T21:20:57Z", ...}
```

Étant donné que l’état est géré par le runtime de Fonctions durables, vous n’avez pas à implémenter votre propre mécanisme de suivi de l’état.

Même si l’extension Fonctions durables intègre des webhooks pour la gestion des orchestrations de longue durée, vous pouvez implémenter ce modèle vous-même à l’aide de vos propres déclencheurs de fonction (par exemple HTTP, une file d’attente ou Event Hub) et la liaison `orchestrationClient`. Ainsi, vous pouvez utiliser un message de file d’attente pour déclencher l’arrêt.  Sinon, servez-vous d’un déclencheur HTTP protégé par une stratégie d’authentification Azure Active Directory à la place des webhooks intégrés qui utilisent une clé générée pour l’authentification.

#### <a name="c"></a>C#

```cs
// HTTP-triggered function to start a new orchestrator function instance.
public static async Task<HttpResponseMessage> Run(
    HttpRequestMessage req,
    DurableOrchestrationClient starter,
    string functionName,
    ILogger log)
{
    // Function name comes from the request URL.
    // Function input comes from the request content.
    dynamic eventData = await req.Content.ReadAsAsync<object>();
    string instanceId = await starter.StartNewAsync(functionName, eventData);

    log.LogInformation($"Started orchestration with ID = '{instanceId}'.");

    return starter.CreateCheckStatusResponse(req, instanceId);
}
```

#### <a name="javascript-functions-2x-only"></a>JavaScript (Functions 2.x uniquement)

```javascript
// HTTP-triggered function to start a new orchestrator function instance.
const df = require("durable-functions");

module.exports = async function (context, req) {
    const client = df.getClient(context);

    // Function name comes from the request URL.
    // Function input comes from the request content.
    const eventData = req.body;
    const instanceId = await client.startNew(req.params.functionName, undefined, eventData);

    context.log(`Started orchestration with ID = '${instanceId}'.`);

    return client.createCheckStatusResponse(req, instanceId);
};
```

> [!WARNING]
> Quand vous développez localement dans JavaScript, vous devez définir la variable d’environnement `WEBSITE_HOSTNAME` sur `localhost:<port>`, par exemple `localhost:7071` pour utiliser les méthodes sur `DurableOrchestrationClient`. Pour plus d’informations sur cette configuration, consultez le [problème GitHub](https://github.com/Azure/azure-functions-durable-js/issues/28).

Dans .NET, le paramètre [DurableOrchestrationClient](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html) `starter` est une valeur provenant de la liaison de sortie `orchestrationClient`, qui fait partie de l’extension Durable Functions. En JavaScript, cet objet est retourné en appelant `df.getClient(context)`. Ces objets fournissent des méthodes pour démarrer, envoyer des événements, terminer et rechercher les instances de fonctions orchestrator nouvelles ou existantes.

Dans l’exemple précédent, une fonction déclenchée par HTTP utilise une valeur `functionName` provenant de l’URL entrante et transmet cette valeur à [StartNewAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_StartNewAsync_). L’API de liaison [CreateCheckStatusResponse](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_CreateCheckStatusResponse_System_Net_Http_HttpRequestMessage_System_String_) retourne ensuite une réponse qui contient un en-tête `Location` et des informations supplémentaires sur l’instance, qui peuvent être utilisées ultérieurement pour rechercher l’état de l’instance démarrée ou y mettre fin.

### <a name="monitoring"></a>Modèle 4 : Surveillance

Le modèle de surveillance fait référence à un processus *récurrent* flexible dans un flux de travail, par exemple l’interrogation jusqu’à ce que certaines conditions soient respectées. Un [déclencheur de minuteur](../functions-bindings-timer.md) standard peut convenir à un scénario simple, comme une tâche de nettoyage périodique, mais son intervalle est statique et la gestion de la durée de vie des instances devient complexe. L’extension Fonctions durables permet d’avoir des intervalles de récurrence flexibles, de gérer la durée de vie des tâches et de créer plusieurs processus de surveillance à partir d’une seule orchestration.

L’inversion du scénario d’API HTTP asynchrone en est un exemple. Au lieu d’exposer un point de terminaison d’un client externe pour surveiller une opération longue, l’analyse de longue durée consomme un point de terminaison externe, attendant un changement d’état.

![Diagramme de moniteurs](./media/durable-functions-concepts/monitor.png)

Grâce aux fonctions durables, plusieurs moniteurs qui observent des points de terminaison arbitraires peuvent être créés en quelques lignes de code. L’exécution des moniteurs peut se terminer quand une condition est respectée, ou être terminée par [DurableOrchestrationClient](durable-functions-instance-management.md), et leur délai d’attente peut être changé en fonction de certaines conditions (par exemple, une interruption exponentielle). Le code suivant implémente un moniteur de base.

#### <a name="c-script"></a>Script C#

```cs
public static async Task Run(DurableOrchestrationContext context)
{
    int jobId = context.GetInput<int>();
    int pollingInterval = GetPollingInterval();
    DateTime expiryTime = GetExpiryTime();

    while (context.CurrentUtcDateTime < expiryTime)
    {
        var jobStatus = await context.CallActivityAsync<string>("GetJobStatus", jobId);
        if (jobStatus == "Completed")
        {
            // Perform action when condition met
            await context.CallActivityAsync("SendAlert", machineId);
            break;
        }

        // Orchestration will sleep until this time
        var nextCheck = context.CurrentUtcDateTime.AddSeconds(pollingInterval);
        await context.CreateTimer(nextCheck, CancellationToken.None);
    }

    // Perform further work here, or let the orchestration end
}
```

#### <a name="javascript-functions-2x-only"></a>JavaScript (Functions 2.x uniquement)

```js
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    const jobId = context.df.getInput();
    const pollingInternal = getPollingInterval();
    const expiryTime = getExpiryTime();

    while (moment.utc(context.df.currentUtcDateTime).isBefore(expiryTime)) {
        const jobStatus = yield context.df.callActivity("GetJobStatus", jobId);
        if (jobStatus === "Completed") {
            // Perform action when condition met
            yield context.df.callActivity("SendAlert", machineId);
            break;
        }

        // Orchestration will sleep until this time
        const nextCheck = moment.utc(context.df.currentUtcDateTime).add(pollingInterval, 's');
        yield context.df.createTimer(nextCheck.toDate());
    }

    // Perform further work here, or let the orchestration end
});
```

Quand une requête est reçue, une nouvelle instance d’orchestration est créée pour cet ID de tâche. L’instance interroge un état jusqu’à ce qu’une condition soit respectée et que vous quittiez la boucle. Un minuteur durable est utilisé pour contrôler la fréquence d’interrogation. Des opérations supplémentaires peuvent ensuite être exécutées, ou l’orchestration peut prendre fin. Quand la valeur `context.CurrentUtcDateTime` (.NET) ou `context.df.currentUtcDateTime` (JavaScript) dépasse la valeur `expiryTime`, le moniteur se termine.

### <a name="human"></a>Modèle 5 : Interaction humaine

De nombreux processus impliquent un certain type d’interaction humaine. La difficulté d’impliquer des personnes dans un processus automatisé réside dans le fait que ces personnes ne sont pas toujours aussi disponibles et réactives que les services cloud. Des processus automatisés doivent être mis en place, qui utilisent souvent des délais d’expiration et une logique de compensation.

Un processus d’approbation est un exemple de processus d’entreprise impliquant une interaction humaine. Par exemple, l’approbation d’un manager peut être requise si une note de frais dépasse un certain montant. Si le manager n’approuve pas cette note de frais sous 72 heures (par exemple, s’il est en vacances), un processus d’escalade est déclenché pour obtenir l’approbation d’une autre personne (par exemple, le supérieur hiérarchique de ce manager).

![Diagramme Interaction humaine](./media/durable-functions-concepts/approval.png)

Ce modèle peut être implémenté à l’aide d’une fonction d’orchestrateur. L’orchestrateur utilise un [minuteur durable](durable-functions-timers.md) pour demander l’approbation et la faire remonter en cas de délai d’expiration. Il attend un [événement externe](durable-functions-external-events.md), soit la notification générée par une intervention humaine.

#### <a name="c-script"></a>Script C#

```cs
public static async Task Run(DurableOrchestrationContext context)
{
    await context.CallActivityAsync("RequestApproval");
    using (var timeoutCts = new CancellationTokenSource())
    {
        DateTime dueTime = context.CurrentUtcDateTime.AddHours(72);
        Task durableTimeout = context.CreateTimer(dueTime, timeoutCts.Token);

        Task<bool> approvalEvent = context.WaitForExternalEvent<bool>("ApprovalEvent");
        if (approvalEvent == await Task.WhenAny(approvalEvent, durableTimeout))
        {
            timeoutCts.Cancel();
            await context.CallActivityAsync("ProcessApproval", approvalEvent.Result);
        }
        else
        {
            await context.CallActivityAsync("Escalate");
        }
    }
}
```

#### <a name="javascript-functions-2x-only"></a>JavaScript (Functions 2.x uniquement)

```js
const df = require("durable-functions");
const moment = require('moment');

module.exports = df.orchestrator(function*(context) {
    yield context.df.callActivity("RequestApproval");

    const dueTime = moment.utc(context.df.currentUtcDateTime).add(72, 'h');
    const durableTimeout = context.df.createTimer(dueTime.toDate());

    const approvalEvent = context.df.waitForExternalEvent("ApprovalEvent");
    if (approvalEvent === yield context.df.Task.any([approvalEvent, durableTimeout])) {
        durableTimeout.cancel();
        yield context.df.callActivity("ProcessApproval", approvalEvent.result);
    } else {
        yield context.df.callActivity("Escalate");
    }
});
```

Le minuteur durable est créé en appelant `context.CreateTimer` (.NET) ou `context.df.createTimer`(JavaScript). La notification est reçue par `context.WaitForExternalEvent` (.NET) ou `context.df.waitForExternalEvent` (JavaScript). Et `Task.WhenAny` (.NET) ou `context.df.Task.any` (JavaScript) est appelée pour déterminer s’il faut faire remonter (le délai d’expiration survient en premier) ou traiter l’approbation (approbation reçue avant le délai d’expiration).

Un client externe peut remettre la notification d’événement à une fonction d’orchestrateur en attente au moyen d’[APIS HTTP intégrées](durable-functions-http-api.md#raise-event) ou de l’API [DurableOrchestrationClient.RaiseEventAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_RaiseEventAsync_System_String_System_String_System_Object_) à partir d’une autre fonction :

```csharp
public static async Task Run(string instanceId, DurableOrchestrationClient client)
{
    bool isApproved = true;
    await client.RaiseEventAsync(instanceId, "ApprovalEvent", isApproved);
}
```

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const isApproved = true;
    await client.raiseEvent(instanceId, "ApprovalEvent", isApproved);
};
```

## <a name="the-technology"></a>La technologie

En arrière-plan, l’extension Fonctions durables repose sur le [framework de l’extension Tâche durable](https://github.com/Azure/durabletask), une bibliothèque open source sur GitHub pour la génération d’orchestrations de tâches durables. Tout comme Azure Functions est l’évolution sans serveur d’Azure Webjobs, Fonctions durables est l’évolution sans serveur de l’infrastructure des tâches durables. L’infrastructure des tâches durables est très utilisée au sein de Microsoft et à l’extérieur pour automatiser des processus critiques. Il convient parfaitement à l’environnement Azure Functions sans serveur.

### <a name="event-sourcing-checkpointing-and-replay"></a>Approvisionnement d’événements, création de points de contrôle et réexécution

Les fonctions d’orchestrateur conservent de façon fiable leur état d’exécution à l’aide d’un modèle de conception appelé [approvisionnement d’événements](https://docs.microsoft.com/azure/architecture/patterns/event-sourcing). Au lieu de stocker directement l’état *actuel* d’une orchestration, l’extension durable utilise un magasin d’ajout uniquement pour enregistrer *toute la série d’actions* exécutées par l’orchestration de la fonction. Cette méthode offre de nombreux avantages, notamment une amélioration des performances, de l’évolutivité et de la réactivité par rapport au « vidage » de l’état d’exécution complet. Autres avantages : elle garantit la cohérence des données transactionnelles et fournit des pistes d’audit complètes ainsi qu’un historique. Les pistes d’audit permettent des actions de compensation fiables.

L’utilisation de l’approvisionnement d’événements par cette extension est transparente. En coulisse, l’opérateur `await` (C#) ou `yield` (JavaScript) d’une fonction orchestrator cède le contrôle du thread orchestrateur au répartiteur Durable Task Framework. Le répartiteur valide ensuite dans le stockage toutes les actions que la fonction d’orchestrateur a planifiées (par exemple, l’appel d’une ou plusieurs fonctions enfant ou la planification d’un minuteur durable). Cette action de validation transparente s’ajoute à *l’historique d’exécution* de l’instance d’orchestration. L’historique est stocké dans une table de stockage. L’action de validation ajoute ensuite des messages à une file d’attente pour planifier le travail réel. À ce stade, la fonction d’orchestrateur peut être déchargée de la mémoire. Sa facturation s’arrête si vous utilisez le plan de consommation Azure Functions.  Si d’autres tâches doivent être effectuées, la fonction redémarre et son état est reconstruit.

Lorsqu’une fonction d’orchestration reçoit plus de tâches à effectuer (par exemple, un message de réponse est reçu ou un minuteur durable expire), l’orchestrateur sort à nouveau de veille et réexécute toute la fonction depuis le début afin de reconstruire l’état local. Si au cours de la réexécution, le code tente d’appeler une fonction (ou toute autre tâche asynchrone), l’infrastructure des tâches durables consulte *l’historique d’exécution* de l’orchestration en cours. Si elle constate que la [fonction d’activité](durable-functions-types-features-overview.md#activity-functions) a déjà été exécutée et a produit un résultat, elle réexécute les résultats de cette fonction, et le code d’orchestrateur continue de s’exécuter. Ce processus se poursuit jusqu'à ce que le code de la fonction atteint un point où il se termine ou s’il a planifié une nouvelle tâche asynchrone.

### <a name="orchestrator-code-constraints"></a>Contraintes du code d’orchestrateur

Le comportement de réexécution crée des contraintes concernant le type de code qui peut être écrit dans une fonction d’orchestrateur. Par exemple, le code d’orchestrateur doit être déterministe car il sera réexécuté à plusieurs reprises et doit générer le même résultat à chaque fois. Vous trouverez la liste complète des contraintes dans la section, [Contraintes du code d’orchestrateur](durable-functions-checkpointing-and-replay.md#orchestrator-code-constraints) de l’article **Création de points de contrôle et redémarrage**.

## <a name="monitoring-and-diagnostics"></a>Surveillance et diagnostics

L’extension Fonctions durables transmet automatiquement des données de suivi structurées à [Application Insights](../functions-monitoring.md) quand l’application de fonction est configurée avec une clé d’instrumentation Application Insights. Ces données de suivi peuvent être utilisées pour surveiller le comportement et la progression de vos orchestrations.

Voici un exemple montrant à quoi ressemblent les événements de suivi Fonctions durables dans le portail Application Insights en utilisant [Application Insights Analytics](../../application-insights/app-insights-analytics.md) :

![Résultats d’une requête App Insights](./media/durable-functions-concepts/app-insights-1.png)

Le champ `customDimensions` de chaque entrée de journal contient de nombreuses données structurées. Voici un exemple d’une telle entrée, entièrement développée.

![Champ customDimensions dans la requête App Insights](./media/durable-functions-concepts/app-insights-2.png)

En raison du comportement de réexécution du répartiteur de l’infrastructure des tâches durables, attendez-vous à voir des entrées de journal redondantes pour les actions réexécutées. Il peut être utile de comprendre le comportement de réexécution du moteur central. L’article [Diagnostics](durable-functions-diagnostics.md) montre des exemples de requêtes qui filtrent les journaux de réexécution pour n’afficher que les journaux « en temps réel ».

## <a name="storage-and-scalability"></a>Stockage et évolutivité

L’extension Fonctions durables utilise des files d’attente Stockage Azure, des tables et des objets blob pour conserver l’état de l’historique d’exécution et déclencher l’exécution d’une fonction. Vous pouvez utiliser le compte de stockage par défaut pour l’application de la fonction ou configurer un compte de stockage distinct. Vous pouvez choisir un compte distinct en raison de limites de débit de stockage. Le code d’orchestrateur que vous écrivez ne doit pas interagir avec les entités de ces comptes de stockage. Les entités sont gérées directement par l’infrastructure des tâches durables en tant que détail d’implémentation.

Les fonctions d’orchestrateur planifient les fonctions d’activité et reçoivent leurs réponses via des messages internes en file d’attente. Lorsque l’application d’une fonction s’exécute dans le plan de consommation Azure Functions, ces files d’attente sont surveillées par le [contrôleur de mise à l’échelle Azure Functions](../functions-scale.md#how-the-consumption-plan-works) et de nouvelles instances de calcul sont ajoutées si nécessaire. En cas de montée en charge sur plusieurs machines virtuelles, une fonction d’orchestrateur peut s’exécuter sur une machine virtuelle pendant que les fonctions d’activité qu’elle appelle s’exécutent sur plusieurs machines virtuelles différentes. Vous trouverez plus d’informations sur le comportement de mise à l’échelle de Fonctions durables dans l’article [Performances et mise à l’échelle](durable-functions-perf-and-scale.md).

Un stockage de table est utilisé pour stocker l’historique d’exécution pour les comptes d’orchestrateur. Chaque fois qu’une instance est réalimentée sur une machine virtuelle particulière, elle extrait son historique d’exécution du stockage de table pour reconstruire son état local. Disposer de l’historique dans le stockage de table vous permet de consulter l’historique de vos orchestrations à l’aide d’outils comme [Explorateur Stockage Azure Microsoft](../../vs-azure-tools-storage-manage-with-storage-explorer.md).

Les objets blob de stockage sont principalement utilisés comme un mécanisme de location pour coordonner la montée en puissance des instances de l’orchestration sur plusieurs machines virtuelles. Ils sont également utilisés pour contenir les données de messages volumineux qui ne peuvent pas être stockées directement dans des tables ou des files d’attente.

![Capture d’écran Explorateur Stockage Azure](./media/durable-functions-concepts/storage-explorer.png)

> [!WARNING]
> Même s’il est facile et pratique d’afficher l’historique d’exécution dans le stockage de table, évitez de créer une dépendance sur cette table. Cette situation peut changer à mesure que l’extension Fonctions durables évolue.

## <a name="known-issues-and-faq"></a>Problèmes connus et FAQ

Tous les problèmes connus doivent être répertoriés dans la liste [Problèmes GitHub](https://github.com/Azure/azure-functions-durable-extension/issues). Si vous rencontrez un problème qui n’apparaît pas dans GitHub, soumettez un nouveau problème en incluant une description détaillée.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Durable Functions, voir [Vue d’ensemble des types de fonctions et fonctionnalités pour Durable Functions (Azure Functions)](durable-functions-types-features-overview.md) ou :

> [!div class="nextstepaction"]
> [Créer une première fonction durable](durable-functions-create-first-csharp.md)

[DurableOrchestrationContext]: https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html
