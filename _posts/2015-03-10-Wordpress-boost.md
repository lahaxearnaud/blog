---
layout:     post
title:      "Booster wordpress"
subtitle:   "Optimiser wordpress afin de diminuer drastiquement le temps de réponse"
date:       2015-03-10 12:00:00
author:     "Arnaud LAHAXE"
header-img: "img/22.jpg"
tags: [wordpress]
---

Diminuer le temps de réponse de son site devrait être la préoccupation de tout webmaster, afin d'améliorer au maximum l'expérience utilisateur mais aussi pour être mieux référencé sur Google. Nous allons voir dans cet article comment optimiser Wordpress pour obtenir un temps de réponse minimal.


[![logo-v-rgb](/img/wordpress/logo.png)](/img/wordpress/logo.png)


Il existe une kyrielle de plugins Wordpress qui permettent de réduire le temps de réponse de Wordpress. Un des plus connus est [w3 Total Cache](http://wordpress.org/plugins/w3-total-cache/) car il possède énormément de fonctionnalités.

Mais après l'avoir installé, je me suis rendu compte que Wordpress consommait quasi 100% de tous les coeurs de mon serveur pour afficher une page. C'est un problème qui est assez fréquent avec ce plugin. C'est pourquoi j'ai cherché des solutions alternatives.

[![htop](/img/wordpress/boost/htop.png)](/img/wordpress/boost/htop.png)


## Coté Wordpress

### WP Super Cache


[WP Super Cache](http://wordpress.org/plugins/wp-super-cache/) est développé par [Donncha O Caoimh](http://ocaoimh.ie/wordpress-plugins/ "Aller sur la page de l’auteur"), auteur de la branche Wp MU.

Le plugin va générer des pages HTML statiques à partir des pages du sites. Les pages sont mises à jour régulièrement (réglable). Ce procédé permet de servir des pages sans avoir à la générer.

La seconde fonctionnalité intéressante du plugin est le pré-chargement des pages pour forcer la mise en cache. Il est possible par exemple de pré-générer ou mettre à jour les caches toutes les 30 minutes. Cette fonctionnalité est très intéressante car même le premier visiteur profitera du cache, contrairement à la plupart des plugins de cache ou c'est le premier visiteur qui va le générer.

### WP Minify

[Wp Minify](http://wordpress.org/plugins/wp-minify/) est un plugin qui permet la compression et la concaténation des fichiers JS et CSS. Selon la qualité de vos fichiers (en terme de code) il n'est peut être pas possible de les compresser. Dans ce cas la il faut aller dans option avancé et désactiver la minification.

La concaténation des assets, même non minifiés, permet de diminuer le nombre de requêtes et donc le temps de réponse.

### WP Optimize


[Wp Optimize](http://wordpress.org/plugins/wp-optimize/) est un plugin qui permet de nettoyer et optimiser la base de données de wordpress. Il va aussi défragmenter les tables. Il suffit de le lancer de temps en temps.


### jQuery lazy load plugin


[jQuery lazy load plugin](http://wordpress.org/plugins/jquery-image-lazy-loading/) est un plugin qui pemet de charger les images des pages à la demande. le plugin est basé sur le plugin JQuery [Lazy Load](http://www.appelsiini.net/projects/lazyload).

Ceci permet de diminuer grandement le temps de chargement initial d'une page. Et c'est quand l'utilisateur affichera l'image qu'elle sera chargée.

Sur une page d'accueil où il y a une dizaine d'articles (ou plus) cela peut faire gagner énormément de connexion HTTP.

## Conseils de bon sens


*   Eviter d'installer trop de plugins
*   Eviter d'utiliser des JS et CSS externe (hors [CDN](http://fr.wikipedia.org/wiki/Content_delivery_network))
*   Eviter les images trop lourdes, au possible faites des [Sprites](http://blog.lahaxe.fr/2011/06/10/optimisation-de-pages-web/ "Optimisation de pages web")

## Côté Serveur


Si on veux que le site soit aussi performant que possible, il va falloir, si possible, aller configurer un peu le serveur. Pour cette partie vous allez avoir besoin des accès root sur le serveur.


### XCache


[XCache](http://xcache.lighttpd.net/) est un accélérateur de code PHP. Il va mettre en cache le code PHP compilé afin de ne pas avoir à le re-compiler une prochaine fois.  Contrairement à PHP APC, XCache est stable et entièrement compatible avec PHP 5.4+.


Voici un tutoriel qui date un peu, mais qui est toujours d'actualité, qui explique comment faire l'installation de XCache sur un serveur debian : [Installer XCache sur Debian](http://fredmac.blogspot.fr/2011/01/installer-xcache-sur-debian-lenny.html).

### Configurer Apache

Apache est très configurable, le tutoriel [ optimiser la configuration d’Apache ](http://www.e-peps.fr/configuration-apache2/) est assez complet à ce sujet.

### MySQL

Pour une installation basique de wordpress le SGBD est MySQL.

Il existe des scripts comme [MySQLTuner](https://github.com/rackerhacker/MySQLTuner-perl "MySQLTuner") qui peuvent aider à configurer correctement le serveur MySQL.

### Installer un Firewall

Un firewall, par exemple [Shorewall](http://blog.lahaxe.fr/2013/08/15/installation-et-configuration-de-shorewall-sous-debian/ "Installation et configuration de shorewall sous debian") permet de faire le tri dans les demandes et de ne pas envoyer à Apache les requêtes de type DDOS.
Contrairement au [mod_evasive](http://www.tux-planet.fr/mod_evasive-un-module-anti-dos-pour-apache/) la requête va être ignorée au niveau de la carte réseau. On va donc limiter au maximum son impact sur le serveur en termes de ressources utilisées.

### Résultat

Voici le résultat de [Yslow](https://addons.mozilla.org/fr/firefox/addon/yslow/) sur la page d'accueil du blog une fois toutes les optimisations mises en place.

[![Yslow](/img/wordpress/boost/grade.png)](/img/wordpress/boost/grade.png)



Et voici le temps de chargement de la vue réseau de Firebug pour le premier et le second chargement de la page d'accueil:

**Premier chargement**

[![Avant](/img/wordpress/boost/after.png)](/img/wordpress/boost/after.png)

**Second chargement**

[![Après](/img/wordpress/boost/before.png)](/img/wordpress/boost/before.png)