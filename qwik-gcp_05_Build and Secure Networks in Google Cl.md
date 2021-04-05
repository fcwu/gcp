# Build and Secure Networks in Google Cloud

[Build and Secure Networks in Google Cloud](https://www.qwiklabs.com/quests/128)

## User Authentication: Identity-Aware Proxy

- Deploy Application and Protect it with IAP
- Access User Identity Information
- Use Cryptographic Verification

```bash
git clone https://github.com/googlecodelabs/user-authentication-with-iap.git
cd user-authentication-with-iap
gcloud app deploy
gcloud app browse
```

Navigation menu > Security > Identity-Aware Proxy.

```bash
cd ~/user-authentication-with-iap/2-HelloUser
cd ~/user-authentication-with-iap/3-HelloVerifiedUser
```

## Multiple VPC Networks

done before

## VPC Networks - Controlling Access

- Create the web servers
- Create the firewall rule
- Explore the Network and Security Admin roles

- Network Admin: Permissions to create, modify, and delete networking resources and list firewall rules, except for firewall rules and SSL certificates.
- Security Admin: Permissions to create, modify, and delete firewall rules and SSL certificates.

Create a service account
For Select a role, select Compute Engine > Compute Network Admin and click CONTINUE.

```bash
gcloud auth activate-service-account --key-file credentials.json
```

## HTTP Load Balancer with Cloud Armor

- Configure HTTP and health check firewall rules
- Configure instance templates and create instance groups
- Configure the HTTP Load Balancer
- Test the HTTP Load Balancer
- Denylist the siege-vm

![http-loadbalancer.png](http-loadbalancer.png)

```bash
gcloud beta compute --project=qwiklabs-gcp-03-a2b5fd2cc5d5 instance-templates create us-east1 --machine-type=e2-medium \
    --subnet=projects/qwiklabs-gcp-03-a2b5fd2cc5d5/regions/us-east1/subnetworks/default --network-tier=PREMIUM \
    --metadata=startup-script-url=gs://cloud-training/gcpnet/httplb/startup.sh --maintenance-policy=MIGRATE --service-account=526830266415-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-east1 --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=us-east1 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any --tags=http-server

gcloud beta compute --project=qwiklabs-gcp-03-a2b5fd2cc5d5 instance-templates create europe-west1 --machine-type=e2-medium \
    --subnet=projects/qwiklabs-gcp-03-a2b5fd2cc5d5/regions/europe-west1/subnetworks/default --network-tier=PREMIUM \
    --metadata=startup-script-url=gs://cloud-training/gcpnet/httplb/startup.sh --maintenance-policy=MIGRATE --service-account=526830266415-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=europe-west1 --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=europe-west1 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any --tags=http-server

gcloud compute --project=qwiklabs-gcp-03-a2b5fd2cc5d5 instance-groups managed create us-east1-mig --base-instance-name=us-east1-mig --template=us-east1 --size=1 --zone=us-east1-b

gcloud beta compute --project "qwiklabs-gcp-03-a2b5fd2cc5d5" instance-groups managed set-autoscaling "us-east1-mig" --zone "us-east1-b" --cool-down-period "45" --max-num-replicas "5" --min-num-replicas "1" --target-cpu-utilization "0.8" --mode "on"

gcloud compute --project=qwiklabs-gcp-03-a2b5fd2cc5d5 instance-groups managed create europe-west1-mig --base-instance-name=europe-west1-mig --template=europe-west1 --size=1 --zone=europe-west1-b

gcloud beta compute --project "qwiklabs-gcp-03-a2b5fd2cc5d5" instance-groups managed set-autoscaling "europe-west1-mig" --zone "europe-west1-b" --cool-down-period "45" --max-num-replicas "5" --min-num-replicas "1" --target-cpu-utilization "0.8" --mode "on"
```

Navigation menu (mainmenu.png) > click Network Services > Load balancing, and then click Create load balancer.

Stress test the HTTP Load Balancer

```bash
sudo apt-get -y install siege
siege -c 250 http://$LB_IP
```

Navigation menu (mainmenu.png), click Network Services > Load balancing.

- Click Backends.
- Click http-backend.

Monitor the Frontend Location (Total inbound traffic) between North America and the two backends for 2 to 3 minutes.

In the Cloud Console, navigate to Navigation menu > Network Security > Cloud Armor.

- Click Create policy.

Navigation menu > Network Security > Cloud Armor.

- Click denylist-siege.
- Click Logs.
- Click View policy logs.

On the Logging page, in the first dropdown list, select Cloud HTTP Load Balancer > http-lb-forwarding-rule > http-lb.

## Create an Internal Load Balancer

- Configure HTTP and health check firewall rules
- Configure instance templates and create instance groups
- Configure the Internal Load Balancer

![internal-lb.png](internal-lb.png)

```bash
gcloud beta compute --project=qwiklabs-gcp-03-d0baa78f984e instance-templates create instance-template-1 --machine-type=e2-medium --subnet=projects/qwiklabs-gcp-03-d0baa78f984e/regions/us-central1/subnetworks/subnet-a --network-tier=PREMIUM --metadata=startup-script-url=gs://cloud-training/gcpnet/ilb/startup.sh --maintenance-policy=MIGRATE --service-account=1033803699245-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-central1 --tags=lb-backend --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-template-1 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

gcloud beta compute --project=qwiklabs-gcp-03-d0baa78f984e instance-templates create instance-template-2 --machine-type=e2-medium --subnet=projects/qwiklabs-gcp-03-d0baa78f984e/regions/us-central1/subnetworks/subnet-b --network-tier=PREMIUM --metadata=startup-script-url=gs://cloud-training/gcpnet/ilb/startup.sh --maintenance-policy=MIGRATE --service-account=1033803699245-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-central1 --tags=lb-backend --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-template-2 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any
```

## Build and Secure Networks in Google Cloud: Challenge Lab
