---
published: true
title: "Architectures scalables avec les solutions Apache"
excerpt: "Cassandra, Kafka, Spark, Elastic"
toc: true
toc_sticky: true
toc_label: "Apache Scalable"
toc_icon: "align-center"
comments: true
author_profile: false
header:
  overlay_image: "assets/images/covers/cover2.jpg"
  teaser: "assets/images/covers/cover2.jpg"
categories: [kafka, cassandra, elastic, spark]
---

D'abord quelques clarifications :
- La licence **Apache** est une licence de logiciel libre et open source, elle permet l'utilisation et la distribution du code sous toutes formes (y compris commercial). Cela implique que le code source est public, et que les modifications sont soumises à l'approbation générale.
- On parle de technologie **scalable** quand on a besoin de faire grossir la charge d'une application : Pour une base de donnée, on veut que les performances soient constantes en lecture/écriture quand le volume de données augmente. Idem pour le traitement de flux de données avec les quantités de données traitées.

<div align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/db/Apache_Software_Foundation_Logo_%282016%29.svg/1200px-Apache_Software_Foundation_Logo_%282016%29.svg.png" alt="Kafka APIs">
</div>

Les technologies open-source Apache sont présentes dans la quasi-totalité des architectures distribuées. Avec plus de <a href="https://streamdata.io/blog/open-source-apache-big-data-projects/" target="_blank">30 projets</a> actifs rien que pour les problématiques data, Apache est un incontournable des solutions data. 

Dans cet article je détaille quatre solutions open-source très répandues, rapides, et scalables. Pour chacune de ces technos, les use-cases les plus adaptés. Enfin nous verrons 3 architectures mettant en oeuvre ces solutions.

# Apache Cassandra

Cassandra a été développé par Facebook et rendu open-source en 2008. C'est un SGBD de type NoSQL qui répond aux contraintes suivantes :
- Stocker un volume massif de données
- Assurer la haute disponibilité des données
- Répartir les données sur plusieurs noeuds
- Éviter les 'single point of failure'

Cassandra peut être vu comme une table de hashage grand volumes. Chaque donnée stockée dans le système possède un identifiant unique. C'est grâce à une table de correspondance de chacun de ces identifiants que les données sont réparties à travers les différents serveurs : il existe plusieurs mécanismes de répartition, et de réplication.

## Quand l'utiliser

- Transactionnel
- Stockage Cross-DC
- Forte contraintes de disponibilité, SLA 99.999%

## Quand ne pas l'utiliser

- Analytics & Data warehouse.
- Applications temps réel et visualisation AdHoc.
- Ne respecte pas ACID, notamment l'isolation.
- Ne pas utiliser Cassandra comme une file d'attente.

# Apache Kafka

<div align="center">
  <img src="https://kafka.apache.org/images/logo.png" alt="Kafka Logo" vspace="30">
</div>


Kafka est une solution très populaire pour la **gestion des files de messages**. C'est un système qui centralise les flux, un Hub. Comme pour les aéroports, le traffic est centralisé et géré efficacement. Le fonctionnement de Kafka est articulé autour de plusieurs éléments : 
- Les **topics**, qui sont des files de messages, où les messages s'accumulent.
- Les **producers** qui produisent des messages et les stockent dans un ou plusieurs topics.
- Les **consumers** qui consomment les messages d'un ou plusieurs topics.

<div align="center">
  <img src="https://kafka.apache.org/0110/images/kafka-apis.png" alt="Kafka APIs">
</div>

## Quand l'utiliser

- Micro-services
- Message bus, metrics
- Data masking, filtering avec Kafka streams (tâches de transformation)

## Quand ne pas l'utiliser

- Ne pas utiliser Kafka comme une source de données.
- Traiter toutes les données séquentiellement depuis un topic.
- Les cas de contraintes temps réel ne sont pas adaptés.

# Apache Spark

<div align="center">
  <img src="https://blog.jetoile.fr/images/rdd/spark.png" alt="Apache Spark">
</div>


Apache Spark propose une alternative plus performante au paradigme MapReduce. Spark est capable de s'executer selon différents schemas :
- En mode **stand-alone**, en local sur un laptop par exemple.
- **YARN**, le négociateur de ressources du framework Hadoop.
- **Mesos**, le scheduler pour gérer les ressources d'un datacenter (similaire à YARN mais cadre plus large).
- **Kubernetes**, déploiement automatisé d'application en containers.

## Quand l'utiliser

- Quand on souhaite travailler sur de **gros volumes de données**. Spark n'est pas adapté aux traitements de faible volumes, en effet, les jobs ont besoin de temps pour être déployés et pour se lancer. Une fois passé ce temps de lancement, la valeur ajoutée de Spark augmente avec le volume de données traitées.
- Pour les tâches dites d'**ETL (Extract, Transform, Load)** : Spark est idéal pour créer des pipelines des données, dans lesquels on peut intégrer des opérations de transformations complexes sur le schema de données, des jointures.
- Faire des opérations AdHoc et des requêtes exploratoires complexes.

## Quand ne pas l'utiliser

Spark ne convient pas pour des opérations en temps réel, en raison du temps de mise en route élevé des jobs Spark.

# Elasticsearch (Apache Lucene)

ElasticSearch est open source, et il est basé sur la solution Apache Lucene. Elasticsearch est aujourd'hui très répandu dans les contextes big data car il propose des temps de réponse très faibles pour explorer les données en mode 'moteur de recherche'.

A l'intérieur d'Elasticsearch les données sont réparties sous forme de **documents** dans des **index**. A l'intérieur d'un index, chacun des documents possèdent les mêmes champs.

La solution de visualisation **Kibana** a beaucoup évolué depuis les premières versions et propose un dashboard très complet :

<div align="center">
  <img src="https://d2.alternativeto.net/dist/s/https--www-elastic-co-products-kibana_989074_full.jpg?format=jpg&width=1600&height=1600&mode=min&upscale=false" alt="Kibana Dashboard">
</div>

## Quand l'utiliser

- Applications de recherche dans des données full-text
- Très pratique pour les GeoData
- Agréger et croiser différentes sources de données.

## Quand ne pas l'utiliser

- On n'utilise pas Elasticsearch comme une base de données. On lui adresse des **recherches**, qui sont différentes des requêtes en bases de données. Une recherche ne renvoie pas un résultat exacte, mais s'effectue beaucoup plus rapidement qu'une requête structurée dans une base de données.
- Comme source de données, Elasticsearch ne convient pas. Comme vu au premier point, on ne peut pas lui adresser de requêtes, ainsi les données obtenues ne respectent pas les contraintes que l'on peut attendre d'une BDD classique.

# Exemples d'architecture

## Microservices

Pour les micro services, Kafka est l'outil de prédilection, il centralise les flux grâce à ses nombreux connecteurs. Les demandes utilisateur peuvent être regroupées en topics afin de consommer les tâches au rythme le plus adapté et de répartir la charge.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/architectures/kafka-microservices.png" alt="" class="center">

## ETL

Les données sont réparties dans divers systèmes de stockage selon les besoins et les usages qui en sont fait. Ici on se sert essentiellement de Spark pour des opérations ETL de migration ou d'enrichissement entre ces espaces de stockage.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/architectures/spark-etl.png" alt="" class="center">

## Objets connectés

Les données collectées sont rassemblées dans Kafka, qui répartit les flux vers chaque système approprié pour le traitement.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/architectures/kafka-iot.png" alt="" class="center">

# Sources
- Salon Big Data Paris 11 Mars 2019 - Ateliers sur les solutions Apache scalables.
