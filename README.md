# Pipeline d'ingestion de donnes depuis une-machine-locale-vers-le-stockage-GCP



## 1. Création de Pub/Sub sur GCP

Pour créer un sujet Pub/Sub et publier des messages (unités de données), vous devez vous authentifier sur GCP, créer un compte de service, télécharger la clé associée, et configurer le SDK et la CLI. 

Par la suite, il faut créer un topic pour communiquer des messages sur Pub/Sub. Nous créons un sujet nommé "gcp-yelp-bigdata". Une fois le sujet créé, une souscription (gcp-yelp-bigdata-sub) est également générée. Cette souscription permet à une application de recevoir et traiter les messages provenant d'un sujet Pub/Sub spécifique.

![image](https://github.com/user-attachments/assets/fbe2c707-d20a-4040-bc07-4ac1e6da0fd4)

## 2. Configuration et activation des services accounts gcp

```[gcp]
project_id =fabric-401015 #project name
topic_id = gcp-yelp-bigdata #topicid
credentials_path = /home/../Data Engineer/GCP/fabric-401015-0990a16be511.json #keyforserviceaccount
file_path = /home/nouro/.. Engineer/GCP/yelp_covid_19/yelp_academic_dataset_covid_features.json #datatotransfert
## activating services
gcloud auth activate-service-account svc2-rw@fabric-401015.iam.gserviceaccount.com --key-file=/home/xx/Data\ Engineer/GCP/fabric-401015-0990a16be511.json --project=fabric-401015
```

## 3. Publier les données (message) sur Pub/Sub

Nous allons utiliser le code python (publish_messages.py) pour transferer les données sur le topic Pub Sub
```
python3.7 scripts/publish_messages.py --config_path=config/publish_config.ini

```
Une fois l'exécution terminée, nous pouvons aller à la suscription pour voir les données stockées dans Pub/Sub. Le "message body" contient les données qui seront utilisées par le subscriber (Dataflow)pour être stockées dans BigQuery.
![image](https://github.com/user-attachments/assets/55e267c1-4d01-4a25-9e0c-d307a9aa2571)


## 4. Création de jobs Dataflow en streaming

Nous allons utiliser le code python dans le fichier "dataflow_stream.py". Dans ce dernier, nous allons spécifier le *SCHEMA* pour la table de sortie dans BigQuery. Nous avons également créé le pipeline en utilisant Apache Beam et une fonction ParseMessage pour le traitement des données des messages dans Pub/Sub. 

```
python3 /scripts/dataflow_stream.py \
--input_subscription=projects/fabric-401015/subscriptions/gcp-yelp-bigdata-sub \
--output_table=test.yelp_covid --output_error_table=test.error --runner DataflowRunner \
--project fabric-401015 --region us-west1 --service_account_email svc2-rw@fabric-401015.iam.gserviceaccount.com \
--staging_location gs://gcs-bigdata-bucketbis/dataflow/staging --temp_location gs://gcs-bigdata-bucketbis/dataflow/temp \
--job_name test-stream-bq --num_workers 1 --max_num_workers 2

```


Une fois ce job (pipeline) lancé, nos messages ou données seront traités et stockés dans BigQuery. Voici un exemple de requêtes sur la table yield_covid creée sur BigQuery.

![image](https://github.com/user-attachments/assets/f82a9617-af06-4c60-9d41-ec0ced768c0a)










