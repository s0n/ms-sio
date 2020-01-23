# ms-sio

## Cours 1: spark 

Objectifs:
    + manipuler le dsl de spark
    + comprendre le concept de RDD
    + comprendre le modele de memoire
    + voir comment configurer un job spark 
    + voir comment lancer un job spark 

- preparer le dataset:

télécharger le dataset via cette page: https://www.kaggle.com/datasnaek/youtube-new

le fichier zip télécharger doit etre a ce path : `~/Downloads/youtube-new.zip`.

```bash
make prepare-dataset
``` 

- lancer pyspark :

```bash
make run-pyspark
``` 
- je charge mon dataset

```python
videos = spark.read.option('header','true').option('inferSchema','true').csv('/data/raw/FRvideos.csv')
```
- je decouvre mon dataset

```python
videos.count()

videos.show()

videos.show(50)

videos.show(10,False)

```

```python 
videos.printSchema()
```
- chercher et corriger les erreurs d'inference de schema:


```python
videos.select('category_id').show()
videos.select('category_id').sample(0.1).distinct().show()
videos.select('category_id').filter(videos.category_id.rlike('\\d*')).count()

videos = videos.filter(videos.category_id.rlike('\\d*')).withColumn('category_id',videos.category_id.cast('integer'))
```

- lire le dataset en json

```python
categories = spark.read.option('multiline','true').json('/data/raw/FR_category_id.json')

categories.count()

categories.show()

categories.printSchema()

```

- je transforme les données pour qu'elles soient plus manipuable:

```python
from pyspark.sql.functions import *

categories = categories.select(explode('items'))

categories = categories.select('col.id','col.snippet.title')

categories = categories.withColumnRenamed('title','category_title')
```

```python

df = videos.join(categories.hint('broadcast'),videos.category_id == categories.id,'inner')

```

__note__ : voir par [ici](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#) pour la syntaxe


### comprendre la gestion de la mémoire:

- sur un navigateur ouvrir l'inteface spark UI http://localhost:4040



```python
 df.cache() 
 df.count() # action pour forcer la mise en cache

```
 
 
- voir l'onglet storage.

- autre facon de mettre en cache:

```python
df.persist(pyspark.StorageLevel.MEMORY_ONLY) # voir https://spark.apache.org/docs/latest/rdd-programming-guide.html#rdd-persistence

```
 
- faire exploser la memoire:

```python 
cs = df.crossJoin(df)
cs = df.crossJoin(df.sample(0.009))
```

- Comment resoudre ce genre de situation?
    + Comprendre le Memory Model de spark
    
    + de quelle resources je dispose sur mon cluster
        + Memoire 
    + quelle quantité de données je vais manipuler
        + Partition
    
- Comment je vais sauvegarder mon resultat:
    + csv
    + json
    + parquet
    + avro
    + ORC
    + Protobuff (TFRecords)    
    
- Comment je lance mon job en production?


## Cours 2: streaming

definir le concept de stream dataset vs methode de processing.

Quel traitement?
A quelle periode/fenetre s'applique le traitement?
Quand est ce qu'on emet le resultat?
Comment on restitue le resultat?


ordering
windowing:
 - fixed window: simple decoupage en periode egale (sliding avec pas == periode)
 - sliding window: definie par un pas et une periode (souvent pas >= periode)
 - session: depend de la presence ou pas d'un pattern dans la donée
le windowing peut se baser sur :
 - event time
 - processing time 
 - capture time   
se pose la question de la completude de donées par rapport au frontiere des windows:
 - watermark: la limite aprés laquelle on considere/suppose que tte les données sont parvenues
 - triggering: quand on considere que les events/données relative a une window sont constituée et prete au processing
autres concepts:
 - accumulation des resultats
 

