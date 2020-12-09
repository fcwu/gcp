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