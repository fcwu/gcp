# Cloud SQL

DB 分類

- BigTable: HBase, low-latency
- DataStore: Based on BigTable, but add MongoDB features
- Cloud SQL: pgSQL, MySQL...
- Cloud Spanner: horizontal scale SQL
- BigQuery: 無敵星星SQL

## Introduction to SQL for BigQuery and Cloud SQL

Import/Insert Data to data BigQuery

- Cloud function to BigQuery
- BigTable to Avro File to BigQuery
- Dataflow to BigQuery
- Cloud Storage to BigQuery
- Public Data to BigQuery
- ...
- Batch load by command (job)
- Data Transfer S3

## Cloud SQL for PostgreSQL: Qwik Start

```
gcloud sql connect myinstance --user=postgres
CREATE TABLE guestbook (guestName VARCHAR(255), content VARCHAR(255), entryID SERIAL PRIMARY KEY);
INSERT INTO guestbook (guestName, content) values ('first guest', 'I got here!');
INSERT INTO guestbook (guestName, content) values ('second guest', 'Me too!');
SELECT * FROM guestbook;
```

## Loading Data into Google Cloud SQL

```
git clone https://github.com/GoogleCloudPlatform/data-science-on-gcp/
cd data-science-on-gcp/03_sqlstudio
export PROJECT_ID=$(gcloud info --format='value(config.project)')
export BUCKET=${PROJECT_ID}-ml
gcloud sql instances create flights --tier=db-n1-standard-1 --activation-policy=ALWAYS
gcloud sql users set-password root --host % --instance flights --password Passw0rd
export ADDRESS=$(wget -qO - http://ipecho.net/plain)/32
gcloud sql instances patch flights --authorized-networks $ADDRESS
MYSQLIP=$(gcloud sql instances describe flights --format="value(ipAddresses.ipAddress)")
mysql --host=$MYSQLIP --user=root  --password --verbose < create_table.sql

counter=0
for FILE in 201501.csv 201502.csv; do
   gsutil cp gs://$BUCKET/flights/raw/$FILE \
             flights.csv-${counter}
   counter=$((counter+1))
done
mysqlimport --local --host=$MYSQLIP --user=root --password --ignore-lines=1 --fields-terminated-by=',' bts flights.csv-*
mysql --host=$MYSQLIP --user=root  --password
```

```
SET @ARR_DELAY_THRESH = 15;
SET @DEP_DELAY_THRESH = 20;
# Correct - true negative
select count(dest) from flights where arr_delay < @ARR_DELAY_THRESH and dep_delay < @DEP_DELAY_THRESH;
# False negative
select count(dest) from flights where arr_delay >= @ARR_DELAY_THRESH and dep_delay < @DEP_DELAY_THRESH;
# False positive
select count(dest) from flights where arr_delay < @ARR_DELAY_THRESH and dep_delay >= @DEP_DELAY_THRESH;
# True positive
select count(dest) from flights where arr_delay >= @ARR_DELAY_THRESH and dep_delay >= @DEP_DELAY_THRESH;
```

## Cloud SQL with Terraform

```
mkdir sql-with-terraform
cd sql-with-terraform
gsutil cp -r gs://spls/gsp234/gsp234.zip .
unzip gsp234.zip
cat main.tf
terraform init
terraform plan -out=tfplan
terraform apply tfplan
```

### Installing the Cloud SQL Proxy

https://cloud.google.com/sql/docs/mysql/connect-admin-proxy?hl=zh-tw

```
wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy
chmod +x cloud_sql_proxy
export GOOGLE_PROJECT=$(gcloud config get-value project)
MYSQL_DB_NAME=$(terraform output -json | jq -r '.instance_name.value')
MYSQL_CONN_NAME="${GOOGLE_PROJECT}:us-central1:${MYSQL_DB_NAME}"
./cloud_sql_proxy -instances=${MYSQL_CONN_NAME}=tcp:3306
cd ~/sql-with-terraform
echo MYSQL_PASSWORD=$(terraform output -json | jq -r '.generated_user_password.value')
mysql -udefault -p --host 127.0.0.1 default
```

## APIs Explorer: Cloud SQL

Enable Cloud Admin SQL API

```
https://cloud.google.com/sql/docs/mysql/admin-api/rest/v1beta4/instances/insert
https://cloud.google.com/sql/docs/mysql/admin-api/v1beta4/databases/insert
```

## Connect to Cloud SQL from an Application in Kubernetes Engine

```
gcloud config set project qwiklabs-gcp-01-e155ca3292b1
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a
git clone https://github.com/GoogleCloudPlatform/gke-cloud-sql-postgres-demo.git
cd gke-cloud-sql-postgres-demo
nano manifests/pgadmin-deployment.yaml
    apiVersion: apps/v1
./create.sh dbadmin pgadmin
POD_ID=$(kubectl --namespace default get pods -o name | cut -d '/' -f 2)
kubectl expose pod $POD_ID --port=80 --type=LoadBalancer
kubectl get svc
http://<SVC_IP>
```
