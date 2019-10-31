## DATASET QUERY
L'intérrogation du Dataset e-commerce s'effectue au travers d'un objet (JSON) **QUERY** dont voici la définition:

```
 root
 |-- select: array (nullable = true)
 |    |-- element: string (containsNull = true)
 |-- scrollID: string (nullable = true)
 |-- index: string (nullable = true)
 |-- filters: array (nullable = true)
 |    |-- element: struct (containsNull = true)
 |    |    |-- filter: struct (nullable = true)
 |    |    |    |-- condition: struct (nullable = true)
 |    |    |    |    |-- field: string (nullable = true)
 |    |    |    |    |-- index: string (nullable = true)
 |    |    |    |    |-- operator: string (nullable = true)
 |    |    |    |    |-- value: string (nullable = true)
 |    |    |    |-- logicalOp: string (nullable = true)
 |    |    |    |-- parenthesis: string (nullable = true)
 |-- groupBy: array (nullable = true)
 |    |-- element: string (containsNull = true)
 |-- orderBy: array (nullable = true)
 |    |-- element: string (containsNull = true)
 |-- prettyPrint: boolean (nullable = true)
 |-- searchParams: struct (nullable = true)
 |    |-- nbResultsToRetrieve: long (nullable = true)
 |    |-- scrollSearchTimeout: long (nullable = true)

```
-----------------------------
### Définition des champs:
#### -- select: array (nullable = true)
Ce champ permet de définir les élements que l'on souhaite récupérer du JSON intérrogé.  
C'est un tableau qui contient la liste des noms complets (avec le path) des champs que l'on souhaite récupérer.

Ainsi pour un customer par exemple:

- customers JSON schema:
```
root
 |-- customer: struct (nullable = true)
 |    |-- civility: string (nullable = true)
 |    |-- dob: string (nullable = true)
 |    |-- id: long (nullable = true)
 |    |-- session_vids: array (nullable = true)
 |    |    |-- element: struct (containsNull = true)
 |    |    |    |-- nbVisit: long (nullable = true)
 |    |    |    |-- session_vid: string (nullable = true)
```
Le *select* suivant:
```
"select":["customer.id","customer.age"]
```
ramène pour chaque *customer* son identifiant ainsi que son âge.  


Il y a une syntaxe particulière qui permet de remonter tous les champs:
```
"select":["customer.*"]
```  
    
Via cette syntaxe il est également possible de récupérer plusieurs champs d'une "même catégorie"
Ainsi sur les *product*, le select suivant:
```
"select":["product.book*"] 
```
ramenra les champs suivants:
- product.book_collection
- product.book_editor
- product.book_isbn
- product.book_people
- product.book_people_Coloriste
- product.book_people_Dessinateur
- product.book_people_Illustrateur
- product.book_people_PrefaceWriter
- product.book_people_ScriptWriter
- product.book_people_Traducteur
- product.book_people_author
- product.book_series   

Enfin, il est possible de définir différentes agrégations:

- **```"select" : ["MAX(customer.session_vids.nbVisit) AS MaxNbVisit"]```**
   Récupère la plus grande valeur *nbVisit*. La clause **AS** (la *case* est importante) permet de définir le nom du champ qui sera remonté pour cette agrégations. C'est-à-dire ```"MaxNbVisit"``` dans l'exemple ci dessus.   
- **```"select" : ["MIN(customer.session_vids.nbVisit)"]```**
   Récupère la plus petite valeur *nbVisit*   
   Comme il n'y a pas de clause **AS**, le nom du champ remonté sera *calculé* comme suit: ```"MIN_"<nom de champ>```. Dans l'exmple ci-dessus, le nom du champ remonté sera: ```"MIN_customer.session_vids.nbVisit" ```   
  
- **```"select" : ["SUM(customer.session_vids.nbVisit) AS TotalNbVisit"]```**
   Récupère la somme de toutes les valeurs *nbVisit*   
- **```"select" : ["COUNT(customer.session_vids.nbVisit) AS NbVisit"]```**
   Récupère le nombre d'élements a somme de toutes les valeurs *nbVisit*   
- **```"select" : ["AVG(customer.session_vids.nbVisit) AS AverageNbVisit"]```**
   Récupère la moyenne des valeurs *nbVisit*   
        
  
##### Afin d'ilustrer le fonctionnement des agrégations, voici un exemple:
Sur la base des résultats suivants:
```
"customers" : [ {
    "customer" : {
      "id" : 24621,
      "session_vids" : [ {
        "session_vid" : "2b79e0f7-8460-472f-bbe2-625fc2dda82c",
        "nbVisit" : 4
      }, {
        "session_vid" : "3f53c6b2-e54b-45e0-a644-84d70472118e",
        "nbVisit" : 1
      } ],
      "civility" : "Mme",
      "dob" : "1986-09-09",
      "age" : 33
    }
  }, {
    "customer" : {
      "id" : 7343,
      "session_vids" : [ {
        "session_vid" : "e8d08aa5-264c-4be6-8971-7fa2168b8b67",
        "nbVisit" : 10
      }, {
        "session_vid" : "565eded6-0dc0-445a-b6df-308ab82abc2c",
        "nbVisit" : 9
      } ],
      "civility" : "Mme",
      "dob" : "1976-03-17",
      "age" : 43
    }
  } ],
```
le *select*:
```"select" : [ "MIN(customer.session_vids.nbVisit)", "MAX(customer.session_vids.nbVisit) AS MaxNbVisit", "SUM(customer.session_vids.nbVisit) AS TotalNbVisit", "COUNT(customer.session_vids.nbVisit) AS NbVisit", "AVG(customer.session_vids.nbVisit) AS AverageNbVisit"]```

donnera lieu à un bloc **aggregations** dans le JSON de réponse:
```
  "aggregations" : [ {
    "name" : "NbVisit",
    "value" : "4.0"
  }, {
    "name" : "MIN_customer.session_vids.nbVisit",
    "value" : "1.0"
  }, {
    "name" : "MaxNbVisit",
    "value" : "10.0"
  }, {
    "name" : "TotalNbVisit",
    "value" : "24.0"
  }, {
    "name" : "AverageNbVisit",
    "value" : "6.0"
  } ]
}
```
#### -- scrollID: string (nullable = true)
Dans des souçis de performance et au vu des volumétries de données importantes, les résultats d'une réquête sont récupérés *morceau par morceau*. La taille du nombre d'éléments que l'on souhaite récupérer (les *morceaux*) est paramètrable et s'effectue au travers du champ **nbResultsToRetrieve** de l'*objet* **searchParams** (voir plus loin).

Si suite à une requête il reste des résultats à récupérer, la réponse contiendra un élément **scrollID** (chaine de caractère)  
Exemple:
```"scrollID" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAABAWS3dYODNOZVlRb0tqMk9vanNhMTdsQQ=="```). 

Pour récupérer la suite des résultats il suffit de passer, dans la requête d'intérrogation, cet élément, conjoitement avec le champ **index** décrit ci-dessous.
Si il reste encore des résultats, un nouveau **scrollID** sera remonté et il faudra à nouveau le renvoyer dans la requête d'intérrogation pour récupérer la suite des résultats. L'opération est à répeter tant que **scrollID** n'est pas *null*, c'est-à-dire tant qu'un champ **scrollID** est présent dans le JSON de réponse.


#### -- index: string (nullable = true)
Indique l'index sur lequel le **scrollID** s'applique. Il y a 4 index:
 - 1: "visits" (visits.json dénormalisé)  
 - 2: "customers" (customers.json)
 - 3: "categories" (categories.json)
 - 4: "products" (product.json dénormalisé)


 #### -- filters: array (nullable = true)
 Ce champ permet de filtrer les données retournées. Il contient une litse de *filter*.
 Chaque *filter* contient un sous élément **condition** qui permet de spécifier les régles de filtrage à appliquer.
 
   - ##### condition:
 ```
 |-- filters: Liste de filtres (filter)
 |   -- filter: un filtre
 |      -- condition: struct (nullable = true)
           -- field: string (nullable = true)
           -- index: string (nullable = true)
           -- operator: string (nullable = true)
           -- value: string (nullable = true)
 |      -- logicalOp: string (nullable = true)
 |      -- parenthesis: string (nullable = true)
```  

###### Liste des champs de *condition*:
+ **field**: Nom complet (avec le path) du champ sur lequel s'applique la condition. 
  Exemple: ```"field" : "customer.civility"```
+ **index:** Nom de l'index auquel apprtient le champ *field*
  Exemple: ```"index" : "customers"```
+ **operator:** Opérateur de la condition sur le champ *field*.
  Les opérateurs suivants sont définis:
   + "=" : égal à
   + "!=": différent de
   + ">": strictement superieure à
   + "<": strictement inferieure à
   + ">=": supérieur et égal à
   + "<=": inférieur et égal à

  Exemple: ```"operator" : "="```

+ **value:** La valeur sur laquelle s'applique l'opérateur
  Exemple: ```"value" : "Mme"```
  Toutes les valeurs doivent être passées sous la forme d'une chaine de caratère. Elles sont converties dans leur type d'origine (Integer, Float, Double, etc) par le **service Query**.
  
  Il existe deux type de valeurs spéciales:
  + "value" : "NULL"    -- Permet de tester la nullité du champ *field*
  + "value" : "[30,40}" -- Permet de définir un *range* de valeurs. **"["** et **"]"** définissent des bornes inférieures et supèrieures **inclusives**. **"{"** et **"}"** définissent des bornes inférieures et supèrieures **exclusives**.
  
En plus de l'objet *condition*, l'objet **filter** contient deux champs supplémentaires:
+ logicalOp
+ parenthesis





