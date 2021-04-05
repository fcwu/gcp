# Perform Foundational Infrastructure Tasks in Google Cloud

[Perform Foundational Infrastructure Tasks in Google Cloud](https://www.qwiklabs.com/quests/118)

## Cloud Storage: Qwik Start - Cloud Console

Navigation menu > Storage > Browser. Click Create Bucket

## Cloud Storage: Qwik Start - CLI/SDK

[gsutil manual](https://cloud.google.com/storage/docs/gsutil?hl=zh-tw)

```bash
gsutil mb gs://YOUR-BUCKET-NAME/
wget --output-document ada.jpg https://upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Ada_Lovelace_portrait.jpg/800px-Ada_Lovelace_portrait.jpg
gsutil cp ada.jpg gs://YOUR-BUCKET-NAME
gsutil cp -r gs://doro-qwik/ada.jpg .
gsutil cp gs://YOUR-BUCKET-NAME/ada.jpg gs://YOUR-BUCKET-NAME/image-folder/
gsutil ls gs://YOUR-BUCKET-NAME
gsutil acl ch -u AllUsers:R gs://doro-qwik/ada.jpg
gsutil acl ch -d AllUsers gs://YOUR-BUCKET-NAME/ada.jpg
gsutil rm gs://YOUR-BUCKET-NAME/ada.jpg
```

## Cloud IAM

Google Cloud's Identity and Access Management (IAM) service lets you create and manage permissions for Google Cloud resources.

- roles/viewer: Permissions for read-only actions that do not affect state, such as viewing (but not modifying) existing resources or data.
- roles/editor: All viewer permissions, plus permissions for actions that modify state, such as changing existing resources.
- roles/owner: All editor permissions and permissions for the following actions: Manage roles and permissions for a project and all resources within the project. Set up billing for a project.
- roles/browser (beta): Read access to browse the hierarchy for a project, including the folder, organization, and Cloud IAM policy. This role doesn't include permission to view resources in the project.

## Cloud Monitoring: Qwik Start

Navigation menu > Monitoring.

    ```bash
    curl -sSO https://dl.google.com/cloudagents/add-monitoring-agent-repo.sh
    sudo bash add-monitoring-agent-repo.sh
    curl -sSO https://dl.google.com/cloudagents/add-logging-agent-repo.sh
    sudo bash add-logging-agent-repo.sh
    sudo apt-get update
    sudo apt-get install stackdriver-agent
    sudo apt-get install google-fluentd
    ```

Create an uptime check

- Title: Lamp Uptime Check, then click Next.
- Protocol: HTTP
- Resource Type: Instance
- Applies to: Single, lamp-1-vm
- Path: leave at default
- Check Frequency: 1 min

Create an alerting policy

- In the left menu, click Alerting, and then click Create Policy.
- Click Add Condition.
  - Set the following in the panel that opens, leave all other fields at the default value. _Target_: Start typing "VM" in the resource type and metric field, and then select:
  - Resource Type: VM Instance (gce_instance)
  - Metric: Type "network", and then select Network traffic (gce*instance+1). Be sure to choose the Network traffic resource with agent `googleapis.com/interface/traffic`: \_Configuration*
    - Condition: is above
    - Threshold: 500
    - For: 1 minute
- Click on drop down arrow next to Notification Channels, then click on Manage Notification Channels.

Create a dashboard and chart

1. In the left menu select Dashboards, and then Create Dashboard.
2. Name the dashboard Cloud Monitoring LAMP Qwik Start Dashboard.
3. Add the first chart
   1. Click Line option in Chart library.
   2. Name the chart title CPU Load.
   3. Set the Resource type to VM Instance.
   4. Set the Metric CPU load (1m). Refresh the tab to view the graph.

Navigation menu > Logging > Logs Viewer.

- Check the uptime check results and triggered alerts
- Check if alerts have been triggered

## Cloud Functions: Qwik Start - Console

What you'll do

- Create a simple cloud function
- Deploy and test the function
- View logs

Serverless in Javascript

```bash
mkdir gcf_hello_world
cd gcf_hello_world
nano index.js
```

index.js

```javascript
/**
 * Background Cloud Function to be triggered by Pub/Sub.
 * This function is exported by index.js, and executed when
 * the trigger topic receives a message.
 *
 * @param {object} data The event payload.
 * @param {object} context The event metadata.
 */
exports.helloWorld = (data, context) => {
  const pubSubMessage = data;
  const name = pubSubMessage.data
    ? Buffer.from(pubSubMessage.data, "base64").toString()
    : "Hello World";

  console.log(`My Cloud Function: ${name}`);
};
```

deploy

```shell
gsutil mb -p [PROJECT_ID] gs://[BUCKET_NAME]
gcloud functions deploy helloWorld \
  --stage-bucket [BUCKET_NAME] \
  --trigger-topic hello_world \
  --runtime nodejs8
gcloud functions describe helloWorld
DATA=$(printf 'Hello World!'|base64) && gcloud functions call helloWorld --data '{"data":"'$DATA'"}'
gcloud functions logs read helloWorld
```

## Google Cloud Pub/Sub: Qwik Start - Console

- Setting up Pub/Sub
- Add a subscription
- Publish a message to the topic
- View the message
- gcloud pubsub subscriptions pull --auto-ack MySub

## Google Cloud Pub/Sub: Qwik Start - Command Line

topic

```bash
gcloud pubsub topics create myTopic
gcloud pubsub topics create Test1
gcloud pubsub topics create Test2
gcloud pubsub topics list
gcloud pubsub topics delete Test1
gcloud pubsub topics delete Test2
```

subscription

```bash
gcloud pubsub subscriptions create --topic myTopic mySubscription
gcloud  pubsub subscriptions create --topic myTopic Test1
gcloud  pubsub subscriptions create --topic myTopic Test2
gcloud pubsub topics list-subscriptions myTopic
gcloud pubsub subscriptions delete Test1
gcloud pubsub subscriptions delete Test2
```

publish and pull

```bash
gcloud pubsub topics publish myTopic --message "Hello"
gcloud pubsub topics publish myTopic --message "Publisher's name is <YOUR NAME>"
gcloud pubsub topics publish myTopic --message "Publisher likes to eat <FOOD>"
gcloud pubsub topics publish myTopic --message "Publisher thinks Pub/Sub is awesome"
gcloud pubsub subscriptions pull mySubscription --auto-ack  # only one message
```

Now is an important time to note a couple features of the pull command that often trip developers up:

    - Using the pull command without any flags will output only one message, even if you are subscribed to a topic that has more held in it.
    - Once an individual message has been outputted from a particular subscription-based pull command, you cannot access that message again with the pull command.

```bash
gcloud pubsub topics publish myTopic --message "Publisher is starting to get the hang of Pub/Sub"
gcloud pubsub topics publish myTopic --message "Publisher wonders if all messages will be pulled"
gcloud pubsub topics publish myTopic --message "Publisher will have to test to find out"
gcloud pubsub subscriptions pull mySubscription --auto-ack --limit=3
```

example: https://github.com/googleapis/python-pubsub/tree/master/samples/snippets

## Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab

Task 1: Create a bucket
Task 2: Create a Pub/Sub topic
Task 3: Create the thumbnail Cloud Function
Task 4: Remove the previous cloud engineer
