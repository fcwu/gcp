# Intro to ML: Image Processing

- [Machine Learning APIs](https://cloud.google.com/products/ai/): use pretrained APIs like Cloud Vision for common ML tasks.
- [AutoML](https://cloud.google.com/automl/): create custom ML models with no coding needed.
- [BigQuery ML](https://cloud.google.com/bigquery/#bigqueryml): use your SQL knowledge to build ML models quickly right where your data already lives in BigQuery.
- [AI Platform](https://cloud.google.com/ml-engine/): build your own custom ML models and put them in production with Google's infrastructure.
- [Vision API](https://cloud.google.com/vision)
- [API explorer](https://cloud.google.com/vision/docs/reference/rest/v1/images/annotate?apix=true)

## AI Platform: Qwik Start

This lab gives you an introductory, end-to-end experience of training and prediction on AI Platform. The lab will use a census dataset to:

- Create a TensorFlow 2.x training application and validate it locally.
- Run your training job on a single worker instance in the cloud.
- Deploy a model to support prediction.
- Request an online prediction and see the response.

AI Platform > Notebook

```bash
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
```

In AI Platform Notebooks, navigate to training-data-analyst/self-paced-labs/ai-platform-qwikstart and open ai_platform_qwik_start.ipynb.

## Classify Images of Clouds in the Cloud with AutoML Vision

Enable APIs & Services > Library > Cloud AutoML API.

Add IAM

```bash
export PROJECT_ID=$DEVSHELL_PROJECT_ID
export QWIKLABS_USERNAME=<USERNAME>
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="user:$QWIKLABS_USERNAME" \
    --role="roles/automl.admin"
gsutil mb -p $PROJECT_ID \
    -c standard    \
    -l us-central1 \
    gs://$PROJECT_ID-vcm/
```

Training Data

```bash
export BUCKET=$PROJECT_ID-vcm
gsutil -m cp -r gs://automl-codelab-clouds/* gs://${BUCKET}
gsutil cp gs://automl-codelab-metadata/data.csv .
sed -i -e "s/placeholder/${BUCKET}/g" ./data.csv
gsutil cp ./data.csv gs://${BUCKET}
```

Vision > + NEW DATASET

Check Label, Train, Evaluate, Confusion Matrix, Prediction

## Detect Labels, Faces, and Landmarks in Images with the Cloud Vision API

In this lab, we will send images to the Vision API and see it detect objects, faces, and landmarks.

APIs & services > Credentials > Create Crendentials > API Key

Feature: https://cloud.google.com/vision/docs/reference/rest/v1/Feature

## Extract, Analyze, and Translate Text from Images with the Cloud ML APIs

- Creating a Vision API request and calling the API with curl
- Using the text detection (OCR) method of the Vision API
- Using the Translation API to translate text from your image
- Using the Natural Language API to analyze the text

APIs & services > Credentials > Create Crendentials > API Key
