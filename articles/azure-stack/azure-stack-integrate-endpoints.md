---
title: 'Intégration au centre de données Azure Stack : publier des points de terminaison | Microsoft Docs'
description: Découvrez comment publier des points de terminaison Azure Stack dans votre centre de données
services: azure-stack
author: jeffgilb
manager: femila
ms.service: azure-stack
ms.topic: article
ms.date: 02/06/2019
ms.author: jeffgilb
ms.reviewer: wamota
ms.lastreviewed: 02/06/2019
ms.openlocfilehash: c3b27291fc413310393cd0270ec750de14a4985b
ms.sourcegitcommit: f715dcc29873aeae40110a1803294a122dfb4c6a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2019
ms.locfileid: "56270060"
---
# <a name="azure-stack-datacenter-integration---publish-endpoints"></a>Intégration au centre de données Azure Stack : publier des points de terminaison

Azure Stack configure des adresses IP virtuelles pour ses rôles d’infrastructure. Ces adresses IP virtuelles sont allouées à partir du pool d’adresses IP publiques. Chaque adresse IP virtuelle est sécurisée à l’aide d’une liste de contrôle d’accès (ACL) dans la couche réseau à définition logicielle. Les listes ACL sont également utilisées dans les commutateurs physiques (TOR et BMC) pour renforcer la solution. Une entrée DNS est créée pour chaque point de terminaison dans la zone DNS externe spécifiée au moment du déploiement.


Le diagramme architectural suivant montre les différentes couches réseau et les listes ACL :

![Image structurelle](media/azure-stack-integrate-endpoints/Integrate-Endpoints-01.png)

## <a name="ports-and-protocols-inbound"></a>Ports et protocoles (en entrée)

Un ensemble d’adresses IP virtuelles d’infrastructure est nécessaire pour la publication des points de terminaison Azure Stack sur des réseaux externes. Le tableau *Point de terminaison (VIP)* affiche chaque point de terminaison, le port requis et le protocole. Consultez la documentation de déploiement spécifique au fournisseur de ressources pour les points de terminaison nécessitant des fournisseurs de ressources supplémentaires, comme le fournisseur de ressources SQL.

Les adresses IP virtuelles ne sont pas répertoriées car elles ne sont pas requises pour la publication Azure Stack.

> [!Note]  
> Les adresses IP virtuelles de l’utilisateur sont dynamiques, définies par les utilisateurs eux-mêmes, sans contrôle de la part de l’opérateur Azure Stack.

> [!Note]
> À partir de la mise à jour 1811, les ports de la plage 12495-30015 ne doivent plus être ouverts en raison de l’ajout de l'[hôte d’extension](azure-stack-extension-host-prepare.md).

|Point de terminaison (VIP)|Enregistrement A d’hôte DNS|Protocole|Ports|
|---------|---------|---------|---------|
|AD FS|Adfs.*&lt;region>.&lt;fqdn>*|HTTPS|443|
|Portail (administrateur)|Adminportal.*&lt;region>.&lt;fqdn>*|HTTPS|443|
|Adminhosting | *.adminhosting.\<region>.\<fqdn> | HTTPS | 443 |
|Azure Resource Manager (administrateur)|Adminmanagement.*&lt;region>.&lt;fqdn>*|HTTPS|443|
|Portail (utilisateur)|Portal.*&lt;region>.&lt;fqdn>*|HTTPS|443|
|Azure Resource Manager (utilisateur)|Management.*&lt;region>.&lt;fqdn>*|HTTPS|443|
|Graph|Graph.*&lt;region>.&lt;fqdn>*|HTTPS|443|
|Liste de révocation de certificat|Crl.*&lt;region>.&lt;fqdn>*|HTTP|80|
|DNS|&#42;.*&lt;region>.&lt;fqdn>*|TCP et UDP|53|
|Hébergement | *.hosting.\<region>.\<fqdn> | HTTPS | 443 |
|Key Vault (utilisateur)|&#42;.vault.*&lt;region>.&lt;fqdn>*|HTTPS|443|
|Key Vault (administrateur)|&#42;.adminvault.*&lt;region>.&lt;fqdn>*|HTTPS|443|
|File d’attente de stockage|&#42;.queue.*&lt;region>.&lt;fqdn>*|HTTP<br>HTTPS|80<br>443|
|Table de stockage|&#42;.table.*&lt;region>.&lt;fqdn>*|HTTP<br>HTTPS|80<br>443|
|Storage Blob|&#42;.blob.*&lt;region>.&lt;fqdn>*|HTTP<br>HTTPS|80<br>443|
|Fournisseur de ressources SQL|sqladapter.dbadapter.*&lt;region>.&lt;fqdn>*|HTTPS|44300-44304|
|Fournisseur de ressources MySQL|mysqladapter.dbadapter.*&lt;region>.&lt;fqdn>*|HTTPS|44300-44304|
|App Service|&#42;.appservice.*&lt;region>.&lt;fqdn>*|TCP|80 (HTTP)<br>443 (HTTPS)<br>8172 (MSDeploy)|
|  |&#42;.scm.appservice.*&lt;region>.&lt;fqdn>*|TCP|443 (HTTPS)|
|  |api.appservice.*&lt;region>.&lt;fqdn>*|TCP|443 (HTTPS)<br>44300 (Azure Resource Manager)|
|  |ftp.appservice.*&lt;region>.&lt;fqdn>*|TCP, UDP|21, 1021, 10001-10100 (FTP)<br>990 (FTPS)|
|Passerelles VPN|     |     |[Consultez le FAQ sur la passerelle VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-vpn-faq#can-i-traverse-proxies-and-firewalls-using-point-to-site-capability).|
|     |     |     |     |

## <a name="ports-and-urls-outbound"></a>Ports et URL (en sortie)

Azure Stack prend en charge uniquement les serveurs proxy transparents. Dans un déploiement où un proxy transparent transfère les données vers un serveur proxy traditionnel, vous devez autoriser les URL et les ports suivants pour les communications sortantes :

> [!Note]  
> Azure Stack ne prend pas en charge l’utilisation d’ExpressRoute pour joindre les services Azure répertoriés dans le tableau suivant.

|Objectif|URL de destination|Protocole|Ports|Réseau source|
|---------|---------|---------|---------|---------|
|Identité|login.windows.net<br>login.microsoftonline.com<br>graph.windows.net<br>https://secure.aadcdn.microsoftonline-p.com<br>office.com|HTTP<br>HTTPS|80<br>443|Adresse IP virtuelle publique - /27<br>Réseau d'infrastructure publique|
|Syndication de Place de marché|https://management.azure.com<br>https://&#42;.blob.core.windows.net<br>https://*.azureedge.net<br>https://&#42;.microsoftazurestack.com|HTTPS|443|Adresse IP virtuelle publique - /27|
|Correctif et mise à jour|https://&#42;.azureedge.net|HTTPS|443|Adresse IP virtuelle publique - /27|
|Inscription|https://management.azure.com|HTTPS|443|Adresse IP virtuelle publique - /27|
|Usage|https://&#42;.microsoftazurestack.com<br>https://*.trafficmanager.net |HTTPS|443|Adresse IP virtuelle publique - /27|
|Windows Defender|.wdcp.microsoft.com<br>.wdcpalt.microsoft.com<br>*.updates.microsoft.com<br>*.download.microsoft.com<br>https://msdl.microsoft.com/download/symbols<br>`https://www.microsoft.com/pkiops/crl`<br>`https://www.microsoft.com/pkiops/certs`<br>`https://crl.microsoft.com/pki/crl/products`<br>`https://www.microsoft.com/pki/certs`<br>https://secure.aadcdn.microsoftonline-p.com<br>|HTTPS|80<br>443|Adresse IP virtuelle publique - /27<br>Réseau d'infrastructure publique|
|NTP|(IP du serveur NTP fourni pour le déploiement)|UDP|123|Adresse IP virtuelle publique - /27|
|DNS|(IP du serveur DNS fourni pour le déploiement)|TCP<br>UDP|53|Adresse IP virtuelle publique - /27|
|CRL|URL (sous Points de distribution CRL sur votre certificat)|HTTP|80|Adresse IP virtuelle publique - /27|
|LDAP|Forêt Active Directory fournie pour l'intégration Graph|TCP<br>UDP|389|Adresse IP virtuelle publique - /27|
|LDAP SSL|Forêt Active Directory fournie pour l'intégration Graph|TCP|636|Adresse IP virtuelle publique - /27|
|LDAP GC|Forêt Active Directory fournie pour l'intégration Graph|TCP|3268|Adresse IP virtuelle publique - /27|
|LDAP GC SSL|Forêt Active Directory fournie pour l'intégration Graph|TCP|3269|Adresse IP virtuelle publique - /27|
|AD FS|Point de terminaison de métadonnées AD FS fourni pour l'intégration AD FS|TCP|443|Adresse IP virtuelle publique - /27|
|     |     |     |     |     |

> [!Note]  
> Les URL sortantes sont équilibrées en charge à l’aide d’Azure Traffic Manager pour offrir la meilleure connectivité possible en fonction de l’emplacement géographique. Avec des URL équilibrées en charge, Microsoft peut mettre à jour et modifier des points de terminaison de backend sans impact sur les clients. Microsoft ne partage pas la liste des adresses IP pour les URL équilibrées en charge. Vous devez utiliser un appareil qui prend en charge le filtrage par URL plutôt que par adresse IP.

## <a name="next-steps"></a>Étapes suivantes

[Exigences relatives à l’infrastructure à clé publique d’Azure Stack](azure-stack-pki-certs.md)
