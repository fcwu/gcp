# Memo

gcloud compute

```bash
gcloud config list
gcloud compute zones list
gcloud compute images list --project ubuntu-os-cloud --no-standard-images
gcloud compute instances list
gcloud compute instances create instance-1 \
    --preemptible \
    --boot-disk-size 20GB \
    --image-family ubuntu-minimal-2004-lts \
    --image-project ubuntu-os-cloud \
    --machine-type e2-micro \
    --tags kubernetes-the-hard-way,controller
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --can-ip-forward \
    --subnet kubernetes \
    --async \
gcloud compute machine-types list --filter="zone ~ asia-east1-a" --sort-by=guestCpus
gcloud compute machine-types describe n1-highmem-64 --zone asia-east1-a
gcloud compute machine-types list --format=json --limit 1
[
  {
    "creationTimestamp": "1969-12-31T16:00:00.000-08:00",
    "description": "Compute Optimized: 16 vCPUs, 64 GB RAM",
    "guestCpus": 16,
    "id": "801016",
    "imageSpaceGb": 0,
    "isSharedCpu": false,
    "kind": "compute#machineType",
    "maximumPersistentDisks": 128,
    "maximumPersistentDisksSizeGb": "263168",
    "memoryMb": 65536,
    "name": "c2-standard-16",
    "selfLink": "https://www.googleapis.com/compute/v1/projects/doro-pca/zones/us-central1-a/machineTypes/c2-standard-16",
    "zone": "us-central1-a"
  }
]
gcloud compute ssh instance-1
gcloud compute instances delete instance-1
gcloud compute addresses create dfa --project=doro-pca --region=asia-east1
gcloud compute instances add-access-config instance-1 --project=doro-pca --zone=asia-east1-a --address=IP_OF_THE_NEWLY_CREATED_STATIC_ADDRESS
```

access metadata

```bash
curl http://metadata.google.internal/computeMetadata/v1/instance/guest-attributes/namespace/ -H "Metadata-Flavor: Google"
```

machine type

- f1-micro
- g1-small
- e2-micro
- e2-small

command output

- filter: https://cloud.google.com/sdk/gcloud/reference/topic/filters?hl=zh-TW
- format: https://cloud.google.com/sdk/gcloud/reference/topic/formats?hl=zh-TW

GKE

```bash
gcloud beta container --project "doro-pca" clusters create "cluster-1" \
    --zone "asia-east1-b" --no-enable-basic-auth --cluster-version "1.18.6-gke.4801" --release-channel "rapid" \
    --machine-type "e2-medium" --image-type "COS" --disk-type "pd-standard" --disk-size "100" --metadata disable-legacy-endpoints=true \
    --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" \
    --num-nodes "3" --enable-stackdriver-kubernetes --enable-ip-alias \
    --network "projects/doro-pca/global/networks/default" --subnetwork "projects/doro-pca/regions/asia-east1/subnetworks/default" --default-max-pods-per-node "110" --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --enable-shielded-nodes
```