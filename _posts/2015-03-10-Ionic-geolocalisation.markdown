---
layout:     post
title:      "Ionic et la géolocalisation"
subtitle:   "Utiliser la géolocalisation dans une application ionic"
date:       2015-03-10 18:00:00
author:     "Arnaud LAHAXE"
header-img: "/assets/img/header/compass.jpg"
tags:       [ionic]
---

## Besoin

Notre application a besoin de la position de l'utilisateur en "temps réel" sans faire des appels incessant au gps du téléphone manuellement.

## Dépendances

### Installation de ngCordova

NgCordova est une collection de services et d'extensions AngularJS qui facilitent l'utilisation des plugins Cordova et mais également les fonctionnalités natives implémentées par Cordova.

Installation de ngCordova via bower
{% highlight bash %}
npm install -g ngCordova --save
{% endhighlight %}

Ajout de la librairie dans notre application. Avant  `cordova.js`.
{% highlight html %}
<script src="/lib/ngCordova/dist/ng-cordova.js"></script>
<script src="/cordova.js"></script>
{% endhighlight %}

Injection de ngCordova dans l'application
{% highlight javascript %}
app = angular.module('geolocalisationApp', ['ngCordova'])
{% endhighlight %}

### Module de géolocalisation

NgCordova propose un module de [Geolocalisation](http://ngcordova.com/docs/plugins/geolocation/).

{% highlight bash %}
cordova plugin add org.apache.cordova.geolocation
{% endhighlight %}

Il est parfois nécessaire de supprimer et et de ré-ajouter les différentes plate-formes apres l'ajout d'un plugin cordova.

## Code

### Géolocalisation par défaut

Il faut prevoir un fallback au cas ou le téléphone n'ait pas de GPS où qu'il soit désactivé.

{% highlight javascript %}
 // fall back
app.constant('defaultLocalisation', {
    'longitude': 6.1799699326036,
    'latitude': 48.689290283084
})
{% endhighlight %}

### Le localStorage

Afin de diminuer le nombre d'accès au GPS nous allons nous faire une factory qui va travailler avec le localstorage. Ce composant est ré-utilisable pour stocker d'autres données.


{% highlight javascript %}
app.factory('$localStorage', ['$window', function ($window) {
    return {
        set: function (key, value) {
            $window.localStorage[key] = value;
        },
        get: function (key, defaultValue) {
            return $window.localStorage[key] || defaultValue;
        },
        setObject: function (key, value) {
            $window.localStorage[key] = JSON.stringify(value);
        },
        getObject: function (key) {
            return JSON.parse($window.localStorage[key] || '{}');
        }
    }
}])
{% endhighlight %}

### La factory qui va contenir la géolocalisation

Cette factory va intéragir avec le localStorage en enregistrant et en récupérant la position GPS.

{% highlight javascript %}
app.factory('geoLocation', function ($localStorage) {
    return {
        setGeolocation: function (latitude, longitude) {
            var position = {
                latitude: latitude,
                longitude: longitude
            }
            $localStorage.setObject('geoLocation', position)
        },
        getGeolocation: function () {
            return glocation = {
                lat: $localStorage.getObject('geoLocation').latitude,
                lng: $localStorage.getObject('geoLocation').longitude
            }
        }
    }
})
{% endhighlight %}

### Initialisation au run

A l'initisalisation de notre module on va lancer la détection de de la position GPS, on va aussi mettre un place un watcher sur cette variable. Quand on va detecter une moification de la géolocalisation on va créer un évènement dans le `$rootscope`. Les autres `objets` pourront s'abonner à cet évènement.


L'intéraction avec les plugins cordova doivent forcement se faire dans un `$ionicPlatform.ready`.

{% highlight javascript %}
// initialisation
app.run(function ($ionicPlatform, $cordovaGeolocation, geoLocation, $rootScope, defaultLocalisation, $ionicPopup) {
    // waiting the platform ready event
    $ionicPlatform.ready(function () {
        $cordovaGeolocation
            .getCurrentPosition()
            .then(function (position) {
                geoLocation.setGeolocation(position.coords.latitude, position.coords.longitude);
            }, function (err) {
                 // you need to enhance that point
                $ionicPopup.alert({
                    title: 'Ooops...',
                    template: err.message
                });

                geoLocation.setGeolocation(defaultLocalisation.latitude, defaultLocalisation.longitude)
            });

        // begin a watch
        var watch = $cordovaGeolocation.watchPosition({
            frequency: 1000,
            timeout: 3000,
            enableHighAccuracy: false
        }).then(function () {
            }, function (err) {
                // you need to enhance that point
                geoLocation.setGeolocation(defaultLocalisation.latitude, defaultLocalisation.longitude);
            }, function (position) {
                geoLocation.setGeolocation(position.coords.latitude, position.coords.longitude);
                //  broadcast this event on the rootScope
                $rootScope.$broadcast('location:change', geoLocation.getGeolocation());
            }
        );
    });
})
{% endhighlight %}

## Utilisation

Le fait d'avoir wrappé toute la logique dans des factories nous permet d'utiliser la geolocalisation en toute simplicité.

Dans cet exemple le controller écoute l'évènement `location:change`.

{% highlight javascript %}
app.controller('myController', function ($scope, geoLocation, $rootScope) {
    var position = geoLocation.getGeolocation();

    // listen location changes
    $rootScope.$on('location:change', function (position) {
        // do something
    });
});
{% endhighlight %}