# Engineer Data in Google Cloud

https://www.qwiklabs.com/quests/132?locale=zh_TW

## Creating a Data Transformation Pipeline with Cloud Dataprep

https://www.qwiklabs.com/focuses/4415?locale=zh_TW&parent=catalog

- Connect BigQuery datasets to Cloud Dataprep.
- Explore dataset quality with Cloud Dataprep.
- Create a data transformation pipeline with Cloud Dataprep.
- Schedule transformation jobs outputs to BigQuery.

```bash
#standardSQL
 CREATE OR REPLACE TABLE ecommerce.all_sessions_raw_dataprep
 OPTIONS(
   description="Raw data from analyst team to ingest into Cloud Dataprep"
 ) AS
 SELECT * FROM `data-to-insights.ecommerce.all_sessions_raw`
 WHERE date = '20170801'; # limiting to one day of data 56k rows for this lab
```

Using a Chrome browser (Reminder: Cloud Dataprep works ONLY in Chrome), open the Navigation menu and under Big Data, select Dataprep.

## Building an IoT Analytics Pipeline on Google Cloud

- Connect and manage MQTT-based devices using Cloud IoT Core (using simulated devices)
- Ingest a stream of information from Cloud IoT Core using Cloud Pub/Sub.
- Process the IoT data using Cloud Dataflow.
- Analyze the IoT data using BigQuery.

1. Create a Cloud Pub/Sub topic
2. Create a BigQuery dataset
3. Create a cloud storage bucket
4. Set up a Cloud Dataflow Pipeline
    1. In the Cloud Console, go to Navigation menu > Dataflow.
    2. In the top menu bar, click + CREATE JOB FROM TEMPLATE.
    3. In the job-creation dialog, for Job name, enter iotlabflow.
    4. For Regional Endpoint, choose the region as us-central1.
    5. For Dataflow template, choose Pub/Sub Topic to BigQuery. When you choose this template, the form updates to review new fields below.
    6. For Input Pub/Sub topic, enter projects/ followed by your Project ID then add /topics/iotlab. The resulting string will look like this: projects/qwiklabs-gcp-d2e509fed105b3ed/topics/iotlab
    7. The BigQuery output table takes the form of Project ID:dataset.table (:iotlabdataset.sensordata). The resulting string will look like this: qwiklabs-gcp-d2e509fed105b3ed:iotlabdataset.sensordata
    8. For Temporary location, enter gs:// followed by your Cloud Storage bucket name (should be your Project ID if you followed the instructions) then /tmp/. The resulting string will look like this: gs://qwiklabs-gcp-d2e509fed105b3ed-bucket/tmp/
    9. Click SHOW OPTIONAL PARAMETERS.
    10. For Max workers, enter 2.
    11. For Machine type, enter n1-standard-1.
    12. Click RUN JOB.
5. Prepare your compute engine VM

    ```bash
    sudo pip install virtualenv
    virtualenv -p python3 venv
    source venv/bin/activate
    sudo apt-get remove google-cloud-sdk -y
    curl https://sdk.cloud.google.com | bash
    exit
    source venv/bin/activate
    gcloud init
    gcloud components update
    gcloud components install beta
    sudo apt-get update
    sudo apt-get install python-pip openssl git -y
    pip install pyjwt paho-mqtt cryptography
    git clone http://github.com/GoogleCloudPlatform/training-data-analyst
    ```

6. Create a registry for IoT devices

    ```bash
    export PROJECT_ID=
    export MY_REGION=
    gcloud beta iot registries create iotlab-registry \
        --project=$PROJECT_ID \
        --region=$MY_REGION \
        --event-notification-config=topic=projects/$PROJECT_ID/topics/iotlab
    ```

7. Create a Cryptographic Keypair

    ```bash
    cd $HOME/training-data-analyst/quests/iotlab/
    openssl req -x509 -newkey rsa:2048 -keyout rsa_private.pem \
        -nodes -out rsa_cert.pem -subj "/CN=unused"
    ```

8. Add simulated devices to the registry

    ```bash
    gcloud beta iot devices create temp-sensor-buenos-aires \
        --project=$PROJECT_ID \
        --region=$MY_REGION \
        --registry=iotlab-registry \
        --public-key path=rsa_cert.pem,type=rs256
    gcloud beta iot devices create temp-sensor-istanbul \
        --project=$PROJECT_ID \
        --region=$MY_REGION \
        --registry=iotlab-registry \
        --public-key path=rsa_cert.pem,type=rs256
    ```

9. Run simulated devices

    ```bash
    cd $HOME/training-data-analyst/quests/iotlab/
    wget https://pki.google.com/roots.pem
    python cloudiot_mqtt_example_json.py \
        --project_id=$PROJECT_ID \
        --cloud_region=$MY_REGION \
        --registry_id=iotlab-registry \
        --device_id=temp-sensor-buenos-aires \
        --private_key_file=rsa_private.pem \
        --message_type=event \
        --algorithm=RS256 > buenos-aires-log.txt 2>&1 &
    python cloudiot_mqtt_example_json.py \
        --project_id=$PROJECT_ID \
        --cloud_region=$MY_REGION \
        --registry_id=iotlab-registry \
        --device_id=temp-sensor-istanbul \
        --private_key_file=rsa_private.pem \
        --message_type=event \
        --algorithm=RS256
    ```

## ETL Processing on Google Cloud Using Dataflow and BigQuery 

- Cloud Storage
- Dataflow
- BigQuery

```bash
gsutil -m cp -R gs://spls/gsp290/dataflow-python-examples .
export PROJECT=qwiklabs-gcp-02-05597e01ce0d
gcloud config set project $PROJECT
gsutil mb -c regional -l us-central1 gs://$PROJECT
gsutil cp gs://spls/gsp290/data_files/usa_names.csv gs://$PROJECT/data_files/
gsutil cp gs://spls/gsp290/data_files/head_usa_names.csv gs://$PROJECT/data_files/
bq mk lake

cd dataflow-python-examples/
# Here we set up the python environment.
# Pip is a tool, similar to maven in the java world
sudo pip install virtualenv

#Dataflow requires python 3.7
virtualenv -p python3 venv

source venv/bin/activate
pip install apache-beam[gcp]==2.24.0

python dataflow_python_examples/data_ingestion.py --project=$PROJECT --region=us-central1 --runner=DataflowRunner --staging_location=gs://$PROJECT/test --temp_location gs://$PROJECT/test --input gs://$PROJECT/data_files/head_usa_names.csv --save_main_session
python dataflow_python_examples/data_transformation.py --project=$PROJECT --region=us-central1 --runner=DataflowRunner --staging_location=gs://$PROJECT/test --temp_location gs://$PROJECT/test --input gs://$PROJECT/data_files/head_usa_names.csv --save_main_session
python dataflow_python_examples/data_enrichment.py --project=$PROJECT --region=us-central1 --runner=DataflowRunner --staging_location=gs://$PROJECT/test --temp_location gs://$PROJECT/test --input gs://$PROJECT/data_files/head_usa_names.csv --save_main_session
```

Data lake to Mart

Now build a Dataflow pipeline that reads data from 2 BigQuery data sources, and then joins the data sources. Specifically, you:

- Ingest files from 2 BigQuery sources.
- Join the 2 data sources.
- Filter out the header row in the files.
- Convert the lines read to dictionary objects.
- Output the rows to BigQuery.

```bash
python dataflow_python_examples/data_lake_to_mart.py --worker_disk_type="compute.googleapis.com/projects//zones//diskTypes/pd-ssd" --max_num_workers=4 --project=$PROJECT --runner=DataflowRunner --staging_location=gs://$PROJECT/test --temp_location gs://$PROJECT/test --save_main_session --region=us-central1
```

## Predict Visitor Purchases with a Classification Model in BQML

done

## Predict Housing Prices with Tensorflow and AI Platform

git clone https://github.com/GoogleCloudPlatform/training-data-analyst

From within the Jupyter console, select training-data-analyst > blogs > housing_prices > cloud-ml-housing-prices.ipynb to begin the lab. Now you're ready to start!

From the top right corner, select Python 2 and change it to Python 3.

What we covered

- How to use Tensorflow's high level Estimator API
- How to deploy tensorflow code for distributed training in the cloud
- How to deploy the resulting model to the cloud for online prediction

What we didn't cover

- How to leverage larger than memory datasets using Tensorflow's queueing system
- How to create synthetic features from our raw data to aid learning (Feature Engineering)
- How to improve model performance by finding the ideal hyperparameters using Cloud AI Platform's HyperTune feature

## Cloud Composer: Copying BigQuery Tables Across Different Locations

- DAG - A Directed Acyclic Graph is a collection of tasks, organised to reflect their relationships and dependencies.
- Operator - The description of a single task, it is usually atomic. For example, the BashOperator is used to execute bash command.
- Task - A parameterised instance of an Operator; a node in the DAG.
- Task Instance - A specific run of a task; characterised as: a DAG, a Task, and a point in time. It has an indicative state: running, success, failed, skipped, ...

https://github.com/GoogleCloudPlatform/python-docs-samples/blob/master/composer/workflows/bq_copy_across_locations.py

```bash
sudo apt-get update
sudo apt-get install virtualenv
virtualenv -p python3 venv
source venv/bin/activate
export DAGS_BUCKET=us-central1-composer-advanc-YOURDAGSBUCKET-bucket
```

airflow envs:

- table_list_file_path: /home/airflow/gcs/dags/bq_copy_eu_to_us_sample.csv
- gcs_source_bucket: {UNIQUE ID}-us
- gcs_dest_bucket: {UNIQUE ID}-eu

```bash
gcloud composer environments run ENVIRONMENT_NAME \
    --location LOCATION variables -- \
    --set KEY VALUE
gcloud composer environments run composer-advanced-lab \
    --location us-central1 variables -- \
    --get gcs_source_bucket
```

```bash
cd ~
git clone https://github.com/GoogleCloudPlatform/python-docs-samples
# gsutil cp -r python-docs-samples/third_party/apache-airflow/plugins/* gs://$DAGS_BUCKET/plugins
gsutil cp python-docs-samples/composer/workflows/bq_copy_across_locations.py gs://$DAGS_BUCKET/dags
gsutil cp python-docs-samples/composer/workflows/bq_copy_eu_to_us_sample.csv gs://$DAGS_BUCKET/dags
```

Validate...

## Chanlledge

1. In the Cloud Console, navigate to Menu > BigQuery.
2. Click on More > Query settings under the Query Editor.
3. Select Set a destination table for query results under Destination; Enter taxi_training_data as the Table name

```bash
Select
  pickup_datetime,
  pickup_longitude AS pickuplon,
  pickup_latitude AS pickuplat,
  dropoff_longitude AS dropofflon,
  dropoff_latitude AS dropofflat,
  passenger_count AS passengers,
  ( tolls_amount + fare_amount ) AS fare_amount
FROM
  `taxirides.historical_taxi_rides_raw`
WHERE
  trip_distance > 0
  AND fare_amount >= 2.5
  AND pickup_longitude > -75
  AND pickup_longitude < -73
  AND dropoff_longitude > -75
  AND dropoff_longitude < -73
  AND pickup_latitude > 40
  AND pickup_latitude < 42
  AND dropoff_latitude > 40
  AND dropoff_latitude < 42
  AND passenger_count > 0
  AND RAND() < 999999 / 1031673361
```

model

```bash
CREATE or REPLACE MODEL
  taxirides.fare_model OPTIONS (model_type='linear_reg',
    labels=['fare_amount']) AS
WITH
  taxitrips AS (
  SELECT
    *,
    ST_Distance(ST_GeogPoint(pickuplon, pickuplat), ST_GeogPoint(dropofflon, dropofflat)) AS euclidean
  FROM
    `taxirides.taxi_training_data` )
  SELECT
    *
  FROM
    taxitrips
```

evaluate

```bash
SELECT
  SQRT(mean_squared_error) AS rmse
FROM
  ML.EVALUATE(MODEL taxirides.fare_model,
    (
    WITH
      taxitrips AS (
      SELECT
        *,
        ST_Distance(ST_GeogPoint(pickuplon, pickuplat), ST_GeogPoint(dropofflon, dropofflat)) AS euclidean
      FROM
        `taxirides.taxi_training_data` )
      SELECT
        *
      FROM
        taxitrips ))
```

predict

```bash
SELECT
  *
FROM
  ML.PREDICT(MODEL `taxirides.fare_model`,
    (
    WITH
      taxitrips AS (
      SELECT
        *,
        ST_Distance(ST_GeogPoint(pickuplon, pickuplat)   , ST_GeogPoint(dropofflon, dropofflat)) AS    euclidean
      FROM
        `taxirides.report_prediction_data` )
    SELECT
      *
    FROM
      taxitrips ))
```