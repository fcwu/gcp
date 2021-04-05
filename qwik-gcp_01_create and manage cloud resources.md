# Create and manage cloud resources

[Create and manage cloud resources](https://www.qwiklabs.com/quests/120)

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

There are several ways you can load balance in Google Cloud. This lab takes you through the setup of the following load balancers.

- Network Load Balancer
- HTTP(s) Load Balancer

```
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

### L4 Network Load Balancer

1. Create 3 instances

   ```bash
   for i in `seq 3`; do
       gcloud compute instances create www$i \
       --image-family debian-9 \
       --image-project debian-cloud \
       --zone us-central1-a \
       --tags network-lb-tag \
       --metadata startup-script="#! /bin/bash
           sudo apt-get update
           sudo apt-get install apache2 -y
           sudo service apache2 restart
           echo '<!doctype html><html><body><h1>www$i</h1></body></html>' | tee /var/www/html/index.html"
   done
   ```

2. Create a firewall rule

   ```bash
   gcloud compute firewall-rules create www-firewall-network-lb \
       --target-tags network-lb-tag --allow tcp:80
   ```

3. check liveness

   ```bash
   gcloud compute instances list
   curl http://[IP_ADDRESS]
   ```

4. Create a static external IP address for your load balancer:

   ```bash
   gcloud compute addresses create network-lb-ip-1 \
   --region us-central1
   ```

5. Add a legacy HTTP health check resource:

   ```bash
   gcloud compute http-health-checks create basic-check
   ```

6. Add a target pool in the same region as your instances.

   ```bash
   gcloud compute target-pools create www-pool \
       --region us-central1 --http-health-check basic-check
   ```

7. Add the instances to the pool

   ```bash
   gcloud compute target-pools add-instances www-pool \
       --instances www1,www2,www3
   ```

8. Add a forwarding rule

   ```bash
   gcloud compute forwarding-rules create www-rule \
       --region us-central1 \
       --ports 80 \
       --address network-lb-ip-1 \
       --target-pool www-pool
   ```

9. test

   ```bash
   gcloud compute forwarding-rules describe www-rule --region us-central1
   while true; do curl -m1 IP_ADDRESS; done
   ```

### L7 HTTP(s) Load Balancer

1. First, create the load balancer template:

   ```bash
   gcloud compute instance-templates create lb-backend-template \
       --region=us-central1 \
       --network=default \
       --subnet=default \
       --tags=allow-health-check \
       --image-family=debian-9 \
       --image-project=debian-cloud \
       --metadata=startup-script='#! /bin/bash
           apt-get update
           apt-get install apache2 -y
           a2ensite default-ssl
           a2enmod ssl
           vm_hostname="$(curl -H "Metadata-Flavor:Google" \
           http://169.254.169.254/computeMetadata/v1/instance/name)"
           echo "Page served from: $vm_hostname" | \
           tee /var/www/html/index.html
           systemctl restart apache2'
   ```

2. Create a managed instance group based on the template:

   ```bash
   gcloud compute instance-groups managed create lb-backend-group \
   --template=lb-backend-template --size=2 --zone=us-central1-a
   ```

3. Create the fw-allow-health-check firewall rule.

   ```bash
   gcloud compute firewall-rules create fw-allow-health-check \
       --network=default \
       --action=allow \
       --direction=ingress \
       --source-ranges=130.211.0.0/22,35.191.0.0/16 \
       --target-tags=allow-health-check \
       --rules=tcp:80
   ```

4. Now that the instances are up and running, set up a global static external IP address that your customers use to reach your load balancer.

   ```bash
   gcloud compute addresses create lb-ipv4-1 \
       --ip-version=IPV4 \
       --global
   gcloud compute addresses describe lb-ipv4-1 \
       --format="get(address)" \
       --global
   ```

5. Create a healthcheck for the load balancer:

   ```bash
   gcloud compute health-checks create http http-basic-check \
       --port 80
   ```

6. Create a backend service

```bash
    gcloud compute backend-services create web-backend-service \
        --protocol=HTTP \
        --port-name=http \
        --health-checks=http-basic-check \
        --global
```

7. Add your instance group as the backend to the backend service:

   ```bash
   gcloud compute backend-services add-backend web-backend-service \
       --instance-group=lb-backend-group \
       --instance-group-zone=us-central1-a \
       --global
   ```

8. Create a URL map to route the incoming requests to the default backend service:

   ```bash
   gcloud compute url-maps create web-map-http \
       --default-service web-backend-service
   ```

9. Create a target HTTP proxy to route requests to your URL map:

   ```bash
   gcloud compute target-http-proxies create http-lb-proxy \
       --url-map web-map-http
   ```

10. Create a global forwarding rule to route incoming requests to the proxy:

    ```bash
    gcloud compute forwarding-rules create http-content-rule \
        --address=lb-ipv4-1\
        --global \
        --target-http-proxy=http-lb-proxy \
        --ports=80
    ```

## Getting Started: Create and Manage Cloud Resources: Challenge Lab

Topics tested:

- Create an instance.
- Create a 3 node Kubernetes cluster and run a simple service.
- Create an HTTP(s) Load Balancer in front of two web servers.
