# Deploy and Manage Cloud Environments with Google Cloud

## Orchestrating the Cloud with Kubernetes

https://github.com/kelseyhightower/app

- Get the sample code
- Quick Kubernetes Demo
- Pods
- Creating Pods
- Interacting with Pods
- Services
- Creating a Service
- Adding Labels to Pods
- Deploying Applications with Kubernetes
- Creating Deployments

```shell
gcloud config set compute/zone us-central1-b
gcloud container clusters create io
gsutil cp -r gs://spls/gsp021/* .
cd orchestrate-with-kubernetes/kubernetes
ls
kubectl create deployment nginx --image=nginx:1.10.0
kubectl get pods
kubectl expose deployment nginx --port 80 --type LoadBalancer
kubectl get services
curl http://<External IP>:80
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
kubectl create -f pods/secure-monolith.yaml
gcloud compute firewall-rules create allow-monolith-nodeport \
  --allow=tcp:31000
kubectl get pods -l "app=monolith"
kubectl get pods -l "app=monolith,secure=enabled"
kubectl label pods secure-monolith 'secure=enabled'
kubectl get pods secure-monolith --show-labels
```

## Deployment Manager - Full Production

done before

## Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine

![cd-spinsnacker.png](cd-spinsnacker.png)

![cd-pipeline.png](cd-pipeline.png)

```bash
gcloud config set compute/zone us-central1-f
gcloud container clusters create spinnaker-tutorial --machine-type=n1-standard-2 --cluster-version "1.15.12-gke.4000"
```

Create a Cloud Identity Access Management (Cloud IAM) service account to delegate permissions to Spinnaker, allowing it to store data in Cloud Storage. Spinnaker stores its pipeline data in Cloud Storage to ensure reliability and resiliency. If your Spinnaker deployment unexpectedly fails, you can create an identical deployment in minutes with access to the same pipeline data as the original.

```shell
gcloud iam service-accounts create spinnaker-account \
    --display-name spinnaker-account
export SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:spinnaker-account" \
    --format='value(email)')
export PROJECT=$(gcloud info --format='value(config.project)')
gcloud projects add-iam-policy-binding $PROJECT \
    --role roles/storage.admin \
    --member serviceAccount:$SA_EMAIL
gcloud iam service-accounts keys create spinnaker-sa.json \
     --iam-account $SA_EMAIL
```

Set up Cloud Pub/Sub to trigger Spinnaker pipelines

```shell
gcloud pubsub topics create projects/$PROJECT/topics/gcr
gcloud pubsub subscriptions create gcr-triggers \
    --topic projects/${PROJECT}/topics/gcr
export SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:spinnaker-account" \
    --format='value(email)')
gcloud beta pubsub subscriptions add-iam-policy-binding gcr-triggers \
    --role roles/pubsub.subscriber --member serviceAccount:$SA_EMAIL
```

Install Helm

```shell
wget https://get.helm.sh/helm-v3.1.0-linux-amd64.tar.gz
tar zxfv helm-v3.1.0-linux-amd64.tar.gz
sudo cp linux-amd64/helm /usr/local/bin/
kubectl create clusterrolebinding user-admin-binding \
    --clusterrole=cluster-admin --user=$(gcloud config get-value account)
helm repo add stable https://kubernetes-charts.storage.googleapis.com
helm repo update
export PROJECT=$(gcloud info \
    --format='value(config.project)')
export BUCKET=$PROJECT-spinnaker-config
gsutil mb -c regional -l us-central1 gs://$BUCKET
export SA_JSON=$(cat spinnaker-sa.json)
export PROJECT=$(gcloud info --format='value(config.project)')
export BUCKET=$PROJECT-spinnaker-config
cat > spinnaker-config.yaml <<EOF
gcs:
  enabled: true
  bucket: $BUCKET
  project: $PROJECT
  jsonKey: '$SA_JSON'

dockerRegistries:
- name: gcr
  address: https://gcr.io
  username: _json_key
  password: '$SA_JSON'
  email: 1234@5678.com

# Disable minio as the default storage backend
minio:
  enabled: false

# Configure Spinnaker to enable GCP services
halyard:
  additionalScripts:
    create: true
    data:
      enable_gcs_artifacts.sh: |-
        \$HAL_COMMAND config artifact gcs account add gcs-$PROJECT --json-path /opt/gcs/key.json
        \$HAL_COMMAND config artifact gcs enable
      enable_pubsub_triggers.sh: |-
        \$HAL_COMMAND config pubsub google enable
        \$HAL_COMMAND config pubsub google subscription add gcr-triggers \
          --subscription-name gcr-triggers \
          --json-path /opt/gcs/key.json \
          --project $PROJECT \
          --message-format GCR
EOF
helm install -n default cd stable/spinnaker -f spinnaker-config.yaml \
           --version 1.23.0 --timeout 10m0s --wait
export DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" \
    -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward --namespace default $DECK_POD 8080:9000 >> /dev/null &

student_00_c8a00fa33c97@cloudshell:~ (qwiklabs-gcp-00-8444ffb9843c)$ helm install -n default cd stable/spinnaker -f spinnaker-config.yaml            --version 1.23.0 --timeout 10m0s --wait
Error: unable to build kubernetes objects from release manifest: unable to recognize "": no matches for kind "StatefulSet" in version "apps/v1beta2"
```

To open the Spinnaker user interface, click the Web Preview icon at the top of the Cloud Shell window and select Preview on port 8080.

Building Docker Image

Navigation Menu > Source Repositories.

```shell
wget https://gke-spinnaker.storage.googleapis.com/sample-app-v2.tgz
tar xzfv sample-app-v2.tgz
cd sample-app
git config --global user.email "$(gcloud config get-value core/account)"
git config --global user.name "[USERNAME]"
git init
git add .
git commit -m "Initial commit"
gcloud source repos create sample-app
git config credential.helper gcloud.sh
export PROJECT=$(gcloud info --format='value(config.project)')
git remote add origin https://source.developers.google.com/p/$PROJECT/r/sample-app
git push origin master
```

In the Cloud Platform Console, click Navigation menu > Cloud Build > Triggers.
Click Create trigger.
Set the following trigger settings:
    Name:sample-app-tags
    Event: Push new tag
    Select your newly created sample-app repository.
    Tag: v.*
    Build configuration: Cloud Build configuration file (yaml or json)
    Cloud Build configuration file location: /cloudbuild.yaml

Prepare your Kubernetes Manifests for use in Spinnaker

```shell
export PROJECT=$(gcloud info --format='value(config.project)')
gsutil mb -l us-central1 gs://$PROJECT-kubernetes-manifests
gsutil versioning set on gs://$PROJECT-kubernetes-manifests
sed -i s/PROJECT/$PROJECT/g k8s/deployments/*
git commit -a -m "Set project ID"
git tag v1.0.0
git push --tags
```

Go to the Cloud Console. Still in Cloud Build, click History . Stay on this page and wait for the build to complete before going on to the next section.

Configuring your deployment pipelines

```shell
curl -LO https://storage.googleapis.com/spinnaker-artifacts/spin/1.14.0/linux/amd64/spin
chmod +x spin
./spin application save --application-name sample \
                        --owner-email "$(gcloud config get-value core/account)" \
                        --cloud-providers kubernetes \
                        --gate-endpoint http://localhost:8080/gate
export PROJECT=$(gcloud info --format='value(config.project)')
sed s/PROJECT/$PROJECT/g spinnaker/pipeline-deploy.json > pipeline.json
./spin pipeline save --gate-endpoint http://localhost:8080/gate -f pipeline.json
```

## Multiple VPC Networks

done before

## Site Reliability Troubleshooting with Cloud Monitoring APM

- Deploy application
- Develop Sample SLOs and SLIs
- Configure Latency SLI
- Configure Availability SLI
- Deploy new release
- Send some data
- Latency SLO Violation - Find the Problem
- Deploy Change to Address Latency
- Error Rate SLO Violation - Find the Problem
- Deploy Change to Address Error Rate
- Application optimization with Cloud Monitoring APM

```shell
gcloud config set compute/zone us-west1-b
export PROJECT_ID=$(gcloud info --format='value(config.project)')
gcloud container clusters list
gcloud container clusters get-credentials shop-cluster --zone us-west1-b
kubectl get nodes
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
ln -s ~/training-data-analyst/blogs/microservices-demo-1 ~/microservices-demo-1
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/v0.36.0/skaffold-linux-amd64 && chmod +x skaffold && sudo mv skaffold /usr/local/bin
cd microservices-demo-1
skaffold run
export EXTERNAL_IP=$(kubectl get service frontend-external | awk 'BEGIN { cnt=0; } { cnt+=1; if (cnt > 1) print $4; }')
curl -o /dev/null -s -w "%{http_code}\n"  http://$EXTERNAL_IP
./setup_csr.sh
```

service level indicators (SLIs), objectives (SLOs), and agreements (SLAs)

"Most services consider request latency — how long it takes to return a response to a request — as a key SLI. Other common SLIs include the error rate, often expressed as a fraction of all requests received, and system throughput, typically measured in requests per second. Another kind of SLI important to SREs is availability, or the fraction of the time that a service is usable. It is often defined in terms of the fraction of well-formed requests that succeed. Durability — the likelihood that data is be retained over a long period of time — is equally important for data storage systems. The measurements are often aggregated: i.e., raw data is collected over a measurement window and then turned into a rate, average, or percentile."

Application Performance Management (APM)

custom.googleapis.com/opencensus/grpc.io/client/roundtrip_latency

## Continuous Delivery with Jenkins in Kubernetes Engine

```bash
gcloud config set compute/zone us-east1-d
gcloud config set project qwiklabs-gcp-01-263b1a8d6591
git clone https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes.git
cd continuous-deployment-on-kubernetes
gcloud container clusters create jenkins-cd \
    --num-nodes 2 \
    --machine-type n1-standard-2 \
    --scopes "https://www.googleapis.com/auth/source.read_write,cloud-platform"
gcloud container clusters list
gcloud container clusters get-credentials jenkins-cd
kubectl cluster-info
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo update
helm install cd stable/jenkins -f jenkins/values.yaml --version 1.2.2 --wait
kubectl get pods
kubectl create clusterrolebinding jenkins-deploy --clusterrole=cluster-admin --serviceaccount=default:cd-jenkins
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
kubectl get svc
printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo

cd sample-app
kubectl create ns production
kubectl apply -f k8s/production -n production
kubectl apply -f k8s/canary -n production
kubectl apply -f k8s/services -n production
kubectl scale deployment gceme-frontend-production -n production --replicas 4
kubectl get pods -n production -l app=gceme -l role=frontend
kubectl get service gceme-frontend -n production
export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
curl http://$FRONTEND_SERVICE_IP/version

gcloud source repos create default
```

## Deploy and Manage Cloud Environments with Google Cloud: Challenge Lab

```bash
gcloud deployment-manager deployments create kraken-prod-network --config 
gcloud container clusters get-credentials kraken-prod --zone us-east1-b
gcloud container clusters get-credentials spinnaker-tutorial --zone us-east1-b
gcloud container clusters get-credentials valkyrie-dev --zone us-east1-d

student_02_9bd16056d4ae@cloudshell:~/valkyrie-app (qwiklabs-gcp-02-c2589803392f)$ printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
d4XtkjMQkJ
```