# Getting Started with Go on Google Cloud

## Use Go Code to Work with Google Cloud Data Sources

git clone https://github.com/GoogleCloudPlatform/DIY-Tools.git

    In the Cloud Console, click Navigation menu > Firestore > Data to open Firestore in the Console.
    Click Select Native Mode.
    In Select a location, scroll down and select us-east1 (South Carolina).
    Click Create Database.

gcloud firestore import gs://$PROJECT_ID-firestore/prd-back

review the code

https://8080-dot-11197143-dot-devshell.appspot.com/fs/qwiklabs-gcp-02-e1a49657e849/symbols/product/symbol

gcloud app deploy app.yaml --project $PROJECT_ID -q
export TARGET_URL=https://$(gcloud app describe --format="value(defaultHostname)")


    Firestore :

[SERVICE_URL]/fs/[PROJECT_ID]/[COLLECTION]/[DOCUMENT]

    BigQuery:

[SERVICE_URL]/bq/[PROJECT_ID]/[DATASET]/[TABLE]

## Deploy Go Apps on Google Cloud Serverless Platforms 

export PROJECT_ID=$(gcloud info --format="value(config.project)")
export PROJECT_ID=qwiklabs-gcp-00-ab9c9bc54885
gcloud config set project $PROJECT_ID
git clone https://github.com/GoogleCloudPlatform/DIY-Tools.git
gcloud firestore import gs://$PROJECT_ID-firestore/prd-back

Enable GCP service
Cloud Build -> Settings

cd ~/DIY-Tools/gcp-data-drive

### cloud run

gcloud builds submit --config cloudbuild_run.yaml \
   --project $PROJECT_ID --no-source \
--substitutions=_GIT_SOURCE_BRANCH="master",_GIT_SOURCE_URL="https://github.com/GoogleCloudPlatform/DIY-Tools"
export CLOUD_RUN_SERVICE_URL=$(gcloud run services --platform managed describe gcp-data-drive --region us-central1 --format="value(status.url)")
curl $CLOUD_RUN_SERVICE_URL/fs/$PROJECT_ID/symbols/product/symbol
curl $CLOUD_RUN_SERVICE_URL/bq/$PROJECT_ID/publicviews/ca_zip_codes

### cloud function

gcloud builds submit --config cloudbuild_gcf.yaml \
   --project $PROJECT_ID --no-source \
--substitutions=_GIT_SOURCE_BRANCH="master",_GIT_SOURCE_URL="https://github.com/GoogleCloudPlatform/DIY-Tools"
gcloud alpha functions add-iam-policy-binding gcp-data-drive --member=allUsers --role=roles/cloudfunctions.invoker
export CF_TRIGGER_URL=$(gcloud functions describe gcp-data-drive --format="value(httpsTrigger.url)")
curl $CF_TRIGGER_URL/fs/$PROJECT_ID/symbols/product/symbol
curl $CF_TRIGGER_URL/bq/$PROJECT_ID/publicviews/ca_zip_codes

### cloud function by event

https://cloud.google.com/functions/docs/calling/cloud-firestore#functions_firebase_firestore-go

### App Engine

gcloud builds submit  --config cloudbuild_appengine.yaml \
   --project $PROJECT_ID --no-source \
   --substitutions=_GIT_SOURCE_BRANCH="master",_GIT_SOURCE_URL="https://github.com/GoogleCloudPlatform/DIY-Tools"

export TARGET_URL=https://$(gcloud app describe --format="value(defaultHostname)")
curl $TARGET_URL/fs/$PROJECT_ID/symbols/product/symbol
curl $TARGET_URL/bq/$PROJECT_ID/publicviews/ca_zip_codes

nano loadgen.sh
#!/bin/bash
for ((i=1;i<=1000;i++));
do
   curl $TARGET_URL/bq/$PROJECT_ID/publicviews/ca_zip_codes > /dev/null &
done
chmod +x loadgen.sh

## App Engine: Qwik Start - Go 

git clone https://github.com/GoogleCloudPlatform/golang-samples.git
cd golang-samples/appengine/go11x/helloworld
gcloud app deploy
gcloud app browse

## HTTP Google Cloud Functions in Go

gcloud services enable cloudfunctions.googleapis.com
curl -LO https://github.com/GoogleCloudPlatform/golang-samples/archive/master.zip
unzip master.zip
cd golang-samples-master/functions/codelabs/gopher
gcloud functions deploy HelloWorld --runtime go111 --trigger-http
gcloud alpha functions add-iam-policy-binding HelloWorld --member=allUsers --role=roles/cloudfunctions.invoker
curl https://<REGION>-$GOOGLE_CLOUD_PROJECT.cloudfunctions.net/HelloWorld

gcloud functions deploy Gopher --runtime go111 --trigger-http


cd golang-samples-master/functions/codelabs/gopher
go test -v

gcloud functions delete Gopher
gcloud functions delete HelloWorld