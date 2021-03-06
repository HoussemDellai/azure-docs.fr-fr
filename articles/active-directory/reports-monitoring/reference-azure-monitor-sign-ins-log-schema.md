---
title: Schéma des journaux de connexion Azure Active Directory dans Azure Monitor (préversion) | Microsoft Docs
description: Décrire le schéma de journal de connexion Azure AD pour une utilisation dans Azure Monitor (préversion)
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: daveba
editor: ''
ms.assetid: 4b18127b-d1d0-4bdc-8f9c-6a4c991c5f75
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: priyamo
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0834333dbec9f8aa23092339ea41d0b6cc5aba08
ms.sourcegitcommit: 301128ea7d883d432720c64238b0d28ebe9aed59
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/13/2019
ms.locfileid: "56169715"
---
# <a name="interpret-the-azure-ad-sign-in-logs-schema-in-azure-monitor-preview"></a>Interpréter le schéma des journaux de connexion Azure Active Directory dans Azure Monitor (préversion)

Cet article décrit le schéma de journal de connexion Azure Active Directory (Azure AD) dans Azure Monitor. La plupart des informations liées aux connexions sont fournies sous l’attribut *Propriétés* de l’objet `records`.

```json
{ 
    "records": [ 
    { 
        "time": "2018-05-16T16:09:58.4634578Z", 
        "resourceId": null, 
        "operationName": "Sign-in activity", 
        "operationVersion": "1.0", 
        "category": "SignIn", 
        "tenantId": "bf85dc9d-cb43-44a4-80c4-469e8c58249e", 
        "resultType": "50140", 
        "resultSignature": "None", 
        "resultDescription": "Other", 
        "durationMs": 0, 
        "callerIpAddress": "167.220.0.158", 
        "correlationId": "13e19598-e040-487f-bd32-d38a2cd75d9a", 
        "identity": "arvind harinder", 
        "Level": 4, 
        "location": "US", 
        "properties": { 
            "id": "0782c515-08b6-4029-a65c-29d9a3d20800", 
            "createdDateTime": "2018-05-16T16:09:58.4634578+00:00", 
            "userDisplayName": "Arvind Harinder", 
            "userPrincipalName": "ah@wingtiptoysonline.onmicrosoft.com", 
            "userId": "5b9f356d-9592-42fd-9ec4-d70963909534", 
            "appId": "c44b4083-3bb0-49c1-b47d-974e53cbdf3c", 
            "appDisplayName": "Azure Portal", 
            "ipAddress": "167.220.0.158", 
            "status": { 
                "errorCode": 50140, 
                "failureReason": "Other" 
            }, 
            "clientAppUsed": "Browser", 
            "deviceDetail": { 
                "operatingSystem": "Windows 10", 
                "browser": "Chrome 66.0.3359" 
            }, 
            "location": { 
                "city": "Sammamish", 
                "state": "Washington", 
                "countryOrRegion": "US", 
                "geoCoordinates": { 
                    "latitude": 47.66630935668945, 
                    "longitude": -122.09821319580078 
                } 
            }, 
            "correlationId": "13e19598-e040-487f-bd32-d38a2cd75d9a", 
            "conditionalAccessStatus": 2, 
            "conditionalAccessPolicies": [ 
            { 
                "id": "de7e60eb-ed89-4d73-8205-2227def6b7c9", 
                "displayName": "[billg] SharePoint limited access policy", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "7412a2d8-cbb1-4f1c-96cf-8410b4b8b37b", 
                "displayName": "[BillG] AIP MFA Policy", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "727ed8ea-059d-4d8f-aba5-c1dc500e8b06", 
                "displayName": "[billg] mfa for mail", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "6701123a-b4c6-48af-8565-565c8bf7cabc", 
                "displayName": "Medium signin risk block", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "fbafa2da-cf7f-4ec3-83cf-281188e53f76", 
                "displayName": "Require MFA for admins [Ignite talk] ", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "15339054-709d-4e06-a9ec-342bf043ea56", 
                "displayName": "Enhanced proofing for Azure portal [Ignite talk]", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "2ff9436f-bc72-4ce6-b17e-e7e51153146e", 
                "displayName": "[calebb] AIP policy", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "46ab586b-9447-4847-a889-e60705d96e56", 
                "displayName": "Test policy, OR", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "ceb6e17e-a5d0-4b3a-a150-6c2be2d5b0e9", 
                "displayName": "mm policy with Duo", 
                "enforcedGrantControls": [ 
                    "Require Duo Mfa" 
                ], 
            "enforcedSessionControls": [], 
            "result": 2 
            }, 
            ], 
            "isRisky": false 
            } 
        } 
    } 
```

## <a name="field-descriptions"></a>Descriptions de champs

| Nom du champ | Description |
|------------|-------------|
| Temps | Date et heure UTC. |
| ResourceId | Valeur non mappée, vous pouvez ignorer ce champ.  |
| OperationName | Pour les connexions, cette valeur est toujours *Activité de connexion*. |
| operationVersion | Version d’API REST demandée par le client. |
| Catégorie | Pour les connexions, cette valeur est toujours *SignIn*. | 
| TenantId | GUID de locataire associé aux journaux. |
| ResultType | Le résultat de l’opération de connexion peut être *Success* (Réussite) ou *Failure* (Échec). | 
| ResultSignature | Contient le code d’erreur éventuel de l’opération de connexion. |
| resultDescription | Fournit la description de l’erreur pour l’opération de connexion. |
| DurationMs |  Valeur non mappée, vous pouvez ignorer ce champ.|
| callerIpAddress | Adresse IP du client à l’origine de la demande. | 
| CorrelationId | GUID facultatif transmis par le client. Cette valeur peut aider à corréler des opérations côté client avec des opérations côté serveur, et est utile lors du suivi de journaux couvrant plusieurs services. |
| Identité | Identité extraite du jeton présenté lors de la création de la demande. Il peut s’agir d’un compte d’utilisateur, d’un compte système ou d’un principal du service. |
| Niveau | Fournit le type de message. Pour l’audit, il s’agit toujours d’*Information*. |
| Lieu | Indique l’emplacement de l’activité de connexion. |
| properties | Répertorie toutes les propriétés associées aux connexions. Pour plus d’informations, voir la [documentation de référence sur l’API Microsoft Graph](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/signin). Ce schéma utilise les mêmes noms d’attribut que la ressource de connexion pour une meilleure lisibilité.

## <a name="next-steps"></a>Étapes suivantes

* [Interpréter le schéma des journaux d’audit dans Azure Monitor](reference-azure-monitor-audit-log-schema.md)
* [En savoir plus sur les journaux de diagnostic Azure](../../azure-monitor/platform/diagnostic-logs-overview.md)
