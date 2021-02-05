# Anthos: Service Mesh

## Installing the Istio on GKE Add-On with Kubernetes Engine

### Install and configure a cluster with the Istio on GKE Add-On

```shell
export CLUSTER_NAME=central
export CLUSTER_ZONE=us-central1-b
export CLUSTER_VERSION=latest
gcloud beta container clusters create $CLUSTER_NAME \
    --zone $CLUSTER_ZONE --num-nodes 4 \
    --machine-type "n1-standard-2" --image-type "COS" \
    --cluster-version=$CLUSTER_VERSION --enable-ip-alias \
    --addons=Istio --istio-config=auth=MTLS_STRICT
export GCLOUD_PROJECT=$(gcloud config get-value project)
gcloud container clusters get-credentials $CLUSTER_NAME \
    --zone $CLUSTER_ZONE --project $GCLOUD_PROJECT
kubectl create clusterrolebinding cluster-admin-binding \
    --clusterrole=cluster-admin \
    --user=$(gcloud config get-value core/account)
```

Verify

```shell
gcloud container clusters list
kubectl get service -n istio-system
# NAME                     TYPE           CLUSTER-IP      ...
# istio-citadel            ClusterIP      10.107.14.228   ...
# istio-galley             ClusterIP      10.107.5.4      ...
# istio-ingressgateway     LoadBalancer   10.107.6.234    ...
# istio-pilot              ClusterIP      10.107.7.175    ...
# istio-policy             ClusterIP      10.107.1.223    ...
# istio-sidecar-injector   ClusterIP      10.107.14.28    ...
# istio-telemetry          ClusterIP      10.107.7.170    ...
# promsd                   ClusterIP      10.107.15.229   ...
kubectl get pods -n istio-system
# NAME                                           READY  STATUS    ...
# istio-citadel-bc69f964d-5qlh4                  1/1    Running   ...
# istio-galley-869ddb78b9-cqnxx                  1/1    Running   ...
# istio-ingressgateway-75f6945d-blcwf            1/1    Running   ...
# istio-pilot-578595b884-8smrd                   2/2    Running   ...
# istio-policy-7975f787f5-qljrn                  2/2    Running   ...
# istio-security-post-install-1.4.6-gke.0-wtqf6  0/1    Completed ...
# istio-sidecar-injector-d5485f495-2rfxt         1/1    Running   ...
# istio-telemetry-688db7d55d-2vd84               2/2    Running   ...
# promsd-696bcc5b96-fwhhj                        2/2    Running   ...
```

### Deploy Bookinfo, an Istio-enabled multi-service application

Install istioctl

```shell
export LAB_DIR=$HOME/bookinfo-lab
export ISTIO_VERSION=1.4.6

mkdir $LAB_DIR
cd $LAB_DIR

curl -L https://git.io/getLatestIstio | ISTIO_VERSION=$ISTIO_VERSION sh -
cd ./istio-*

export PATH=$PWD/bin:$PATH
istioctl version
```

Deploy bookinfo

```shell
cat samples/bookinfo/platform/kube/bookinfo.yaml
istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)
```

Enable external access using an Istio Ingress Gateway

```shell
cat samples/bookinfo/networking/bookinfo-gateway.yaml
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

Verify

```shell
kubectl get services
# NAME          TYPE        CLUSTER-IP    ...
# details       ClusterIP   10.107.0.77   ...
# kubernetes    ClusterIP   10.107.0.1    ...
# productpage   ClusterIP   10.107.8.22   ...
# ratings       ClusterIP   10.107.13.29  ...
# reviews       ClusterIP   10.107.12.100 ...
kubectl get pods
# NAME                              READY     STATUS
# details-v1-1520924117-48z17       2/2       Running
# productpage-v1-560495357-jk1lz    2/2       Running
# ratings-v1-734492171-rnr5l        2/2       Running
# reviews-v1-874083890-f0qf0        2/2       Running
# reviews-v2-1343845940-b34q5       2/2       Running
# reviews-v3-1813607990-8ch52       2/2       Running
kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') \
    -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
# <title>Simple Bookstore App</title>
kubectl get gateway
# NAME               AGE
# bookinfo-gateway   20m
kubectl get svc istio-ingressgateway -n istio-system
# NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP
# istio-ingressgateway   LoadBalancer   10.107.15.123   104.154.143.236
export GATEWAY_URL=[EXTERNAL-IP]

curl -I http://${GATEWAY_URL}/productpage
```

## Installing Anthos Service Mesh on Google Kubernetes Engine

### Set up your project

```shell
gcloud config list
gcloud config set project [project_id]
gcloud services enable \
    container.googleapis.com \
    compute.googleapis.com \
    monitoring.googleapis.com \
    logging.googleapis.com \
    cloudtrace.googleapis.com \
    meshca.googleapis.com \
    meshtelemetry.googleapis.com \
    meshconfig.googleapis.com \
    iamcredentials.googleapis.com \
    anthos.googleapis.com \
    gkeconnect.googleapis.com \
    gkehub.googleapis.com \
    cloudresourcemanager.googleapis.com
export PROJECT_ID=$(gcloud config get-value project)
export PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} \
    --format="value(projectNumber)")
export CLUSTER_NAME=central
export CLUSTER_ZONE=us-central1-b
export WORKLOAD_POOL=${PROJECT_ID}.svc.id.goog
export MESH_ID="proj-${PROJECT_NUMBER}"
gcloud projects get-iam-policy $PROJECT_ID \
    --flatten="bindings[].members" \
    --filter="bindings.members:user:$(gcloud config get-value core/account 2>/dev/null)"
```

### Set up your GKE cluster

Connect for Anthos allows you to register any of your Kubernetes clusters to Google Cloud, even clusters running on-premises or on other cloud providers. This enables access to cluster and workload management features in Cloud Console, including a unified user interface to interact with your clusters.

```shell
gcloud config set compute/zone ${CLUSTER_ZONE}
gcloud beta container clusters create ${CLUSTER_NAME} \
    --machine-type=n1-standard-4 \
    --num-nodes=4 \
    --workload-pool=${WORKLOAD_POOL} \
    --enable-stackdriver-kubernetes \
    --subnetwork=default \
    --release-channel=regular \
    --labels mesh_id=${MESH_ID}
kubectl auth can-i '*' '*' --all-namespaces
kubectl create clusterrolebinding [BINDING_NAME] \
    --clusterrole cluster-admin --user [USER]
gcloud iam service-accounts create connect-sa
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
 --member="serviceAccount:connect-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
 --role="roles/gkehub.connect"
gcloud iam service-accounts keys create connect-sa-key.json \
  --iam-account=connect-sa@${PROJECT_ID}.iam.gserviceaccount.com
gcloud container hub memberships register ${CLUSTER_NAME}-connect \
  --gke-cluster=${CLUSTER_ZONE}/${CLUSTER_NAME}  \
  --service-account-key-file=./connect-sa-key.json
```

### Prepare to install Anthos Service Mesh

```shell
curl --request POST \
  --header "Authorization: Bearer $(gcloud auth print-access-token)" \
  --data '' \
  https://meshconfig.googleapis.com/v1alpha1/projects/${PROJECT_ID}:initialize
curl -LO https://storage.googleapis.com/gke-release/asm/istio-1.6.11-asm.1-linux-amd64.tar.gz
curl -LO https://storage.googleapis.com/gke-release/asm/istio-1.6.11-asm.1-linux-amd64.tar.gz.1.sig
openssl dgst -verify /dev/stdin -signature istio-1.6.11-asm.1-linux-amd64.tar.gz.1.sig istio-1.6.11-asm.1-linux-amd64.tar.gz <<'EOF'
-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEWZrGCUaJJr1H8a36sG4UUoXvlXvZ
wQfk16sxprI2gOJ2vFFggdq3ixF2h4qNBt0kI7ciDhgpwS8t+/960IsIgw==
-----END PUBLIC KEY-----
EOF
tar xzf istio-1.6.11-asm.1-linux-amd64.tar.gz
cd istio-1.6.11-asm.1
export PATH=$PWD/bin:$PATH
```

### Preparing Resource Configuration Files

```shell
sudo apt-get install google-cloud-sdk-kpt
mkdir ${CLUSTER_NAME}
cd ${CLUSTER_NAME}
kpt pkg get https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages@1.6.8-asm.9 asm
cd asm
kpt cfg set asm gcloud.container.cluster ${CLUSTER_NAME}
kpt cfg set asm gcloud.project.environProjectNumber ${PROJECT_NUMBER}
kpt cfg set asm gcloud.core.project ${PROJECT_ID}
kpt cfg set asm gcloud.compute.location ${CLUSTER_ZONE}
kpt cfg set asm anthos.servicemesh.profile asm-gcp
```

### Install Anthos Service Mesh

```shell
istioctl install -f asm/cluster/istio-operator.yaml
kubectl wait --for=condition=available --timeout=600s deployment \
    --all -n istio-system
asmctl validate
kubectl label namespace default istio-injection=enabled --overwrite
```

### Deploy Bookinfo, an Istio-enabled multi-service application

refer previous lab

### Use the Bookinfo application

```shell
sudo apt install siege
siege http://${GATEWAY_URL}/productpage
```

### Evaluate service performance using the Anthos Service Mesh dashboard

play UI

## Observing Services using Prometheus, Grafana, Jaeger, and Kiali

### Complete cluster configuration

```shell
export CLUSTER_NAME=central
export CLUSTER_ZONE=us-central1-b
export GCLOUD_PROJECT=$(gcloud config get-value project)
gcloud container clusters get-credentials $CLUSTER_NAME \
    --zone $CLUSTER_ZONE --project $GCLOUD_PROJECT
```

### Understand the installation of the Istio Telemetry Add-Ons

### Query Istio metrics with Prometheus

```shell
cat > ~/metrics.yaml <<EOF
# Configuration for metric instances
apiVersion: config.istio.io/v1alpha2
kind: instance
metadata:
  name: doublerequestcount
  namespace: istio-system
spec:
  compiledTemplate: metric
  params:
    value: "2" # count each request twice
    dimensions:
      reporter: conditional((context.reporter.kind | "inbound") == "outbound", "client", "server")
      source: source.workload.name | "unknown"
      destination: destination.workload.name | "unknown"
      message: '"twice the fun!"'
    monitored_resource_type: '"UNSPECIFIED"'
---
# Configuration for a Prometheus handler
apiVersion: config.istio.io/v1alpha2
kind: handler
metadata:
  name: doublehandler
  namespace: istio-system
spec:
  compiledAdapter: prometheus
  params:
    metrics:
    - name: double_request_count # Prometheus metric name
      instance_name: doublerequestcount.instance.istio-system # Mixer instance name (fully-qualified)
      kind: COUNTER
      label_names:
      - reporter
      - source
      - destination
      - message
---
# Rule to send metric instances to a Prometheus handler
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: doubleprom
  namespace: istio-system
spec:
  actions:
  - handler: doublehandler
    instances: [ doublerequestcount ]
EOF
kubectl apply -f ~/metrics.yaml
```

Generate metrics data by sending traffic to the application

```shell
kubectl get svc istio-ingressgateway -n istio-system
# NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP
# istio-ingressgateway   LoadBalancer   10.107.15.123   104.154.143.236
export GATEWAY_URL=[external-IP]

for n in `seq 1 9`; do curl -s -o /dev/null http://$GATEWAY_URL/productpage; done
```

Preview with prometheus

```shell
kubectl -n istio-system port-forward \
    $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') 9090:9090
```

Open the Prometheus UI.

Click the Web preview button in the Cloud Shell toolbar, then Preview on Port 8080. Change URL 8080 to 9090

Enter this Expression to show total requests to productpage: istio_requests_total{destination_service="productpage.default.svc.cluster.local"}

If you have time, add more graphs, and try other expressions, like `rate(istio_requests_total{destination_service=~"productpage.*", response_code="200"}[5m])`.

Create a graph for the new metric:

In the Prometheus UI:

- Click Add Graph.
- Enter istio_double_request_count for Expression.
- Click Execute.
- Click the Graph tab, to see visualizations.

### Visualize Istio metrics with Grafana

```shell
kubectl -n istio-system port-forward \
    $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000
```

Generate metrics data by sending traffic to the application

```shell
kubectl get svc istio-ingressgateway -n istio-system
# NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP
# istio-ingressgateway   LoadBalancer   10.107.15.123   104.154.143.236
export GATEWAY_URL=[external-IP]

for n in `seq 1 9`; do curl -s -o /dev/null http://$GATEWAY_URL/productpage; done
```

### Generate and visualize traces with Jaeger

```shell
kubectl get svc istio-ingressgateway -n istio-system
# NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP
# istio-ingressgateway   LoadBalancer   10.107.15.123   104.154.143.236
export GATEWAY_URL=[external-IP]

for n in `seq 1 99`; do curl -s -o /dev/null http://$GATEWAY_URL/productpage; done
```

Open the Jaeger UI

```shell
kubectl -n istio-system port-forward \
    $(kubectl -n istio-system get pod -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 20001:16686            
```

### Visualize your service Mesh with Kiali

```shell
kubectl -n istio-system port-forward \
    $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001
```

## Traffic Management with Anthos Service Mesh

https://github.com/istio/istio/tree/master/samples/bookinfo

### Review Traffic Management use cases

- Example: Traffic Splitting
- Example: Timeouts
- Example: Retries
- Example: Fault injection: inserting delays
- Example: Fault injection: inserting aborts
- Example: Conditional routing: Based on source labels
- Example: Complete lab setup

### Complete lab setup

```shell
export CLUSTER_NAME=central
export CLUSTER_ZONE=us-central1-b
export GCLOUD_PROJECT=$(gcloud config get-value project)
gcloud container clusters get-credentials $CLUSTER_NAME \
    --zone $CLUSTER_ZONE --project $GCLOUD_PROJECT
sudo apt install siege
```

### Understand ingress configuration using an Istio Gateway

```shell
export GATEWAY_URL=$(kubectl get svc istio-ingressgateway -o=jsonpath='{.status.loadBalancer.ingress[0].ip}' -n istio-system)
echo The gateway address is $GATEWAY_URL
siege http://${GATEWAY_URL}/productpage
```

Explore using the ingress gateway with your application

```shell
export CLUSTER_NAME=central
export CLUSTER_ZONE=us-central1-b
export GCLOUD_PROJECT=$(gcloud config get-value project)
gcloud container clusters get-credentials $CLUSTER_NAME \
--zone $CLUSTER_ZONE --project $GCLOUD_PROJECT
export GATEWAY_URL=$(kubectl get svc istio-ingressgateway \
-o=jsonpath='{.status.loadBalancer.ingress[0].ip}' -n istio-system)
kubectl exec -it \
    $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') \
    -c ratings -- curl productpage:9080/productpage \
    | grep -o "<title>.*</title>"
kubectl describe gateway bookinfo-gateway
kubectl describe virtualservices bookinfo
```

### Use the Anthos Service Mesh dashboard view routing to multiple versions

Check UI

### Download Anthos Service Mesh with sample configurations

```shell
curl -LO https://storage.googleapis.com/gke-release/asm/istio-1.6.8-asm.9-linux-amd64.tar.gz
tar xzf istio-1.6.8-asm.9-linux-amd64.tar.gz
cd istio-1.6.8-asm.9
export PATH=$PWD/bin:$PATH
kubectl get virtualservices
kubectl describe virtualservices
kubectl get destinationrules
```

### Apply default destination rules, for all available versions

```shell
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
kubectl get destinationrules
```

### Apply virtual services to route by default to only one version

```shell
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
kubectl get virtualservices
echo $GATEWAY_URL
```

### Route to a specific version of a service based on user identity

```shell
cat ./samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
kubectl get virtualservice reviews
curl -c cookies.txt -F "username=jason" -L -X \
    POST http://$GATEWAY_URL/login
cookie_info=$(grep -Eo "session.*" ./cookies.txt)
cookie_name=$(echo $cookie_info | cut -d' ' -f1)
cookie_value=$(echo $cookie_info | cut -d' ' -f2)
siege -c 5 http://$GATEWAY_URL/productpage \
    --header "Cookie: $cookie_name=$cookie_value"
kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

### Shift traffic gradually from one version of a microservice to another

```shell
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
kubectl apply -f \
      samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

## Managing Policies and Security with Istio and Citadel

### Complete cluster configuration

```shell
export CLUSTER_NAME=central
export CLUSTER_ZONE=us-central1-b
export GCLOUD_PROJECT=$(gcloud config get-value project)
gcloud container clusters get-credentials $CLUSTER_NAME \
    --zone $CLUSTER_ZONE --project $GCLOUD_PROJECT
export LAB_DIR=$HOME/security-lab
export ISTIO_VERSION=1.5.2

mkdir $LAB_DIR
cd $LAB_DIR
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=$ISTIO_VERSION sh -
cd ./istio-*
export PATH=$PWD/bin:$PATH
istioctl version
```

### Deploy Hipster Shop

```shell
cd $LAB_DIR

git clone https://github.com/GoogleCloudPlatform/istio-samples.git
cd istio-samples/security-intro
mkdir ./hipstershop
curl -o ./hipstershop/kubernetes-manifests.yaml https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/kubernetes-manifests.yaml
cat ./hipstershop/kubernetes-manifests.yaml
istioctl kube-inject -f hipstershop/kubernetes-manifests.yaml -o ./hipstershop/kubernetes-manifests-withistio.yaml
cat ./hipstershop/kubernetes-manifests-withistio.yaml
kubectl apply -f ./hipstershop/kubernetes-manifests-withistio.yaml
curl -o ./hipstershop/istio-manifests.yaml https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/istio-manifests.yaml
cat hipstershop/istio-manifests.yaml
kubectl apply -f hipstershop/istio-manifests.yaml
kubectl get -n istio-system service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
curl http://[EXTERNAL_IP]
```

### Understand authentication and enable service-to-service authentication with mTLS

```shell
kubectl -n istio-system port-forward \
    $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 8080:20001
```

Open the Kiali UI:

- Click the Web preview button in the Cloud Shell toolbar
- Click Preview on Port 8080

You will see Log in Kiali. Enter:

- Username: admin
- Password: admin
- Click Log In

Click the Display dropdown, and check the Security box.
Notice that in permissive mode, no locks appear on the service graph.

Enable mTLS for one service: frontend

```shell
export LAB_DIR=$HOME/security-lab
cd $LAB_DIR/istio-samples/security-intro
cat ./manifests/mtls-frontend.yaml
kubectl apply -f ./manifests/mtls-frontend.yaml
kubectl exec $(kubectl get pod -l app=productcatalogservice -o jsonpath={.items..metadata.name}) -c istio-proxy -- curl http://frontend:80/ -o /dev/null -s -w '%{http_code}\n'
```

Exit code 56 means "failure to receive network data" from the curl command.

```shell
kubectl exec $(kubectl get pod -l app=productcatalogservice -o jsonpath={.items..metadata.name}) -c istio-proxy \
-- curl https://frontend:80/ -o /dev/null -s -w '%{http_code}\n'  --key /etc/certs/key.pem --cert /etc/certs/cert-chain.pem --cacert /etc/certs/root-cert.pem -k
```

Enable mTLS for an entire namespace: default

```shell
cat ./manifests/mtls-default-ns.yaml
kubectl apply -f ./manifests/mtls-default-ns.yaml
```

### Enable end-user JWT authentication alongside mTLS

```shell
cat ./manifests/jwt-frontend-request.yaml
kubectl apply -f ./manifests/jwt-frontend-request.yaml
TOKEN=helloworld; echo $TOKEN
kubectl exec $(kubectl get pod -l app=productcatalogservice -o jsonpath={.items..metadata.name}) -c istio-proxy \
-- curl  http://frontend:80/ -o /dev/null --header "Authorization: Bearer $TOKEN" -s -w '%{http_code}\n'
# error
kubectl exec $(kubectl get pod -l app=productcatalogservice -o jsonpath={.items..metadata.name}) -c istio-proxy \
-- curl  https://frontend:80/ -o /dev/null -s -w '%{http_code}\n' \
--key /etc/certs/key.pem --cert /etc/certs/cert-chain.pem --cacert /etc/certs/root-cert.pem -k
# ok when no token by default
cat ./manifests/jwt-frontend-authz.yaml
kubectl apply -f manifests/jwt-frontend-authz.yaml
kubectl exec $(kubectl get pod -l app=productcatalogservice -o jsonpath={.items..metadata.name}) -c istio-proxy \
-- curl  https://frontend:80/ -o /dev/null -s -w '%{http_code}\n' \
--key /etc/certs/key.pem --cert /etc/certs/cert-chain.pem --cacert /etc/certs/root-cert.pem -k
# 403
TOKEN=$(curl -k https://raw.githubusercontent.com/istio/istio/release-1.4/security/tools/jwt/samples/demo.jwt -s); echo $TOKEN
kubectl exec $(kubectl get pod -l app=productcatalogservice -o jsonpath={.items..metadata.name}) -c istio-proxy \
-- curl --header "Authorization: Bearer $TOKEN" https://frontend:80/ -o /dev/null -s -w '%{http_code}\n' \
--key /etc/certs/key.pem --cert /etc/certs/cert-chain.pem --cacert /etc/certs/root-cert.pem -k
# 200
kubectl delete -f manifests/jwt-frontend-authz.yaml
kubectl delete -f manifests/jwt-frontend-request.yaml
```

### Understand Istio authorization and enable frontend authorization

Enable authorization for one service: frontend

```shell
cat ./manifests/authz-frontend.yaml
kubectl apply -f ./manifests/authz-frontend.yaml
kubectl exec $(kubectl get pod -l app=productcatalogservice -o jsonpath={.items..metadata.name}) -c istio-proxy \
-- curl  --header "Authorization: Bearer $TOKEN" https://frontend:80/ -o /dev/null -s -w '%{http_code}\n' \
--key /etc/certs/key.pem --cert /etc/certs/cert-chain.pem --cacert /etc/certs/root-cert.pem -k
# 403
kubectl exec $(kubectl get pod -l app=productcatalogservice -o jsonpath={.items..metadata.name}) -c istio-proxy \
-- curl --header "Authorization: Bearer $TOKEN" --header "hello:world"  \
   https://frontend:80/ -o /dev/null -s -w '%{http_code}\n' \
  --key /etc/certs/key.pem --cert /etc/certs/cert-chain.pem --cacert /etc/certs/root-cert.pem -k
# 200
```
