---
layout: post
title: Comment monitorer son appli en deux soirée
category: blog
tags: Wizacha Monitoring Graphite
author: guillaume
banner: algolia-night.png
---
Un grand homme a dit, un jour : «On ne peut améliorer que ce qu'on mesure». 

Cette phrase étant pleine de bon sens et notre site {Wizacha} commençant à absorber un traffic de plus
en plus important (entre les appels API, les imports journaliers de CSV et les front-end, on atteint
les XXX requetes/heures) il n'était plus possible de surveiller manuellement le site. De plus, lorsque des 
probléme apparaissent, à partir d'un certain volume de données, il n'est plus possible de faire des 
corrélations entre plusieurs évenement si tout ce qu'on a sous la main sont des logs apache (c'est
particulieerement vrai lorsqu'un grand nombre d'actions est mis en file et où l'instant de la requete 
n'est pas le même que l'instant de l'action réelle).

#Le choix technique
Après avoir cherché ce qu'il se fait ailleurs, et parce que nous avons assisté à plusieurs conférences
abordant le sujet, notre choix s'est assez naturellement porté sur le couple «Graphite-Statsd».
Choix d'autant plus évident qu'il existe déjà des classes PHP pour gérer l'envoi des données
vers statsd. Tout ce qui n'est pas à écrire soit même est autant de temps passer à travailler sur
autre chose ! 

#La mise en oeuvre des choix
Premiere question : *Comment on installe graphite et statsd ?*.
Le choix le plus simple a été de 
récuperer une image docker déjà existance sur [github](lien qu iva bien), et de la lancer sur la
machine qui récupere déjà nos logs.

Seconde question : *Maintenant qu'on a un serveur graphite qui tourne, comment on consulte son 
contenu ?*.
Comme on utilise déjà [Kibana]() pour nos logs, notre choix s'est porté sur [Grafana]() qui offre une
interface assez similaire (les deux reposent sur [brique logicielle qui va bien]()). L'installation se
fait en dezippant le contenu d'un dossier et en configurant deux lignes dans un fichier `config.js`. 
C'est le navigateur internet qui fait tout le boulot dérriere.

Troisiéme question : *Et l'envoi des données depuis notre application vers statsd ?*. Remercions [M6Web]()
 pour leur bundle [bundle qui va bien](utl qui va bien) qui nous fournit une classe toute prête et simpe à configurer.

##Les details de la mise en oeuvre
Parce que rien n'est jamais aussi simple que lors de la démonstration.

###Docker, graphite et statsd
Comme dit plus haut, pour faire vite, nous avons utilisé une image docker déjà  disponible. On a rapidement
eu des resultats qui nous semblait pas complétement cohérent, des metriques qui affichait une valeur pendant dix secondes puis qui disparaissaient. Bref, il fallait mettre un peu les mains dans le cambouis.

Premiere étape, accéder à l'interieur de l'image : 

###Coté php
Le code fournit par le bundle est fait pour s'integrer proprement à Symfony2. Comme nous n'avons pas encore la chance d'utiliser un framework complet, on a juste créer une classe qui hérite de `le nom de la classe` et
on a juste rajouté un destructeur pour garantir que les données recoltées dans le script seront bien envoyé
à notre serveur.

On a aussi créé un petit objet `timer` [^code_timer] qui monitore le temps entre sa création et sa destruction.
Ca permet de mettre un ` $timer = new \Wizacha\Monitoring\Timer([__FUNCTION__]); ` tout en haut d'une fonction
pour avoir son temps d'execution. C'est très pratique.
