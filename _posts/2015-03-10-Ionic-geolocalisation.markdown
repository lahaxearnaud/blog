---
layout:     post
title:      "Ionic geolocalisation"
subtitle:   "Utiliser la géolocalisation dans une application ionic"
date:       2015-03-10 18:00:00
author:     "Arnaud LAHAXE"
header-img: "/assets/img/header/60.jpg"
tags: [ionic, protip]
---

## Code

{% highlight javascript %}
angular.module('geolocalisation', ['ngCordova'])

    // fall back
    .constant('defaultLocalisation', {
        'longitude': 6.1799699326036,
        'latitude': 48.689290283084
    })

    // initialisation
    .run(function ($ionicPlatform, $cordovaGeolocation, geoLocation, $rootScope, defaultLocalisation, $ionicPopup) {

        $ionicPlatform.ready(function () {
            $cordovaGeolocation
                .getCurrentPosition()
                .then(function (position) {
                    geoLocation.setGeolocation(position.coords.latitude, position.coords.longitude);
                }, function (err) {

                    $ionicPopup.alert({
                        title: 'Ooops...',
                        template: err.message
                    });

                    geoLocation.setGeolocation(defaultLocalisation.latitude, defaultLocalisation.longitude)
                });

            // begin a watch
            var options = {
                frequency: 1000,
                timeout: 3000,
                enableHighAccuracy: false
            };

            var watch = $cordovaGeolocation.watchPosition(options);
            watch.then(function () {
                },
                function (err) {
                    geoLocation.setGeolocation(defaultLocalisation.latitude, defaultLocalisation.longitude);
                }, function (position) {
                    geoLocation.setGeolocation(position.coords.latitude, position.coords.longitude);
                    $rootScope.$broadcast('location:change', position.coords);
                }
            );
        });
    })

    .factory('$localStorage', ['$window', function ($window) {
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

    .factory('geoLocation', function ($localStorage) {
        return {
            setGeolocation: function (latitude, longitude) {
                var _position = {
                    latitude: latitude,
                    longitude: longitude
                }
                $localStorage.setObject('geoLocation', _position)
            },
            getGeolocation: function () {
                return glocation = {
                    lat: $localStorage.getObject('geoLocation').latitude,
                    lng: $localStorage.getObject('geoLocation').longitude
                }
            }
        }
    });
{% endhighlight %}

## Utilisation

{% highlight javascript %}
    app.controller('myController', function ($scope, geoLocation, $rootScope) {
        var position = geoLocation.getGeolocation();

        // listen location changes
        $rootScope.$on('location:change', function (position) {
            // do something
        });
    });

{% endhighlight %}