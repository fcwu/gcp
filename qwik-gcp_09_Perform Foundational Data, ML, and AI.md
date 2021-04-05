# Perform Foundational Data, ML, and AI Tasks in Google Cloud

## AI Platform: Qwik Start

    Click on the Navigation Menu and navigate to AI Platform, then to Notebooks.

git clone https://github.com/GoogleCloudPlatform/training-data-analyst

In AI Platform Notebooks, navigate to training-data-analyst/self-paced-labs/ai-platform-qwikstart and open ai_platform_qwik_start.ipynb.

notebook

## Dataprep: Qwik Start

Dataflow
from pubsub to BigQuery
create bucket in Storage for temporary data

## Dataproc: Qwik Start - Console

Spark, Hardoop...

## Video Intelligence: Qwik Start

gcloud iam service-accounts create quickstart
gcloud iam service-accounts keys create key.json --iam-account quickstart@qwiklabs-gcp-02-ee8e80a7bb13.iam.gserviceaccount.com
gcloud auth activate-service-account --key-file key.json
gcloud auth print-access-token
ya29.c.KqQB4wdotUThbqN16nB-qxtF9R6oFl9E0vv63-CfJiaROJ0yb46kvDg9gNGiYHCm1h2EHMgaVSrXp2aS4nnvVkqZ1Q94DjEmSGdP8BanCbKL49BeRz10VllpS4Z1aG08zrTdRHZ16Pz-GFv7EF3qgt5H6keZLf4uyWq5kzskEITJR66ev4S22pape6upYW81R9U8iwWnooboO0m-I4hg1OkwdXgH1uU
cat <<EOF >request.json
{
"inputUri":"gs://spls/gsp154/video/train.mp4",
"features": [
"LABEL_DETECTION"
]
}
EOF
curl -s -H 'Content-Type: application/json' \
 -H 'Authorization: Bearer '$(gcloud auth print-access-token)'' \
    'https://videointelligence.googleapis.com/v1/videos:annotate' \
    -d @request.json
{
  "name": "projects/736131497822/locations/asia-east1/operations/17957116887298426869"
}
curl -s -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer '$(gcloud auth print-access-token)'' \
 'https://videointelligence.googleapis.com/v1/operations/projects/736131497822/locations/asia-east1/operations/17957116887298426869'
