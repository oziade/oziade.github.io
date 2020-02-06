### 2. Découverte de l'API


   __2.1 Premier document indexé__    
__POST__ programmer/_create/1
{% highlight json %}
{
    "name": "Lovelace",
    "firstname": "Ada",
    "birthday" : "1815-12-10"
}
{% endhighlight %}   
  
---    
     
 - **programmer** est le nom de l'index   
 - **1** est l'id  
    
---  
           
   __2.2 Retrouver le document par son id__
__GET__ programmer/_doc/1  
    
  __2.3 Indexer d'autres documents (Avec les id 2 et 3)__  
{% highlight json %}
{
    "name": "Gosling",
    "firstname": "James",
    "birthday" : "1954-05-19"
}
{% endhighlight %}  
--- 
{% highlight json %}
{
    "name": "Berners-Lee",
    "firstname": "Tim",
    "birthday" : "1955-06-08"
}
{% endhighlight %}
---
  __2.4 Recherche sans critère__  
__GET__ programmer/_search

  __2.5 Recherche full text <a name="2.5"></a>__  
__GET__ programmer/_search
{% highlight json %}
{
    "query": {
        "match": {
            "name": "lee"
        }
    }
}
{% endhighlight %}
---
  __2.6 Recherche full text avec highlighting (mise en surbrillance du terme qui "match" le texte de recherche)__  
__GET__ programmer/_search
{% highlight json %}
{
    "query": {
        "match": {
            "name": "lee"
        }
    },
    "highlight": {
        "fields": {
            "name":{}
        }
    }
}
{% endhighlight %}
---    
  __2.7 Voir le mapping__  
__GET__ programmer/_mapping
  
  __2.8 Supprimer un document__  
__DELETE__ programmer/_doc/{id}
      
  __2.9 Supprimer l'index__  
__DELETE__ programmer

---