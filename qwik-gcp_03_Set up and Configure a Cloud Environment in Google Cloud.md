# Set up and Configure a Cloud Environment in Google Cloud

## Cloud IAM: Qwik Start

done before

## Introduction to SQL for BigQuery and Cloud SQL

- The Basics of SQL
- Exploring the BigQuery Console
- More SQL Keywords: GROUP BY, COUNT, AS, and ORDER BY
- Working with Cloud SQL
- New Queries in Cloud SQL

project → dataset → table

- Navigation menu > BigQuery
- Navigation menu > SQL.

```bash
gcloud sql connect  qwiklabs-demo --user=root
```

In your Cloud SQL instance page, click IMPORT.

## Multiple VPC Networks

![gcp_3.png](gcp_3.png)

- Create custom mode VPC networks with firewall rules
- Create VM instances
- Explore the connectivity between VM instances
- Create a VM instance with multiple network interfaces

Navigation menu (mainmenu.png) > VPC network > VPC networks.

```bash
gcloud compute networks create privatenet --subnet-mode=custom
gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24
gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20
gcloud compute networks list
gcloud compute networks subnets list --sort-by=NETWORK
```

Navigation menu (mainmenu.png) > VPC network > Firewall

```bash
gcloud compute --project=qwiklabs-gcp-00-b7bca3ae025c firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0
gcloud compute firewall-rules list --sort-by=NETWORK
gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=privatesubnet-us
```

You are able to ping privatenet-us-vm by its name because VPC networks have an internal DNS service that allows you to address instances by their DNS names rather than their internal IP addresses. When an internal DNS query is made with the instance hostname, it resolves to the primary interface (nic0) of the instance. Therefore, this only works for privatenet-us-vm in this case.

Check

- default global VPC
- default firewall
- if you delete default VPC, you cannot create GCE, can create cloud function and storage

## Cloud Monitoring: Qwik Start

Done before

## Deployment Manager - Full Production

In this lab, you learn to:

- Launch a cloud service from a collection of templates.
- Configure basic black box monitoring of an application.
- Create an uptime check to recognize a loss of service.
- Establish an alerting policy to trigger incident response procedures.
- Create and configure a dashboard with dynamically updated charts.
- Test the monitoring and alerting regimen by applying a load to the service.
- Test the monitoring and alerting regimen by simulating a service outage.

- Create a virtual environment
- Clone the Deployment Manager Sample Templates
- Explore the Sample Files
- Customize the Deployment
- Run the Application
- Verify that the application is operational
- Configure an uptime check and alert policy in Cloud Monitoring
- Configure an alerting policy and notification
- Configure a Dashboard with a Couple of Useful Charts
- Create a test VM with ApacheBench
- Simulate a Service Outage
- Test your knowledge

```bash
git clone https://github.com/GoogleCloudPlatform/deploymentmanager-samples.git
cd deploymentmanager-samples/examples/v2/nodesjs/python
gcloud deployment-manager deployments create advanced-configuration --config nodejs.yaml
sudo apt-get update
sudo apt-get -y install apache2-utils
ab -n 10000 -c 100 http://<Your_IP>:8080/
```

## Managing Deployments Using Kubernetes Engine

```bash
git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git
cd orchestrate-with-kubernetes/kubernetes
gcloud config set compute/zone us-central1-a
gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
kubectl explain deployment
kubectl explain deployment --recursive
kubectl explain deployment.metadata.name
kubectl create -f deployments/auth.yaml
kubectl get deployments
kubectl create -f services/auth.yaml
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`
```

scale

```bash
kubectl scale deployment hello --replicas=5
```

roling update

```bash
kubectl edit deployment hello
kubectl rollout history deployment/hello
kubectl rollout pause deployment/hello
kubectl rollout status deployment/hello
kubectl rollout resume deployment/hello
kubectl rollout undo deployment/hello
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

Canary deployments

```bash
kubectl create -f deployments/hello-canary.yaml
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
# Canary deployments in production - session affinity
kind: Service
apiVersion: v1
metadata:
  name: "hello"
spec:
  sessionAffinity: ClientIP
  selector:
    app: "hello"
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80
```

Blue-green deployments

```bash
kubectl apply -f services/hello-blue.yaml
kubectl create -f deployments/hello-green.yaml
kubectl apply -f services/hello-green.yaml
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```

## Set up and Configure a Cloud Environment in Google Cloud: Challenge Lab

Topics tested:

- Creating and using VPCs and subnets
- Configuring and launching a Deployment Manager configuration
- Creating a Kubernetes cluster
- Configuring and launching a Kubernetes deployment and service
- Setting up stackdriver monitoring
- Configuring an IAM role for an account

```bash
gcloud deployment-manager deployments create dm --config prod-network.yaml
```

Task 4: Create and configure Cloud SQL Instance

```bash
gcloud sql connect griffin-dev-db --user=root
CREATE DATABASE wordpress;
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
FLUSH PRIVILEGES;
```

Task 6: Prepare the Kubernetes cluster

```bash
gcloud container clusters get-credentials griffin-dev --zone us-east1-b
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

Task 7: Create a WordPress deployment

Now you have provisioned the MySQL database, and set up the secrets and volume, you can create the deployment using wp-deployment.yaml. Before you create the deployment you need to edit wp-deployment.yaml and replace YOUR_SQL_INSTANCE with griffin-dev-db's Instance connection name. Get the Instance connection name from your Cloud SQL instance.