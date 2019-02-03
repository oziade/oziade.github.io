### 1. Préparation de votre environnement
Pour réaliser les différentes étapes de ce TD, vous avez besoin d'installer Elasticsearch et Kibana.  
Plusieurs choix s'offrent à vous :

* Installer un Elasticsearch et Kibana sur votre poste de travail :
    - Installer Java 8
    - Télécharger Elasticsearch et Kibana dans un répertoire
    - Dézipper Elasticsearch et Kibana, vous devez avoir 2 répertoires Elasticsearch et Kibana
    - Démarrer Elasticsearch avec la commande `{elasticsearch_directory}/bin/elasticsearch ou `{elasticsearch_directory}/bin/elasticsearch.bat` (windows)
    - Démarrer Kibana avec la commande `{kibana_directory}/bin/kibana` ou `{kibana_directory}/bin/kibana.bat` (windows).  
Vous pouvez ensuite accéder à Elasticsearch sur [http://localhost:9200](http://localhost:9200/) et Kibana sur [http://localhost:5601](http://localhost:5601).

* Utiliser Elasticsearch via [l'image Docker officielle](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html).  
Vous pouvez ensuite accéder à Elasticsearch sur [http://localhost:9200/](http://localhost:9200) et à Kibana sur [http://localhost:5601](http://localhost:5601).
    
 ---