---
layout:     post
title:      "Shorewall sous debian"
subtitle:   "Installation et configuration des règles de shorewall sous debian"
date:       2015-03-10 9:00:00
author:     "Arnaud LAHAXE"
header-img: "img/54.jpg"
tags: [debian]
---

_Le tuto fonctionne aussi sous Ubuntu 12.04_


Le pare-feu "Shoreline Firewall", plus communément appelé "[Shorewall](http://shorewall.net/ "Shorewall")", est un outil qui permet de configurer plus facilement [Netfilter](http://www.netfilter.org/ "Netfilter") ([IpTable](http://fr.wikipedia.org/wiki/Iptables "IpTable")).

[Shorewall](http://shorewall.net/ "Shorewall") est un outil qui permet de configurer [Netfilter](http://www.netfilter.org/ "Netfilter"), son travail est fini.

L'avantage de [Shorewall](http://shorewall.net/ "Shorewall") est qu'il est très flexible, une fois que l'on a compris la syntaxe.

Si vous avez d'autres firewalls il faut les désinstaller avant de commencer (ex: [firestarter](http://www.fs-security.com/ "FireStarter"))

##### Installation de shorewall

{% highlight bash %}
apt-get install shorewall shorewall-doc
{% endhighlight %}

{% highlight bash %}
# on va dans le HOME
cd ~
# Copie de la doc one-interface
cp -r /usr/share/doc/shorewall/examples/one-interface ~
#on va dans la doc one-interface
cd one-interface/
# on met en place les configs de base
cp interfaces policy zones rules /etc/shorewall/
# retour au home
cd ~
# suppression de la doc du home
rm -r one-interface/
{% endhighlight %}

Maintenant que les fichiers de configuration de base sont placés aux bons endroits nous allons pouvoir commencer à configurer [Shorewall](http://shorewall.net/ "Shorewall") selon nos besoins.

<div class="alert  alert-block">Par défaut les connexions entrantes sont refusées mais les connexions sortantes sont autorisées. Selon vos besoins vous pouvez changer la politique en éditant le fichier `/etc/shorewall/policy`.
Dans la suite de l'article on partira du principe que la politique utilisée est la politique par défaut.</div>

##### Mise en place des règles du firewall

Ouvrir le fichier qui contient les règles:

{% highlight bash %}
	nano /etc/shorewall/rules`
{% endhighlight %}

On a deux zones le "net" et le "$FW" (firewall).

Trois actions:

*   ACCEPT = accepter
*   REJECT = refuser
*   DROP = ignorer (préferez-le à reject)

Voici un exemple de fichier de configuration:

{% highlight bash %}
#ACTION         SOURCE          DEST            PROTO   DEST    SOURCE          ORIGINAL        RATE            USER/  $
#                                                       PORT    PORT(S)         DEST            LIMIT           GROUP
#SECTION ALL
#SECTION ESTABLISHED
#SECTION RELATED
SECTION NEW

# Drop Ping from the "bad" net zone.. and prevent your log from being flooded..

Ping(DROP)    net               $FW

# Open 123 for ntpd service on startup
ACCEPT   fw             net               udp   123

# Allow standard services in and out of the box
ACCEPT   fw             net               udp    53     # DNS
ACCEPT   fw             net               tcp    53     # DNS
ACCEPT   net            fw                tcp    22     # SSH
ACCEPT   net            fw                tcp    80     # HTTP
ACCEPT   net            fw                tcp    443    # HTTPS
ACCEPT   net            fw                tcp    21     # FTP
{% endhighlight %}


<div class="alert alert-warning alert-block">Avant de continuer pensez à vérifier que vous avez bien ajouté une règle pour autoriser votre connexion SSH. Par défaut c'est le port 22 mais il a pu être changé. Vérifiez que Port est bien à 22 dans : `etc/ssh/sshd_config`. Si ce n'est pas le cas il faudra modifier la règle correspondant au SSH</div>

Maintenant que nous avons correctement configuré la bête reste à lui dire qu'elle peut être lancée.

{% highlight bash %}
nano /etc/default/shorewall
{% endhighlight %}


Mettre la variable startup à 1 au lieu de 0.

##### Lancement de Shorewall

Allez il est temps de le lancer:

{% highlight bash %}

shorewall start
{% endhighlight %}


La sortie devrait plus ou moins ressembler à ça :

{% highlight bash %}
Compiling...
Processing /etc/shorewall/params ...
Processing /etc/shorewall/shorewall.conf...
Compiling /etc/shorewall/zones...
Compiling /etc/shorewall/interfaces...
Determining Hosts in Zones...
Locating Action Files...
Compiling /usr/share/shorewall/action.Drop for chain Drop...
Compiling /usr/share/shorewall/action.Broadcast for chain Broadcast...
Compiling /usr/share/shorewall/action.Invalid for chain Invalid...
Compiling /usr/share/shorewall/action.NotSyn for chain NotSyn...
Compiling /usr/share/shorewall/action.Reject for chain Reject...
Compiling /etc/shorewall/policy...
Adding Anti-smurf Rules
Adding rules for DHCP
Compiling TCP Flags filtering...
Compiling Kernel Route Filtering...
Compiling Martian Logging...
Compiling Accept Source Routing...
Compiling MAC Filtration -- Phase 1...
Compiling /etc/shorewall/rules...
Compiling MAC Filtration -- Phase 2...
Applying Policies...
Generating Rule Matrix...
Creating iptables-restore input...
Shorewall configuration compiled to /var/lib/shorewall/.start
Starting Shorewall....
Initializing...
Setting up Route Filtering...
Setting up Martian Logging...
Setting up Accept Source Routing...
Setting up Traffic Control...
Preparing iptables-restore input...
Running /sbin/iptables-restore...
done.
{% endhighlight %}


Si ce n'est pas le cas et qu'une erreur apparait c'est que vous avez un problème de configuration.