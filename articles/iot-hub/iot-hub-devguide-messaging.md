---
title: Présentation des messages Azure IoT Hub | Microsoft Docs
description: Guide du développeur - Messagerie d’appareil-à-cloud et de cloud-à-appareil avec IoT Hub. Comprend des informations sur les formats de message et les protocoles de communication pris en charge.
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/29/2018
ms.author: dobett
ms.openlocfilehash: d3a7284555fb592956d4e1dc3f56137c88d108e1
ms.sourcegitcommit: 5843352f71f756458ba84c31f4b66b6a082e53df
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/01/2018
ms.locfileid: "47584387"
---
# <a name="send-device-to-cloud-and-cloud-to-device-messages-with-iot-hub"></a>Envoyer des messages d’appareil-à-cloud et de cloud-à-appareil avec IoT Hub

IoT Hub permet une communication bidirectionnelle avec vos appareils. Utilisez la messagerie IoT Hub pour communiquer avec vos appareils en envoyant des messages depuis vos appareils au backend de vos solutions et pour envoyer des commandes depuis le backend de vos solutions IoT à vos appareils. Découvrez plus d’informations sur le [format des messages IoT Hub](iot-hub-devguide-messages-construct.md).

## <a name="sending-device-to-cloud-messages-to-iot-hub"></a>Envoi de messages appareil-à-cloud à IoT Hub

IoT Hub a un point de terminaison de service intégré qui peut être utilisé par des services backend pour lire des messages de télémétrie provenant de vos appareils. Ce point de terminaison est compatible avec [Event Hubs](https://docs.microsoft.com/azure/event-hubs/) et vous pouvez utiliser les SDK IoT Hub standard pour [lire sur ce point de terminaison intégré](iot-hub-devguide-messages-read-builtin.md).

IoT Hub prend également en charge les [points de terminaison personnalisés](iot-hub-devguide-endpoints.md#custom-endpoints), qui peuvent être définis par les utilisateurs pour envoyer des données de télémétrie et des événements des appareils à des services Azure avec le [routage des messages](iot-hub-devguide-messages-d2c.md).

## <a name="sending-cloud-to-device-messages-from-iot-hub"></a>Envoi de messages cloud-à-appareil depuis IoT Hub

Vous pouvez envoyer des messages [cloud-à-appareil](iot-hub-devguide-messages-c2d.md) depuis le backend de la solution vers vos appareils.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Les principales propriétés de la fonctionnalité de messagerie IoT Hub sont la fiabilité et la durabilité des messages. Ces propriétés activent la résilience de la connectivité intermittente côté appareils et des pics de chargement dans le traitement d’événements côté cloud. IoT Hub implémente *au moins une fois* des garanties de remise pour l’envoi de messages appareil-à-cloud et cloud-à-appareil.

## <a name="choosing-the-right-type-of-iot-hub-messaging"></a>Choix du type approprié de messagerie IoT Hub

Utilisez les messages appareil-à-cloud pour envoyer des alertes et des données de télémétrie de série chronologique à partir de votre application pour appareil, et des messages cloud-à-appareil pour envoyer des notifications unidirectionnelles à votre application pour appareil.

* Reportez-vous à [Aide sur la communication appareil-à-cloud](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-d2c-guidance) pour choisir entre les messages appareil-à-cloud, les propriétés signalées ou le chargement de fichiers.

* Reportez-vous à [l’aide sur la communication cloud-à-appareil](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-c2d-guidance) pour choisir entre les messages cloud-à-appareil, les propriétés souhaitées ou les méthodes directes.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez plus d’informations sur le [routage des messages](iot-hub-devguide-messages-d2c.md) IoT Hub.

* Découvrez plus d’informations sur la [messagerie cloud-à-appareil](iot-hub-devguide-messages-c2d.md) IoT Hub.