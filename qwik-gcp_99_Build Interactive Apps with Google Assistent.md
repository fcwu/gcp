# Build Interactive Apps with Google Assistant

## Google Assistant: Qwik Start - Dialogflow

- Create an Actions project
- Test your Understanding
- Set up Dialogflow
- Build a custom Dialogflow intent
- Check your Google Permission Settings

Google Assistant is a personal voice assistant that offers a host of actions and integrations. Actions is the central platform for developing Google Assistant applications. Actions work with a number of human-computer interaction suites, which simplifies conversational app development. Out of all the platforms, the most popular is Dialogflow, which uses an underlying machine learning (ML) and natural language understanding (NLU) schema to build rich Assistant applications.

http://console.actions.google.com/

Google Assistant: The virtual assistant that's found on smartphones, homes, and a host of other devices.
Actions: The developer platform that allows you to build Assistant applications.
Dialogflow: A platform that abstracts the complexities of NLU and ML to build conversational applications.
Action: an interaction built for Google Assistant that performs specific tasks based on user input.
Intent: the goal of the Action (e.g. generate quotes). An intent takes user input and channels it to trigger an event.
Agent (Dialogflow): a module that uses NLU and ML to transform user input into actionable data to be used by an Assistant application.

https://myaccount.google.com/activitycontrols

## Google Assistant: Customizing Templates

Actions templates allow you to build Google Assistant applications like trivia games or flashcard generators without writing a single line of code. You can integrate Speech Synthesis Markup Language (SSML) to give your application more character and make responses seem more life-like. Google Assistant has also added multilingual capabilities, which allows you to expand your application to a much a wider audience.

https://docs.google.com/spreadsheets/d/1N8fmNB2crfZRw3y_f-Lb57rAKz35hLCCG7yBnECrWzI/edit?usp=sharing

## Introduction to APIs in Google

https://developers.google.com/oauthplayground/

```
{  "name": "<YOUR_BUCKET_NAME>",
   "location": "us",
   "storageClass": "multi_regional"
}

export OAUTH2_TOKEN=<YOUR_TOKEN>
export PROJECT_ID=<YOUR_PROJECT_ID>
curl -X POST --data-binary @values.json \
    -H "Authorization: Bearer $OAUTH2_TOKEN" \
    -H "Content-Type: application/json" \
    "https://www.googleapis.com/storage/v1/b?project=$PROJECT_ID"

export OBJECT=<DEMO_IMAGE_PATH>
export BUCKET_NAME=<YOUR_BUCKET>
curl -X POST --data-binary @$OBJECT \
    -H "Authorization: Bearer $OAUTH2_TOKEN" \
    -H "Content-Type: image/png" \
    "https://www.googleapis.com/upload/storage/v1/b/$BUCKET_NAME/o?uploadType=media&name=demo-image"
```

## Google Assistant: Build an Application with Dialogflow and Cloud Functions

Training phrases

Actions and parameters section. Check the REQUIRED checkbox for the number parameter. This tells Dialogflow to not trigger the intent until the parameter is properly provided by the user. Now click on Define prompts for the number parameter (right-hand side) and provide a re-prompt phrase. Type in What's your lucky number?

Fulfillment section and click Enable fulfillment. Then click the Enable webhook call for this intent slider

Initialize and configure a Cloud Function

```
index.js
'use strict';

const {dialogflow} = require('actions-on-google');
const functions = require('firebase-functions');

const app = dialogflow({debug: true});

app.intent('make_name', (conv, {color, number}) => {
  conv.close(`Alright, your silly name is ${color} ${number}! ` +
    `I hope you like it. See you next time.`);
});

exports.sillyNameMaker = functions.https.onRequest(app);

package.json
{
  "name": "silly-name-maker",
  "description": "Find out your silly name!",
  "version": "0.0.1",
  "author": "Google Inc.",
  "engines": {
    "node": "8"
  },
  "dependencies": {
    "actions-on-google": "^2.0.0",
    "firebase-admin": "^4.2.1",
    "firebase-functions": "1.0.0"
  }
}
```

## Google Assistant: Build a Youtube Entertainment App

```
index.js
'use strict';

const {
  dialogflow,
  Image,
  Suggestions
} = require('actions-on-google');

const functions = require('firebase-functions');
const app = dialogflow({debug: true});

const API_KEY = '<YOUR_API_KEY_HERE>';

app.intent('youtube', (conv, {any}) => {
    var url = "https://www.googleapis.com/youtube/v3/search?part=snippet&maxResults=5&q=" + encodeURIComponent(any)+ "&type=video&order=viewCount&videoCategoryId=10&key=" + API_KEY;
    const axios = require('axios');
    return axios.get(url)
        .then(response => {
          var output = JSON.stringify(response.data);
          var song_fields = response.data.items[1];
          return song_fields;
      }).then(output => {
		      var song_title = output.snippet.title;
          song_title = song_title.replace(/&amp;/g, '&');
          song_title = song_title.replace(/&quot;/g, '\"');
          var song_link = JSON.stringify(output.id.videoId);
          var song_thumbnail = output.snippet.thumbnails.high.url;
          conv.ask(`Fetching your request...`)
          conv.ask(new Image({
              url: song_thumbnail,
              alt: 'Song thumbnail'
            }))
          conv.close(`The most popular song is: ` + song_title + `. The link to this song is: https://www.youtube.com/watch?v=` + song_link.slice(1, -1) + `. See you next time.`);
      })
});

exports.youtube = functions.https.onRequest(app);

package.json
{
  "name": "youtube",
  "description": "Get restaurant reviews.",
  "version": "0.0.1",
  "author": "Google Inc.",
  "engines": {
    "node": "8"
  },
  "dependencies": {
    "actions-on-google": "^2.0.0",
    "firebase-admin": "^4.2.1",
    "firebase-functions": "1.0.0",
    "axios": "0.16.2"
  }
}

## Google Assistant: Build a Restaurant Locator with the Places API 

AIzaSyCM79QGNXVdujO0opNV7KJPvYuDlUJSUWQ

index.js
'use strict';

const {
  dialogflow,
  Image,
  Suggestions
} = require('actions-on-google');

const functions = require('firebase-functions');
const app = dialogflow({debug: true});

function getMeters(i) {
     return i*1609.344;
}

app.intent('get_restaurant', (conv, {location, proximity, cuisine}) => {
      const axios = require('axios');
      var api_key = "<YOUR_API_KEY_HERE>";
      var user_location = JSON.stringify(location["street-address"]);
      var user_proximity;
      if (proximity.unit == "mi") {
        user_proximity = JSON.stringify(getMeters(proximity.amount));
      } else {
        user_proximity = JSON.stringify(proximity.amount * 1000);
      }
      var geo_code = "https://maps.googleapis.com/maps/api/geocode/json?address=" + encodeURIComponent(user_location) + "&region=<YOUR_REGION>&key=" + api_key;
      return axios.get(geo_code)
        .then(response => {
          var places_information = response.data.results[0].geometry.location;
          var place_latitude = JSON.stringify(places_information.lat);
          var place_longitude = JSON.stringify(places_information.lng);
          var coordinates = [place_latitude, place_longitude];
          return coordinates;
      }).then(coordinates => {
        var lat = coordinates[0];
        var long = coordinates[1];
        var place_search = "https://maps.googleapis.com/maps/api/place/findplacefromtext/json?input=" + encodeURIComponent(cuisine) +"&inputtype=textquery&fields=photos,formatted_address,name,opening_hours,rating&locationbias=circle:" + user_proximity + "@" + lat + "," + long + "&key=" + api_key;
        return axios.get(place_search)
        .then(response => {
            var photo_reference = response.data.candidates[0].photos[0].photo_reference;
            var address = JSON.stringify(response.data.candidates[0].formatted_address);
            var name = JSON.stringify(response.data.candidates[0].name);
            var photo_request = 'https://maps.googleapis.com/maps/api/place/photo?maxwidth=400&photoreference=' + photo_reference + '&key=' + api_key;
            conv.ask(`Fetching your request...`);
            conv.ask(new Image({
                url: photo_request,
                alt: 'Restaurant photo',
              }))
            conv.close(`Okay, the restaurant name is ` + name + ` and the address is ` + address + `. The following photo uploaded from a Google Places user might whet your appetite!`);
        })
    })
});

exports.get_restaurant = functions.https.onRequest(app);

package.json
{
  "name": "get_reviews",
  "description": "Get restaurant reviews.",
  "version": "0.0.1",
  "author": "Google Inc.",
  "engines": {
    "node": "8"
  },
  "dependencies": {
    "actions-on-google": "^2.0.0",
    "firebase-admin": "^4.2.1",
    "firebase-functions": "1.0.0",
    "axios": "0.16.2"
  }
}

## Build Interactive Apps with Google Assistant: Challenge Lab 

main.py
import random
import logging
import google.cloud.logging
from google.cloud import translate_v2 as translate
from flask import Flask, request, make_response, jsonify

def magic_eight_ball(request):

    client = google.cloud.logging.Client()
    client.get_default_handler()
    client.setup_logging()

    choices = [
        "It is certain.", "It is decidedly so.", "Without a doubt.",
        "Yes - definitely.", "You may rely on it.", "As I see it, yes.",
        "Most likely.", "Outlook good.", "Yes.","Signs point to yes.",
        "Reply hazy, try again.", "Ask again later.",
        "Better not tell you now.", "Cannot predict now.",
        "Concentrate and ask again.", "Don't count on it.",
        "My reply is no.", "My sources say no.", "Outlook not so good.",
        "Very doubtful."
    ]

    magic_eight_ball_response = random.choice(choices)

    logging.info(magic_eight_ball_response)

    return make_response(jsonify({'fulfillmentText': magic_eight_ball_response }))

requirements.txt
google-cloud-translate
google-cloud-logging
```

test by

```
{"responseId":"5f996cad-5dfc-40f2-a43d-b6b20e8b24d2-a14fa99c","queryResult":{"queryText":"Will I complete this challenge？","action":"input.unknown","parameters":{},"allRequiredParamsPresent":true,"outputContexts":[{"name":"projects/qwiklabs-gcp-02-b32433699489/agent/sessions/3767e11a-acf0-30f7-5a36-2e8e720f26cd/contexts/__system_counters__","lifespanCount":1,"parameters":{"no-input":0,"no-match":3}}],"intent":{"name":"projects/qwiklabs-gcp-02-b32433699489/agent/intents/80346ab5-5af8-4f52-bc0c-784529bf6990","displayName":"Default Fallback Intent","isFallback":true,"endInteraction":true},"intentDetectionConfidence":1,"languageCode":"en"},"originalDetectIntentRequest":{"payload":{}},"session":"projects/qwiklabs-gcp-02-b32433699489/agent/sessions/3767e11a-acf0-30f7-5a36-2e8e720f26cd"}
```

## google translate

```
    request_json = request.get_json()

    if request_json and 'queryResult' in request_json:
        question = request_json.get('queryResult').get('queryText')

    # try to identify the language
    language = 'en'
    translate_client = translate.Client()
    detected_language = translate_client.detect_language(question)
    if detected_language['language'] == 'und':
        language = 'en'
    elif detected_language['language'] != 'en':
        language = detected_language['language']

    # translate if not english
    if language != 'en':
        logging.info('translating from en to %s' % language)
        translated_text = translate_client.translate(
             magic_eight_ball_response, target_language=language)
        magic_eight_ball_response = translated_text['translatedText']
```