---
title: Itinéraires multiples avec Azure Maps | Microsoft Docs
description: Rechercher des itinéraires pour différents modes de déplacement avec Azure Maps
author: walsehgal
ms.author: v-musehg
ms.date: 11/14/2018
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: 0ec047f38596bed4d3f0bc5520dc9c7fc18b4c24
ms.sourcegitcommit: 7723b13601429fe8ce101395b7e47831043b970b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/21/2019
ms.locfileid: "56585234"
---
# <a name="find-routes-for-different-modes-of-travel-using-azure-maps"></a>Rechercher des itinéraires pour différents modes de déplacement avec Azure Maps

Ce didacticiel montre comment utiliser votre compte Azure Maps et Route Service pour trouver l’itinéraire vers le point d’intérêt de votre choix, en fonction de votre mode de déplacement. Vous affichez deux itinéraires différents sur votre carte, un dédié aux voitures et l’autre aux camions, qui peuvent présenter des restrictions relatives à la hauteur, au poids ou aux chargements dangereux. Ce tutoriel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une page web à l’aide de l’API Map Control
> * Visualiser le flux du trafic sur votre carte
> * Créer des requêtes d’itinéraire déclarant le mode de déplacement
> * Afficher plusieurs itinéraires sur votre carte

## <a name="prerequisites"></a>Prérequis

Avant de continuer, exécutez les étapes du premier didacticiel afin de [créer votre compte Azure Maps](./tutorial-search-location.md#createaccount) et d’[obtenir la clé d’abonnement pour votre compte](./tutorial-search-location.md#getkey).

## <a name="create-a-new-map"></a>Créer une carte

Les étapes suivantes vous indiquent comment créer une page HTML statique intégrée avec l’API Map Control.

1. Sur votre ordinateur local, créez un fichier et nommez-le **MapTruckRoute.html**.
2. Ajoutez les composants HTML suivants au fichier :

    ```HTML
    <!DOCTYPE html>
    <html>
    <head>
        <title>Multiple routes by mode of travel</title>
        
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/css/atlas.min.css?api-version=1" type="text/css" />
        <script src="https://atlas.microsoft.com/sdk/js/atlas.min.js?api-version=1"></script>

        <!-- Add a reference to the Azure Maps Services Module JavaScript file. -->
        <script src="https://atlas.microsoft.com/sdk/js/atlas-service.js?api-version=1"></script>

        <script>
            var map, datasource, client;

            function GetMap() {
                //Add Map Control JavaScript code here.
            }
        </script>
        <style>
            html,
            body {
                width: 100%;
                height: 100%;
                padding: 0;
                margin: 0;
            }

            #myMap {
                position: relative;
                width: 100%;
                height: 100%;
            }
        </style>
    </head>
    <body onload="GetMap()">
        <div id="myMap"></div>
    </body>
    </html>
    ```
    
    Notez que l’en-tête HTML inclut les fichiers de ressources CSS et JavaScript hébergés par la bibliothèque Azure Map Control. Notez l’événement `onload` sur le corps de la page, qui appelle la fonction `GetMap` lorsque le corps de la page est chargé. Cette fonction est destinée à contenir du code JavaScript inline pour accéder aux API Azure Maps.

3. Ajoutez le code JavaScript suivant à la fonction `GetMap`. Remplacez la chaîne **\<Your Azure Maps Key\>** (Votre clé Azure Maps) par la clé primaire copiée de votre compte Maps.

    ```JavaScript
    //Add your Azure Maps subscription key to the map SDK. 
    atlas.setSubscriptionKey('<Your Azure Maps Key>');

    //Initialize a map instance.
    map = new atlas.Map('myMap');
    ```

    **atlas.Map** fournit le contrôle d’une carte web visuelle et interactive et est un composant de l’API Azure Map Control.

4. Enregistrez le fichier et ouvrez-le dans votre navigateur. À ce stade, vous disposez d’une carte de base que vous pouvez développer.

   ![Afficher la carte de base](./media/tutorial-prioritized-routes/basic-map.png)

## <a name="visualize-traffic-flow"></a>Visualiser le flux du trafic

1. Ajoutez l’affichage du trafic à la carte. `map.events.add` assure que toutes les fonctions de correspondance ajoutées à la carte sont chargées une fois le chargement de la carte terminé.

    ```JavaScript
    map.events.add("load", function() {
        // Add Traffic Flow to the Map
        map.setTraffic({
            flow: "relative"
        });
    });
    ```
    
     Un événement de chargement est ajouté à la carte, qui est déclenché lorsque les ressources de la carte ont été entièrement chargées. Dans le gestionnaire d’événements de chargement de la carte, le paramètre de flux de trafic sur la carte est défini sur `relative`, c’est-à-dire la vitesse de la route par rapport au trafic fluide. Vous pouvez également le définir sur la vitesse `absolute` de la route, ou sur `relative-delay` qui affiche la vitesse relative lorsqu’elle diffère du trafic fluide.

2. Enregistrez le fichier **MapTruckRoute.html** et actualisez la page dans votre navigateur. Vous devriez observer les rues de Los Angeles avec leurs données de trafic actuelles.

   ![Afficher les données de trafic](./media/tutorial-prioritized-routes/traffic-map.png)

<a id="queryroutes"></a>

## <a name="define-how-the-route-will-be-rendered"></a>Définir le rendu de l’itinéraire

Dans ce didacticiel, deux itinéraires sont calculés et affichés sur la carte. Un itinéraire utilise des routes accessibles aux voitures et l’autre des routes accessibles aux camions. Lors du rendu, nous allons afficher une icône de symbole pour le départ et l’arrivée de l’itinéraire, et des lignes de couleurs différentes pour chaque tracé d’itinéraire.

1. Dans la fonction GetMap, après l’initialisation de la carte, ajoutez le code JavaScript suivant.

    ```JavaScript
    //Wait until the map resources have fully loaded.
    map.events.add('load', function () {

        //Create a data source and add it to the map.
        datasource = new atlas.source.DataSource();
        map.sources.add(datasource);

        //Add a layer for rendering the route lines and have it render under the map labels.
        map.layers.add(new atlas.layer.LineLayer(datasource, null, {
            strokeColor: ['get', 'strokeColor'],
            strokeWidth: ['get', 'strokeWidth'],
            lineJoin: 'round',
            lineCap: 'round',
            filter: ['==', '$type', 'LineString']
        }), 'labels');

        //Add a layer for rendering point data.
        map.layers.add(new atlas.layer.SymbolLayer(datasource, null, {
            iconOptions: {
                image: ['get', 'icon'],
                allowOverlap: true
            },
            textOptions: {
                textField: ['get', 'title'],
                offset: [0, 1.2]
            },
            filter: ['==', '$type', 'Point']
        }));
    });
    ```
    
    Un événement de chargement est ajouté à la carte, qui est déclenché lorsque les ressources de la carte ont été entièrement chargées. Dans le gestionnaire d’événements de chargement de la carte, une source de données est créée pour stocker les lignes d’itinéraire ainsi que les points de départ et d’arrivée. Une couche de ligne est créée et jointe à la source de données pour définir le rendu de la ligne d’itinéraire. Des expressions sont utilisées pour récupérer la largeur et la couleur de ligne des propriétés de la fonctionnalité de ligne d’itinéraire. Un filtre est ajouté pour s’assurer que cette couche n’affiche que les données GeoJSON LineString. Lors de l’ajout de la couche à la carte, un deuxième paramètre avec la valeur `'labels'` est transmis et spécifie le rendu de cette couche sous les étiquettes de carte. Cela garantit que la ligne d’itinéraire ne couvre pas les étiquettes de route. Une couche de symbole est créée et jointe à la source de données. Cette couche spécifie comment les points de départ et d’arrivée sont affichés, dans ce cas des expressions ont été ajoutées pour récupérer les informations d’image d’icône et d’étiquette de texte des propriétés de chaque objet de point. 
    
2. Pour ce didacticiel, définissez le point de départ sur une entreprise fictive de Seattle appelée Fabrikam et le point d’arrivée sur un siège Microsoft. Dans le gestionnaire d’événements de chargement de la carte, ajoutez le code suivant.

    ```JavaScript
    //Create the GeoJSON objects which represent the start and end point of the route.
    var startPoint = new atlas.data.Feature(new atlas.data.Point([-122.356099, 47.580045]), {
        title: 'Fabrikam, Inc.',
        icon: 'pin-round-blue'
    });

    var endPoint = new atlas.data.Feature(new atlas.data.Point([-122.201164, 47.616940]), {
        title: 'Microsoft - Lincoln Square',
        icon: 'pin-blue'
    });
    ```
    
    Ce code crée deux [objets GeoJSON](https://en.wikipedia.org/wiki/GeoJSON) pour représenter les points de départ et d’arrivée de l’itinéraire. Une propriété `title` et `icon` est ajoutée à chaque point.

3. Ajoutez ensuite le code JavaScript suivant pour ajouter à la carte les marqueurs des points de départ et d’arrivée :

    ```JavaScript
    //Add the data to the data source.
    datasource.add([startPoint, endPoint]);

    //Fit the map window to the bounding box defined by the start and end positions.
    map.setCamera({
        bounds: atlas.data.BoundingBox.fromData([startPoint, endPoint]),
        padding: 100
    });
    ```
    
    Les points de départ et d’arrivée sont ajoutés à la source de données. Le rectangle englobant des points de départ et d’arrivée est calculé à l’aide de la fonction `atlas.data.BoundingBox.fromData`. Ce rectangle englobant est utilisé pour définir la vue de caméra de la carte sur le point de départ et d’arrivée à l’aide de la fonction `map.setCamera`. Une marge intérieure est ajoutée pour compenser les dimensions en pixels des icônes de symbole.

4. Enregistrez le fichier et actualisez votre navigateur afin d’afficher les repères sur votre carte. Désormais, la carte est centrée sur Seattle, et vous pouvez observer le repère rond bleu marquant le point de départ et l’autre repère bleu marquant le point d’arrivée.

   ![Afficher la carte avec les points de départ et d’arrivée](./media/tutorial-prioritized-routes/pins-map.png)

<a id="multipleroutes"></a>

## <a name="render-routes-prioritized-by-mode-of-travel"></a>Afficher les itinéraires priorisés par mode de déplacement

Cette section montre comment utiliser l’API Route Service d’Azure Maps pour rechercher plusieurs itinéraires entre un point de départ donné et une destination, en fonction de votre mode de transport. Route Service fournit des API pour planifier les itinéraires les plus *rapides*, *courts*, *économiques* ou *intéressants* entre deux emplacements, en fonction des conditions de circulation actuelles. Grâce à la base de données de trafic historique complète d’Azure, il permet également aux utilisateurs de planifier des itinéraires, durées comprises, pour n’importe quels jour et heure. Pour plus d’informations, consultez [Get route directions](https://docs.microsoft.com/rest/api/maps/route/getroutedirections) (Obtenir les itinéraires). Tous les blocs de code suivants doivent être ajoutés **dans l’eventListener du chargement de la carte** pour assurer leur chargement une fois la carte complètement chargée.

1. Instanciez le service client en ajoutant le code JavaScript suivant dans le gestionnaire d’événements de chargement de la carte.

    ```JavaScript
    //If the service client hasn't been created, create an instance of it.
    if (!client) {
        client = new atlas.service.Client(atlas.getSubscriptionKey());
    }
    ```
    
2. Ajoutez le bloc de code suivant pour construire une chaîne de requête d’itinéraire.

    ```JavaScript
    //Create the route request with the query being the start and end point in the format 'startLongitude,startLatitude:endLongitude,endLatitude'.
    var routeQuery = startPoint.geometry.coordinates[1] +
        ',' +
        startPoint.geometry.coordinates[0] +
        ':' +
        endPoint.geometry.coordinates[1] +
        ',' +
        endPoint.geometry.coordinates[0];
    ```

3. Ajoutez le code JavaScript suivant afin de solliciter l’itinéraire pour un camion transportant des marchandises USHazmatClass2 et d’afficher les résultats :

    ```JavaScript
    //Execute the truck route query then add the route to the map.
    client.route.getRouteDirections(routeQuery, {
        travelMode: 'truck',
        vehicleWidth: 2,
        vehicleHeight: 2,
        vehicleLength: 5,
        vehicleLoadType: 'USHazmatClass2'
    }).then(function (response) {
        // Parse the response into GeoJSON
        var geoJsonResponse = new atlas.service.geojson.GeoJsonRouteDirectionsResponse(response);

        //Get the route line and add some style properties to it.
        var routeLine = geoJsonResponse.getGeoJsonRoutes().features[0];
        routeLine.properties.strokeColor = '#2272B9';
        routeLine.properties.strokeWidth = 9;

        //Add the route line to the data source. We want this to render below the car route which will likely be added to the data source faster, so insert it at index 0.
        datasource.add(routeLine, 0);
    });
    ```
    Cet extrait de code interroge le service de routage d’Azure Maps via la méthode [getRouteDirections](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.models.routedirectionsrequestbody?view=azure-iot-typescript-latest), puis analyse la réponse au format GeoJSON à l’aide de [getGeoJsonRouteDirectionsResponse](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.routegeojson?view=azure-iot-typescript-latest). Il crée ensuite un tableau de coordonnées pour l’itinéraire retourné et l’ajoute à la source de données, mais ajoute également un index de 0 pour vous assurer qu’il est restitué avant toutes les autres lignes dans la source de données. Ceci est dû au fait que le calcul d’itinéraire réservé aux camions est généralement plus lent qu’un calcul d’itinéraire réservé aux voitures et que, si la ligne d’itinéraire réservé aux camions est ajoutée à la source de données après l’itinéraire réservé aux voitures, il s’affiche au-dessus de celui-ci. Deux propriétés sont ajoutées à la ligne d’itinéraire réservé aux camions : un trait de couleur bleue et une épaisseur de trait de 9 pixels. 

4. Ajoutez le code JavaScript suivant afin de solliciter l’itinéraire pour une voiture et afficher les résultats :

    ```JavaScript
    //Execute the car route query then add the route to the map.
    client.route.getRouteDirections(routeQuery).then(function (response) {
        // Parse the response into GeoJSON
        var geoJsonResponse = new atlas.service.geojson.GeoJsonRouteDirectionsResponse(response);

        //Get the route line and add some style properties to it.
        var routeLine = geoJsonResponse.getGeoJsonRoutes().features[0];
        routeLine.properties.strokeColor = '#B76DAB';
        routeLine.properties.strokeWidth = 5;

        //Add the route line to the data source.
        datasource.add(routeLine);
    });
    ```
    Cet extrait de code utilise la requête d’itinéraire de camion, mais pour une voiture. Il interroge le service de routage d’Azure Maps via la méthode [getRouteDirections](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.models.routedirectionsrequestbody?view=azure-iot-typescript-latest), puis analyse la réponse au format GeoJSON à l’aide de [getGeoJsonRouteDirectionsResponse](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.routegeojson?view=azure-iot-typescript-latest). Il crée ensuite un tableau de coordonnées pour l’itinéraire retourné et l’ajoute à la source de données. Deux propriétés sont ajoutées à la ligne d’itinéraire réservé aux voitures : un trait de couleur violette et une épaisseur de trait de 5 pixels. 

5. Enregistrez le fichier **MapTruckRoute.html** et actualisez votre navigateur afin d’afficher les résultats. Dans le cadre d’une connexion efficace avec les API Maps, vous devriez observer une carte similaire au contenu suivant.

    ![Itinéraires priorisés avec Azure Route Service](./media/tutorial-prioritized-routes/prioritized-routes.png)

    L’itinéraire réservé aux camions est bleu et plus épais, tandis que celui réservé aux voitures est mauve et plus fin. L’itinéraire réservé aux voitures passe au-dessus du Lac Washington via l’I-90, qui traverse des tunnels installés sous des zones résidentielles, ce qui interdit tout transport de déchets dangereux. L’itinéraire des camions, pour lequel est défini le type de chargement USHazmatClass2, utilise à raison une voie de circulation différente.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à :

> [!div class="checklist"]
> * Créer une page web à l’aide de l’API Map Control
> * Visualiser le flux du trafic sur votre carte
> * Créer des requêtes d’itinéraire déclarant le mode de déplacement
> * Afficher plusieurs itinéraires sur votre carte

Vous trouverez l’exemple de code de ce didacticiel à cet emplacement :

> [Itinéraires multiples avec Azure Maps](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/truckRoute.html)

[Consulter l’exemple en direct ici](https://azuremapscodesamples.azurewebsites.net/?sample=Multiple%20routes%20by%20mode%20of%20travel)

Le tutoriel suivant montre comment créer un localisateur de magasin simple à l’aide d’Azure Maps.

> [!div class="nextstepaction"]
> [Créer un localisateur de magasin à l’aide d’Azure Maps](./tutorial-create-store-locator.md)
