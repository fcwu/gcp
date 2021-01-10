# Machine Learning APIs

## Introduction to APIs in Google 

https://developers.google.com/oauthplayground/

## Extract, Analyze, and Translate Text from Images with the Cloud ML APIs 

- Creating a Vision API request and calling the API with curl
- Using the text detection (OCR) method of the Vision API
- Using the Translation API to translate text from your image
- Using the Natural Language API to analyze the text

API Key: APIs & services > Credentials > Create Crendential > API Key

```bash
export API_KEY=<YOUR_API_KEY>
```

Upload an image to a cloud storage bucket: `wget https://cdn.qwiklabs.com/cBoI5P4dZ6k%2FAr5Mv7eME%2F0fCb4G6nIGB0odCXzpEa4%3D -o sign.png`

```bash
gsutil mb gs://doro-storage/
gsutil cp sign.png gs://doro-storage/
gsutil acl ch -u AllUsers:R gs://doro-storage/sign.png
```

Create your Vision API request

```bash
cat >ocr-request.json <<EOF
{
  "requests": [
      {
        "image": {
          "source": {
              "gcsImageUri": "gs://doro-storage/sign.jpg"
          }
        },
        "features": [
          {
            "type": "TEXT_DETECTION",
            "maxResults": 10
          }
        ]
      }
  ]
}
EOF
curl -s -X POST -H "Content-Type: application/json" --data-binary @ocr-request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY} -o ocr-response.json
cat >translation-request.json<<EOF
{
  "q": "your_text_here",
  "target": "en"
}
EOF
STR=$(jq .responses[0].textAnnotations[0].description ocr-response.json) && STR="${STR//\"}" && sed -i "s|your_text_here|$STR|g" translation-request.json
curl -s -X POST -H "Content-Type: application/json" --data-binary @translation-request.json https://translation.googleapis.com/language/translate/v2?key=${API_KEY} -o translation-response.json
```

### Analyzing the image's text with the Natural Language API

```bash
cat >nl-request <<EOF
{
  "document":{
    "type":"PLAIN_TEXT",
    "content":"your_text_here"
  },
  "encodingType":"UTF8"
}
EOF
STR=$(jq .data.translations[0].translatedText  translation-response.json) && STR="${STR//\"}" && sed -i "s|your_text_here|$STR|g" nl-request.json
curl "https://language.googleapis.com/v1/documents:analyzeEntities?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @nl-request.json
```

result

```bash
{
  "entities": [
    {
      "name": "PUBLIC PROPERTY",
      "type": "OTHER",
      "metadata": {},
      "salience": 0.4682728,
      "mentions": [
        {
          "text": {
            "content": "PUBLIC PROPERTY",
            "beginOffset": 0
          },
          "type": "PROPER"
        }
      ]
    },
```

mid: https://developers.google.com/knowledge-graph/

language list: https://cloud.google.com/translate/docs/languages

## Classify Text into Categories with the Natural Language API 

What you'll learn

- Creating a Natural Language API request and calling the API with curl
- Using the NL API's text classification feature
- Using text classification to understand a dataset of news articles

- Know what category the sentense is
- Use Python to retrieve sentense in BigQuery and analyze the category by Natural language API

## Detect Labels, Faces, and Landmarks in Images with the Cloud Vision API 

Done before

## Entity and Sentiment Analysis with the Natural Language API 

What you'll learn

- Creating a Natural Language API request and calling the API with curl
- Extracting entities and running sentiment analysis on text with the Natural Language API
- Performing linguistic analysis on text with the Natural Language API
- Creating a Natural Language API request in a different language

For each entity in the response, you get the entity type, the associated Wikipedia URL if there is one, the salience, and the indices of where this entity appeared in the text.

Sentiment analysis with the Natural Language API

- score - is a number from -1.0 to 1.0 indicating how positive or negative the statement is.
- magnitude - is a number ranging from 0 to infinity that represents the weight of sentiment expressed in the statement, regardless of being positive or negative.

Analyzing syntax and parts of speech

The Natural Language API also supports languages other than English ([full list here](https://cloud.google.com/natural-language/docs/languages)).

## Awwvision: Cloud Vision API from a Kubernetes Cluster

create k8s cluster

```bash
gcloud config set compute/zone us-central1-a
gcloud container clusters create awwvision \
    --num-nodes 2 \
    --scopes cloud-platform
gcloud container clusters get-credentials awwvision
kubectl cluster-info
sudo apt-get update
sudo apt-get install virtualenv
virtualenv -p python3 venv
source venv/bin/activate
gsutil -m cp -r gs://spls/gsp066/cloud-vision .
cd cloud-vision/python/awwvision
make all
```

### Check the Kubernetes resources on the cluster

```bash
# kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
awwvision-webapp-vwmr1   1/1       Running   0          1m
awwvision-worker-oz6xn   1/1       Running   0          1m
awwvision-worker-qc0b0   1/1       Running   0          1m
awwvision-worker-xpe53   1/1       Running   0          1m
redis-master-rpap8       1/1       Running   0          2m
# kubectl get deployments -o wide
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS         IMAGES                                SELECTOR
awwvision-webapp   1         1         1            1           1m        awwvision-webapp   gcr.io/your-project/awwvision-webapp   app=awwvision,role=frontend
awwvision-worker   3         3         3            3           1m        awwvision-worker   gcr.io/your-project/awwvision-worker   app=awwvision,role=worker
redis-master       1         1         1            1           1m        redis-master       redis                                 app=redis,role=master
# kubectl get svc awwvision-webapp
NAME               CLUSTER_IP      EXTERNAL_IP    PORT(S)   SELECTOR                      AGE
awwvision-webapp   10.163.250.49   23.236.61.91   80/TCP    app=awwvision,role=frontend   13m:w
```
