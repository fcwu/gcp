# Create and manage cloud resources

## A Tour of Qwiklabs and Google Cloud

https://cloud.google.com/docs/overview/cloud-platform-services#top_of_page

- Compute: houses a variety of machine types that support any type of workload. The different computing options let you decide how involved you want to be with operational details and infrastructure amongst other things.
- Storage: data storage and database options for structured or unstructured, relational or non relational data.
- Networking: services that balance application traffic and provision security rules amongst other things.
- Cloud Operations: a suite of cross-cloud logging, monitoring, trace, and other service reliability tools.
- Tools: services for developers managing deployments and application build pipelines.
- Big Data: services that allow you to process and analyze large datasets.
- Artificial Intelligence: a suite of APIs that run specific artificial intelligence and machine learning tasks on Google Cloud.

## Creating a Virtual Machine

```bash
gcloud auth list
gcloud config list project
gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-c
gcloud config set compute/zone ...
gcloud config set compute/region ...
gcloud compute ssh gcelab2 --zone us-central1-c
```

## Compute Engine: Qwik Start - Windows

```bash
gcloud compute instances get-serial-port-output instance-1 --zone us-central1-a
```

## Getting Started with Cloud Shell & gcloud

```bash
gcloud compute project-info describe --project <your_project_ID>
export PROJECT_ID=<your_project_ID>
export ZONE=<your_zone>
gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone $ZONE
gcloud config list --all
gcloud components list
gcloud beta interactive
```

## Kubernetes Engine: Qwik Start

When you run a Kubernetes Engine cluster, you also gain the benefit of advanced cluster management features that Google Cloud provides. These include:

- Load-balancing for Compute Engine instances.
- Node Pools to designate subsets of nodes within a cluster for additional flexibility.
- Automatic scaling of your cluster's node instance count.
- Automatic upgrades for your cluster's node software.
- Node auto-repair to maintain node health and availability.
- Logging and Monitoring with Cloud Monitoring for visibility into your cluster.

```bash
gcloud config list project
gcloud container clusters create [CLUSTER-NAME]
gcloud container clusters get-credentials [CLUSTER-NAME]
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
gcloud container clusters delete
```

## Set Up Network and HTTP Load Balancers

There are several ways you can load balance in Google Cloud. This lab takes you through the setup of the following load balancers.:

### L4 Network Load Balancer

```bash
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
gcloud compute instance-templates create nginx-template \
         --metadata-from-file startup-script=startup.sh
gcloud compute target-pools create nginx-pool
gcloud compute instance-groups managed create nginx-group \
         --base-instance-name nginx \
         --size 2 \
         --template nginx-template \
         --target-pool nginx-pool
gcloud compute instances list
gcloud compute firewall-rules create www-firewall --allow tcp:80
gcloud compute forwarding-rules create nginx-lb \
         --region us-central1 \
         --ports=80 \
         --target-pool nginx-pool
gcloud compute forwarding-rules list
```

### L7 HTTP(s) Load Balancer

```bash
gcloud compute http-health-checks create http-basic-check
gcloud compute instance-groups managed \
       set-named-ports nginx-group \
       --named-ports http:80
gcloud compute backend-services create nginx-backend \
      --protocol HTTP --http-health-checks http-basic-check --global
gcloud compute backend-services add-backend nginx-backend \
    --instance-group nginx-group \
    --instance-group-zone asia-east1-b \
    --global
gcloud compute url-maps create web-map \
    --default-service nginx-backend
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map
gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80
gcloud compute forwarding-rules list

Compute Engine
    -> instance groups (instance-group, instance-templates)
        -> nginx-group
    -> instance templates
        -> nginx-template
    -> Health check
        -> http-basic-check

Network Services
    -> frontends (forwarding-rules)
        -> http-lb-proxy
        -> http-content-rule
    -> Load balancing (url-maps)
        -> web-map (in: backend-services)
        -> nginx-pool (attach to nginx-group)
    -> backends (target-pools, backend-services)
        -> nginx-backend (in: instance-group, http-health-checks)
```

## Getting Started: Create and Manage Cloud Resources: Challenge Lab

Topics tested:

- Create an instance.
- Create a 3 node Kubernetes cluster and run a simple service.
- Create an HTTP(s) Load Balancer in front of two web servers.
