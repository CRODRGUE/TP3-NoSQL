
# Atelier Elasticsearch & Kibana

## Installation et Configuration du Cluster Elasticsearch

#### Choix de l’environnement 

Le parti-pris pour mettre en place le cluster Redis, a été la conteneurisation de celui-ci avec l’utilisation de Docker. Ce choix s’explique par le fait que cette approche offre une gestion plus flexible des ressources, une isolation accrue et facilite le déploiement sur des environnements différents, donc utilisable avec environnements Windows ou bien Linux…

[lien documentation configuration kibana](https://www.elastic.co/guide/en/kibana/current/docker.html)
[lien documentation configuration elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/docker.html)

#### Architecture du cluster 

Le cluster est composé de trois services : es01, es02 et es03. C'est trois nœuds forme un cluster avec es01 configuré comme nœud maître puis les nœuds es02 et es03 comme nœud de données qui peuvent être élu maître en cas de problème avec le nœud es01. Les services tournent sur un réseau Docker nommé "lan_service". Une persistance des données à était mise en place puisque chaque servie possède un volume. Les ports par défauts d'elasticsearch sont utilisés (9200 pour les données & 9300 pour le cluster). 

Exemple d'un service :

```yml
 es01:
    // image elasticsearche
    image: docker.elastic.co/elasticsearch/elasticsearch:7.12.1
    container_name: es01
    environment:
      // Nom cluster
      - cluster.name=es-cluster
      // Nom nœud
      - node.name=es01
      // Nom des nœuds du cluster
      - cluster.initial_master_nodes=es01,es02,es03 
      // Liste des nœuds suppléant pour le rôle de maître
      - discovery.seed_hosts=es02,es03 
      // Limitation de l'utilisation de la RAM
      - bootstrap.memory_lock=true 
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" 
    volumes:
      - esdata01:/usr/share/elasticsearch/data
    networks:
      - lan_services
```

#### Comment démarrer le cluster avec Docker ?

1. Ouvrir un terminal sur la machine, puis cloner le dépôt grâce à la commande suivante :

``` bash
git clone https://github.com/CRODRGUE/TP3-NoSQL.git
```

2. Se déplacer dans le dossier nommé "TP3-NoSQL", Grâce à la commande :

``` bash
cd TP3-NoSQL/
```
3. Démarrer les services en utilisant docker compose pour effectuer cela, il faut exécuter la commande suivante qui permet de lancer les services en arrière-plan :

``` bash
docker-compose up -d 
```

## Premiers Pas avec le Cluster Elasticsearch

Cette section est consacrée à la manipulation du cluster Elasticsearch, c’est-à-dire d’effectuer les opérations de bases (manipulation des indexes, ajout, suppression, modification, lecture de documents…) mais également d’appréhender le mapping de données. Nous verrons à travers différentes étapes illustrées avec des exemples via l’utilisation de l’API REST de Elasticsearch. 

[lien documentation API REST](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/rest-apis.html)


#### Les différentes étapes illustrées

Pour rendre la réponse plus lisible, nous rajoutons en query string : format=json&pretty pour obtenir des réponsesnse au format JSON et mise en forme

* **Créer un index nommé 'test01'**

Pour créer, nous allons utiliser l’API en effectuant une requête à l’Endpoint racine avec la méthode PUT, voici l’exemple pour créer un index nommé *'test01'*

``` bash
curl -X PUT "localhost:9200/test01?format=json&pretty"
```
Réponse, on remarque que la création de l'index à était effectuée avec succès

``` json
 {
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test01"
}
```
* **Vérifier de la présence d'un index**

Pour vérifier la présence d'un index, nous avons plusieurs possibilités. La première possibilité ci-dessous nous permet de lister les index présents, mais également d'obtenir différentes informations sur chacun des index. Voici l'exemple ci-dessous :

 ``` bash
curl -X GET "localhost:9200/_cat/indices?format=json&pretty"
```
Réponse, nous obtenons un tableau contenant la liste des index avec les informations associées

 ``` json
[
  {
    "health" : "green",
    "status" : "open",
    "index" : "test01",
    "uuid" : "F7j9UBvUR0qqguSBmQySCg",
    "pri" : "1",
    "rep" : "1",
    "docs.count" : "0",
    "docs.deleted" : "0",
    "store.size" : "416b",
    "pri.store.size" : "208b"
  }
]
```
La deuxième solution, nous permet de vérifier la présence d'un index précis en fonction de son nom. 

``` bash
curl -I "localhost:9200/test01?format=json&pretty"
```
Réponse, on remarque que l'index est bien présent, car nous avons le code statut égal à 200

``` bash
HTTP/1.1 200 OK
content-type: application/json; charset=UTF-8
content-length: 669
```
* **Ajout d'un document à un index**

Pour ajouter un document à un index existant, il suffit d'émettre une requête à l'Endpoint "/:NomIndex/_doc/:IdDocument" avec la méthode POST pour indiquer le souhait de l'ajout d'un document. Voici un exemple :

``` bash
curl -X POST "localhost:9200/test01/_doc/1?format=json&pretty" \
-H 'Content-Type: application/json' \
-d @- <<EOF
{
  "titre": "Premier document",
  "description": "C'est la première fois que je crée un document dans ES.",
  "quantité": 12,
  "date_creation": "10-05-2024"
}
EOF
```
Réponse, la réponse nous retourne plusieurs informations liées au document. Les informations importantes sont les champs *"_id"* qui nous donne l'identifiant du document et le champ *"result"* qui nous informe sur l'état de l'ajout  
``` json
{
  "_index" : "test01",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

* **Afficher un document**

Pour afficher un document contenu lié à un index, il suffit d'utiliser l'Endpoint suivant : *"/:NomIndex/_doc/:IdDocument"* avec la méthode *GET*. Voici l'exemple ci-dessous :

 ``` bash
curl -X GET "localhost:9200/test01/_doc/1?format=json&pretty"
```
Réponse, nous obtenons une multitude d'infos sur le document, les données sont présent dans le champs *"_source"*
``` json
{
  "_index" : "test01",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "titre" : "Premier document",
    "description" : "C'est la première fois que je crée un document dans ES.",
    "quantité" : 12,
    "date_creation" : "10-05-2024"
  }
}
```

* **Récupérer les informations du mapping d'un index**

Pour récupérer les informations liées au mapping appliqué à un index, il faut utiliser l'Endpoint *"/:NomIndex/_mapping"* avec la méthode *GET*. Ci-dessous un exemple avec l'index "test01" :

 ``` bash
curl -X GET "localhost:9200/test01/_mapping?format=json&pretty"
```
Réponse, nous observons dans le champ *"properties"* la liste des champs présents dans notre document avec les informations du mapping liées à chacune d'entre elles avec les infos sur leur type, mais également sur les paramètres du mapping appliqués au champ

[lien information type](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/mapping-types.html) & [lien information mapping](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/mapping-params.html)

 ``` json
{
  "test01" : {
    "mappings" : {
      "properties" : {
        "date_creation" : {
          // type de la donnée stockée
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              // ignore les chaînes de caractère supérieur à 256 (pas de stockage ni d'indexage de celle-ci)
              "ignore_above" : 256 
            }
          }
        },
        "description" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "quantité" : {
          "type" : "long"
        },
        "titre" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```

* **Modifier le mapping d'un index existant**

Elasticsearch, ne permet pas de modifier les paramètres de mapping d'un index existant pour éviter de rendre invalides les données déjà présentes. Il y a donc deux solutions : Première solution de crier un nouvel index avec le mapping désiré, puis de réindexer les données présentes dans l'index que nous souhaitons modifier dans le nouvel index crée. Deuxième solution supprimée l'index puis recréer un index avec le mapping désiré (perte des données stockées) puis ajouter les prochaines données. Voici un exemple de la deuxieme solution :

Suppression de l'index nommé *"test01"*

 ``` bash
curl -X DELETE "localhost:9200/test01?format=json&pretty"
```
Réponse
 ``` json
{
  "acknowledged" : true
}
```
Création de l'index test01 avec un mapping adapté au champs *"date_creation"* avec le type *"date"* et le format de date adapté à la région française *"dd-MM-yyyy"*, contrairement au type par défaut *"text"*

``` bash
curl -X PUT "localhost:9200/test01?format=json&pretty" \
-H 'Content-Type: application/json' \
-d @- <<EOF
{
  "mappings": {
    "properties": {
      "date_creation": {
        "type": "date",
        "format": "dd-MM-yyyy"
      }
    }
  }
}
EOF
```
Réponse
``` json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test01"
}
```

* **Création d'un index avec un mapping défini**

Pour créer un index avec un mapping défini, il suffit d'utiliser l'Endpoint *"/:NomIndex"* avec la méthode *"PUT"* puis enfin les informations liées au mapping des champs dans le corps de la requête. Voici un exemple pour l'index *"test02"* :

 ``` bash
curl -X PUT "localhost:9200/test02?format=json&pretty" \
-H 'Content-Type: application/json'  \
-d @- <<EOF
{
  "mappings": {
    "properties": {
      "titre": { "type": "text" },
      "description": { "type": "text" },
      "quantité": { "type": "integer" },
      "date_creation": {
        "type": "date",
        "format": "dd-MM-yyyy"
      }
    }
  }
}
EOF
```
Réponse,
``` json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test02"
}
```
Ajout d'un document à l'index nommé "test02"
``` bash
curl -X POST "localhost:9200/test02/_doc/1?format=json&pretty" \
-H 'Content-Type: application/json' \
-d @- <<EOF
{
  "titre": "Premier document",
  "description": "C'est la première fois que je crée un document dans ES.",
  "quantité": 12,
  "date_creation": "10-05-2024"
}
EOF
```

* **Information sur le traitement d'une ressource par l'analyzer**

Pour obtenir des informations sur le traitement effectué par l'analyzer du moteur, il faut utiliser l'Endpoint *"/:NomIndex/_analyze" avec la méthode GET, puis joindre dans le corps de la requête les informations sur la donnée désirée. Voici un exemple avec la description de l'index *"test02"*

``` bash
curl -X GET "localhost:9200/test02/_analyze?format=json&pretty" \
-H 'Content-Type: application/json' \
-d @- <<EOF
{
  "field": "description",
  "text": "C'est la première fois que je crée un document dans ES."
}
EOF
```
Réponse, on peut remarquer que le texte de la description est analysé mot par mot. Voici la réponse : 
``` bash
{
  "tokens" : [
    {
      "token" : "c'est",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "la",
      "start_offset" : 6,
      "end_offset" : 8,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "première",
      "start_offset" : 9,
      "end_offset" : 17,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "fois",
      "start_offset" : 18,
      "end_offset" : 22,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "que",
      "start_offset" : 23,
      "end_offset" : 26,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "je",
      "start_offset" : 27,
      "end_offset" : 29,
      "type" : "<ALPHANUM>",
      "position" : 5
    },
    {
      "token" : "crée",
      "start_offset" : 30,
      "end_offset" : 34,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "un",
      "start_offset" : 35,
      "end_offset" : 37,
      "type" : "<ALPHANUM>",
      "position" : 7
    },
    {
      "token" : "document",
      "start_offset" : 38,
      "end_offset" : 46,
      "type" : "<ALPHANUM>",
      "position" : 8
    },
    {
      "token" : "dans",
      "start_offset" : 47,
      "end_offset" : 51,
      "type" : "<ALPHANUM>",
      "position" : 9
    },
    {
      "token" : "es",
      "start_offset" : 52,
      "end_offset" : 54,
      "type" : "<ALPHANUM>",
      "position" : 10
    }
  ]
}
```

* **Création d'un index avec optimisation de l'analyzer**

Pour optimiser l'analyzer, il suffit de crier un index avec un mapping adapter à chaque champ des documents, mais également de configurer l'analyser pour chaque champ, pour effectuer cela on procédé de la même manière que pour la création d'un index avec mapping custom sauf que l'on rajoute une clé *"settings"* qui va contenir la configuration de l'analyzer. Voici un exemple pour l'index *"test03"* avec l'optimisation de l'analyzer pour le champ *"description"* :

 ``` bash
curl -X PUT "localhost:9200/test03" \
-H 'Content-Type: application/json' \
-d @- <<EOF
{
  "settings": {
    "analysis": {
      "analyzer": {
        "french_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "french_elision",
            "french_stop",
            "french_stemmer"
          ]
        }
      },
      "filter": {
        "french_elision": {
          "type": "elision",
          "articles_case": true,
          "articles": [
            "l", "m", "t", "qu", "n", "s",
            "j", "d", "c", "jusqu", "quoiqu",
            "lorsqu", "puisqu"
          ]
        },
        "french_stop": {
          "type": "stop",
          "stopwords": "_french_"
        },
        "french_stemmer": {
          "type": "stemmer",
          "language": "light_french"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "titre": { "type": "text" },
      "description": {
        "type": "text",
        "fields": {
          "raw": {
            "type": "keyword"
          },
          "french": {
            "type": "text",
            "analyzer": "french_analyzer"
          }
        }
      },
      "quantité": { "type": "integer" },
      "date_creation": {
        "type": "date",
        "format": "dd-MM-yyyy"
      }
    }
  }
}
EOF
```

Explication des filtres :

- Le filtre *"french_elision"*, permet de supprimer toutes les élisions joint à la liste
- Le filtre *"french_stop"*, permet de supprimer les mots couramment utilisés qui n'apporte pas de sens comme par exemple les articles : la, le, les...
- Le filtre *"french_stemmer"*, permet de récupère uniquement la racine de chaque mot donc de ne pas prendre en compte la conjugaison

[Lien blog optimisation de l'analyzer](https://jolicode.com/blog/construire-un-bon-analyzer-francais-pour-elasticsearch)

* **Ajout de multiples documents à un index**

Pour ajouter plusieurs documents à un index, il faut utiliser l'Endpoint "/:NomIndex/_bulk" avec la méthode POST avec les documents joint dans le corps de la requête. *Note : il faut des "\n" pour éviter les erreurs !* Voici un exemple pour l'ajout de 4 documents à l'index *"test03"* :

``` bash
curl -X POST "localhost:9200/test03/_bulk?format=json&pretty" \
-H 'Content-Type: application/json' \
-d $'{ "index": { "_id": "1" } }\n
{ "titre": "Premier  document", "description": "Description du premier  document.", "quantité": 1, "date_creation": "20-05-2024" }\n
{ "index": { "_id": "2" } }\n
{ "titre": "Deuxième document", "description": "Description du deuxième document.", "quantité": 2, "date_creation": "20-05-2024" }\n{ "index": { "_id": "3" } }\n
{ "titre": "Troisième document", "description": "Description du troisième document.", "quantité": 3, "date_creation": "20-05-2024" }\n
{ "index": { "_id": "4" } }\n
{ "titre": "Quatrième document", "description": "Description du quatrième document.", "quantité": 4, "date_creation": "20-05-2024" }\n'
```

Réponse, elle nous indique les informations liées à l'ajout de chaque document

``` json
{
  "took" : 75,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "test03", 
        "_type" : "_doc",    
        "_id" : "2",
        "_version" : 1,      
        "result" : "created",
        "_shards" : {        
          "total" : 2,       
          "successful" : 2,  
          "failed" : 0       
        },
        "_seq_no" : 0,       
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "test03",
        "_type" : "_doc",
        "_id" : "3",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "test03",
        "_type" : "_doc",
        "_id" : "4",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "test03",
        "_type" : "_doc",
        "_id" : "5",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 3,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}
```

 