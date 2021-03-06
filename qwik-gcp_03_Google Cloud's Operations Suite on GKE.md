# Google Cloud's Operations Suite on GKE

[Google Cloud's Operations Suite on GKE](https://www.qwiklabs.com/quests/133)

## Monitoring GKE with Prometheus and Cloud Monitoring

export PROJECT_ID=qwiklabs-gcp-04-e23c17625801
gcloud services enable compute.googleapis.com container.googleapis.com spanner.googleapis.com cloudprofiler.googleapis.com
git clone https://github.com/saturnism/spring-petclinic-gcp
gcloud container clusters create petclinic \
 --region=us-central1 \
 --num-nodes=2 \
 --machine-type=n1-standard-2 \
 --enable-autorepair \
 --enable-stackdriver-kubernetes \
 --cluster-version "1.15.12-gke.6001"

### Set up Cloud Monitoring

kubectl apply -f https://storage.googleapis.com/stackdriver-kubernetes/freshness-fixes/rbac-setup.yaml --as=admin --as-group=system:masters
kubectl create configmap \
 --namespace=stackdriver-agents \
 google-cloud-config \
 --from-literal=project_id=$PROJECT_ID \
 --from-literal=credentials_path=""
kubectl create configmap \
 --namespace=stackdriver-agents \
 cluster-config \
 --from-literal=cluster_name=petclinic \
 --from-literal=cluster_location=us-central1
kubectl apply -f https://storage.googleapis.com/stackdriver-kubernetes/freshness-fixes/agents.yaml
kubectl get pods --namespace=stackdriver-agents

### Install the Cloud Monitoring Prometheus Collector

kubectl apply -f https://storage.googleapis.com/spls/gsp243/rbac-setup.yaml --as=admin --as-group=system:masters
curl -s https://storage.googleapis.com/spls/gsp243/prometheus-service.yaml | \
 sed -e "s/\(\s*\_kubernetes_cluster_name:*\)._/\1 'petclinic'/g" | \
 sed -e "s/\(\s_\_kubernetes*location:*\).\_/\1 'us-central1'/g" | \
 sed -e "s/\(\s*\_stackdriver_project_id:*\).\*/\1 '${PROJECT_ID}'/g" | \
  kubectl apply -f -
cd ~/
export ISTIO_VERSION=0.7.1
curl -L https://git.io/getLatestIstio | sh -
cd istio-$ISTIO_VERSION
kubectl apply -f install/kubernetes/istio.yaml --as=admin --as-group=system:masters --validate=false

cd ~/
gcloud spanner instances create petclinic --config=regional-us-central1 --nodes=1 --description="PetClinic Spanner Instance"
gcloud spanner databases create petclinic --instance=petclinic
gcloud spanner databases ddl update petclinic --instance=petclinic --ddl='CREATE TABLE owners (owner_id STRING(36) NOT NULL, first_name STRING(128) NOT NULL, last_name STRING(128) NOT NULL, address STRING(256) NOT NULL, city STRING(128) NOT NULL, telephone STRING(20) NOT NULL) PRIMARY KEY (owner_id);'
gcloud spanner databases ddl update petclinic --instance=petclinic --ddl='CREATE TABLE pets (owner_id STRING(36) NOT NULL, pet_id STRING(36) NOT NULL, name STRING(128) NOT NULL, birth_date DATE NOT NULL, type STRING(16) NOT NULL,) PRIMARY KEY (owner_id, pet_id), INTERLEAVE IN PARENT owners ON DELETE CASCADE;'
gcloud spanner databases ddl update petclinic --instance=petclinic --ddl='CREATE TABLE vets (vet_id STRING(36) NOT NULL, first_name STRING(128) NOT NULL, last_name STRING(128) NOT NULL, specialties ARRAY<STRING(32)> NOT NULL,) PRIMARY KEY (vet_id);'
gcloud spanner databases ddl update petclinic --instance=petclinic --ddl='CREATE INDEX vets_by_last_name ON vets(last_name);'
gcloud spanner databases ddl update petclinic --instance=petclinic --ddl='CREATE TABLE visits (owner_id STRING(36) NOT NULL, pet_id STRING(36) NOT NULL, visit_id STRING(36) NOT NULL, date DATE NOT NULL, description STRING(MAX) NOT NULL,) PRIMARY KEY (owner_id, pet_id, visit_id), INTERLEAVE IN PARENT pets ON DELETE CASCADE;'

### Generate a service account and grant appropriate permissions

gcloud iam service-accounts create petclinic --display-name "Petclinic Service Account"
gcloud projects add-iam-policy-binding $PROJECT_ID \
     --member serviceAccount:petclinic@$PROJECT_ID.iam.gserviceaccount.com \
 --role roles/cloudprofiler.agent
gcloud projects add-iam-policy-binding $PROJECT_ID \
     --member serviceAccount:petclinic@$PROJECT_ID.iam.gserviceaccount.com \
 --role roles/clouddebugger.agent
gcloud projects add-iam-policy-binding $PROJECT_ID \
     --member serviceAccount:petclinic@$PROJECT_ID.iam.gserviceaccount.com \
 --role roles/cloudtrace.agent
gcloud projects add-iam-policy-binding $PROJECT_ID \
     --member serviceAccount:petclinic@$PROJECT_ID.iam.gserviceaccount.com \
 --role roles/spanner.databaseUser
gcloud iam service-accounts keys create ~/petclinic-service-account.json \
 --iam-account petclinic@$PROJECT_ID.iam.gserviceaccount.com
gcloud iam service-accounts keys create ~/petclinic-service-account.json \
    --iam-account petclinic@$PROJECT_ID.iam.gserviceaccount.com
kubectl create secret generic petclinic-credentials --from-file=$HOME/petclinic-service-account.json

### Deploy the Petclinic demo app

cd ~/spring-petclinic-gcp
kubectl apply -f kubernetes/

## Cloud Operations for GKE

gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a
git clone https://github.com/GoogleCloudPlatform/gke-monitoring-tutorial.git
cd gke-monitoring-tutorial
gcloud auth application-default login

## Cloud Logging on Kubernetes Engine

gcloud config set project qwiklabs-gcp-02-854f9e3de16d
git clone https://github.com/GoogleCloudPlatform/gke-logging-sinks-demo
cd gke-logging-sinks-demo
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

## Using Cloud Trace on Kubernetes Engine

The lab begins by deploying a Kubernetes Engine cluster. To this cluster will be deployed a simple web application fronted by a load balancer. The web app will publish messages provided by the user to a Cloud Pub/Sub topic. The application is instrumented such that HTTP requests to it will result in the creation of a trace whose context will be propagated to the Cloud Pub/Sub publish API request.

```bash
gcloud config set project qwiklabs-gcp-02-854f9e3de16d
git clone https://github.com/GoogleCloudPlatform/gke-tracing-demo
cd gke-tracing-demo
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a
```

If `provider.tf` contains static version string for the google provider as given below then remove it.

```bash
...
provider "google" {
  project = var.project
}
...
```

initialize terraform

```bash
terraform init
```

../scripts/generate-tfvars.sh
gcloud config list
terraform plan
terraform apply
kubectl apply -f tracing-demo-deployment.yaml
gcloud pubsub subscriptions pull --auto-ack --limit 10 tracing-demo-cli

Trace
Monitor
Logging

## Debugging Apps on Google Kubernetes Engine

gcloud config set project qwiklabs-gcp-00-b3dbe849dc72
gcloud config set compute/zone us-central1-b
export PROJECT_ID=$(gcloud info --format='value(config.project)')
gcloud container clusters list
gcloud container clusters get-credentials central --zone us-central1-b
kubectl get nodes
git clone https://github.com/xiangshen-dk/microservices-demo.git
cd microservices-demo
kubectl apply -f release/kubernetes-manifests.yaml
kubectl get pods
export EXTERNAL_IP=$(kubectl get service frontend-external | awk 'BEGIN { cnt=0; } { cnt+=1; if (cnt > 1) print $4; }')
curl -o /dev/null -s -w "%{http_code}\n"  http://$EXTERNAL_IP
