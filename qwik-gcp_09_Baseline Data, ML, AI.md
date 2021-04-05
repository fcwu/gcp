# Baseline: Data, ML, AI

[qwikilabs](https://www.qwiklabs.com/quests/34)

## AI Platform: Qwik Start

- Click on the Navigation Menu and navigate to AI Platform, then to Notebooks.
- Click Open JupyterLab. A JupyterLab window will open in a new tab.

```bash
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
```

In AI Platform Notebooks, navigate to training-data-analyst/self-paced-labs/ai-platform-qwikstart and open ai_platform_qwik_start.ipynb.

## Dataprep: Qwik Start

- Create a Cloud Storage bucket in your project
- Select Navigation menu > Dataprep.

## Dataflow: Qwik Start - Templates

### Create a Cloud BigQuery Dataset and Table Using Cloud Shell

```bash
bq mk taxirides
bq mk \
--time_partitioning_field timestamp \
--schema ride_id:string,point_idx:integer,latitude:float,longitude:float,\
timestamp:timestamp,meter_reading:float,meter_increment:float,ride_status:string,\
passenger_count:integer -t taxirides.realtime
```

### Run the Pipeline

From the Navigation menu find the Big Data section and click on Dataflow.

Click on + Create job from template at the top of the screen.

Enter a Job name for your Cloud Dataflow job.

Under Dataflow Template, select the Pub/Sub Topic to BigQuery template.

Under Input Pub/Sub topic, enter:

```bash
projects/pubsub-public-data/topics/taxirides-realtime
```

Under BigQuery output table, enter the name of the table that was created:

```bash
<myprojectid>:taxirides.realtime
```

Add your bucket as Temporary Location:

```bash
gs://Your_Bucket_Name/temp
```

## Dataproc: Qwik Start - Console

https://www.qwiklabs.com/focuses/12662

## Cloud Natural Language API: Qwik Start

- Syntax Analysis: Extract tokens and sentences, identify parts of speech (PoS) and create dependency parse trees for each sentence.
- Entity Recognition: Identify entities and label by types such as person, organization, location, events, products and media.
- Sentiment Analysis: Understand the overall sentiment expressed in a block of text.
- Content Classification: Classify documents in predefined 700+ categories.
- Multi-Language: Enables you to easily analyze text in multiple languages including English, Spanish, Japanese, Chinese (Simplified and Traditional), French, German, Italian, Korean and Portuguese.
- Integrated REST API: Access via REST API. Text can be uploaded in the request or integrated with Cloud Storage.

## Google Cloud Speech API: Qwik Start

## Video Intelligence: Qwik Start

## Reinforcement Learning: Qwik Start

https://www.qwiklabs.com/focuses/12671

- Understand the fundamental concepts of reinforcement learning.
- Create an AI Platform Tensorflow 2.1 Notebook.
- Clone the sample repository from the training data analyst repo found on Github.
- Read, understand, and run the steps found in the notebook.

From the left-hand navigation menu, select AI Platform > Notebooks. Then from the top-hand menu, select + New Instance > TensorFlow 2.x > Without GPUs:

```bash
git clone https://github.com/GoogleCloudPlatform/training-data-analyst.git
```

Terminal > Wait for this command to propagate. Then from the left-hand menu, select training-data-analyst> quests> rl > early_rl > early_rl.ipynb. This will open a new tab.

Run through the notebook
