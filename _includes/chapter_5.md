### 5. Analyse d'opérations bancaires

Vous travaillez actuellement sur l'application de gestion des comptes bancaires de la banque X-Banque. 
Les mouvements sont indexés dans `Elasticsearch` avec le format suivant  : 

* __amount__ : Le montant de l'opération
* __operationDate__ : Date/heure de l'opération
* __type__ : type d'opération, 2 valeurs possibles : credit et debit
* __userId__ : l'identifiant du propriétaire du compte 
    
Voici un exemple : 
{% highlight json %}   
{
  "amount": 914,
  "operationDate": "2017-03-31T17:18",
  "type": "debit",
  "userId": "richard.serge"
}{% endhighlight %}  


L'objectif de ce chapitre est de construire pas à pas des requêtes d'agrégations complexes afin de 
pouvoir extraire de l'information de ces opérations.

---
__5.1 Indexer les documents__  
Pour indexer tous ces documents en une étape, vous allez utiliser curl :  

 * Télécharger le dataset [operations.data](data/operations.data)
 * Exécuter une requête bulk indexing :  
 
`curl -XPUT http://{host:port}/bank-account/_bulk --data-binary @operations.data -H 'Content-Type: application/json'`
  
 __Vérifier que les 124 documents sont correctements indexés :__  
 __GET__ bank-account/_count
   
---                                     
__5.2 Simple agrégation : Opérations par mois__  
Afin de remonter le nombre d'opérations par mois, écrire de même que pour l'exercice [3.11](#3.11), une agrégation mais cette fois 
de type **date_histogram**.  
**Aidez vous de l'auto-complétion des devs tools pour déterminer les paramètres de cette agrégation.**

<blockquote class = 'solution' markdown="1">

GET bank-account/_search
{% highlight json %}   
{
  "size": 0,
  "aggs": {
    "by_month": {
      "date_histogram": {
        "field": "operationDate",
        "interval": "month"
      }
    }
  }
}
{% endhighlight %}
</blockquote>

---    
__5.3 Sous agrégation : opérations par mois et par userId__  
Afin de remonter les opérations par mois de chaque compte (userId), ajouter à l'agrégation précédente une sous-agrégation (format exercice [3.12](#3.12)) de type **term** qui cible 
la valeur "non analysée" du champ userId.

<blockquote class = 'solution' markdown="1">

GET bank-account/_search
{% highlight json %}   
{
  "size": 0,
  "aggs": {
    "by_month": {
      "date_histogram": {
        "field": "operationDate",
        "interval": "month"
      },
      "aggs": {
        "by_user_id": {
          "terms": {
            "field": "userId.keyword"
          }
        }
      }
    }
  }
}
{% endhighlight %}
</blockquote>

---    
__5.4 Filtre sur agrégation : agréger uniquement les crédits__  
L'objectif est de construire une requête permettant de détecter des montants trop importants reçus. Il ne faut donc 
garder que les opérations dont le type est **credit**.  
Ajouter à l'agrégation précédente une **query** avec un filtre sur ce champ afin de remonter les débits par compte et par mois.

<blockquote class = 'solution' markdown="1">

GET bank-account/_search
{% highlight json %}   
{
  "size": 0,
  "query": {
    "bool": {
      "filter": {
        "term": {
          "type.keyword": "credit"
        }
      }
    }
  },
  "aggs": {
    "by_month": {
      "date_histogram": {
        "field": "operationDate",
        "interval": "month"
      },
      "aggs": {
        "by_user_id": {
          "terms": {
            "field": "userId.keyword"
          }
        }
      }
    }
  }
}
{% endhighlight %}
</blockquote>

--- 
__5.5 Sum agregation : Crédit total par mois et par compte (userId)__  
L'agrégation précédente permet de remonter le nombre d'opération de crédit par mois pour chaque userId. Ajouter à cette agrégation une sous-agrégation
de type **sum** afin de remonter la somme de tous les montants par mois et par compte.    

<blockquote class = 'solution' markdown="1">

GET bank-account/_search
{% highlight json %}   
{
  "size": 0,
  "query": {
    "bool": {
      "filter": {
        "term": {
          "type.keyword": "credit"
        }
      }
    }
  },
  "aggs": {
    "by_month": {
      "date_histogram": {
        "field": "operationDate",
        "interval": "month"
      },
      "aggs": {
        "by_user_id": {
          "terms": {
            "field": "userId.keyword"
          }
          ,"aggs": {
            "amount_sum": {
              "sum": {
                "field": "amount"
              }
            }
          }
        }
      }
    }
  }
}
{% endhighlight %}
</blockquote>

--- 
__5.6 Pipeline bucket selector agregation : Ne remonter que certaines agrégations__  
La requête précédente remonte un grand nombre de résultats. Hors nous souhaitons pouvoir remonter uniquement les agrégations dont la somme par mois et par compte
est supérieur à 5000 euros.  
Pour cela ajouter à la dernière sous-agrégation une **pipeline agrégation** de type **bucket_selector**.  
   __Exemple  de bucket selector agrégation pour remonter les buckets contenant un prix égal à 1000:__  
{% highlight json %}      
 {
  ...
   "aggs" : {
     "by_price" :{
       "sum": {
         "field": "price"
       }
     },
      "my_pipeline_aggregation_selector" :{
         "bucket_selector" : {
           "buckets_path": {
             "by_price": "my_var"
           },
           "script": "params.myVar = 1000"
         }
       }
   }
 }
 {% endhighlight %}  
 
 
<blockquote class = 'solution' markdown="1">

GET bank-account/_search
{% highlight json %}   
{
  "size": 0,
  "query": {
    "bool": {
      "filter": {
        "term": {
          "type.keyword": "credit"
        }
      }
    }
  },
  "aggs": {
    "by_month": {
      "date_histogram": {
        "field": "operationDate",
        "interval": "month"
      },
      "aggs": {
        "by_user_id": {
          "terms": {
            "field": "userId.keyword"
          },
          "aggs": {
            "amount_sum": {
              "sum": {
                "field": "amount"
              }
            },
            "sum_selector": {
              "bucket_selector": {
                "buckets_path": {
                  "amount_sum": "amount_sum"
                },
                "script": "params.amount_sum > 5000"
              }
            }
          }
        }
      }
    }
  }
}
{% endhighlight %}
</blockquote>

---         