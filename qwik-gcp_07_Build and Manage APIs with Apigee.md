# Build and Manage APIs with Apigee

## Introducing API Management on Top of a Legacy Platform

Apigee Account - If you don't have an account, sign-up for a trial account at https://login.apigee.com/sign_up. You will get notified through the registered email once the account is created. This could take a few minutes.

REST Client - To invoke the APIs you can use curl, or use the simplified REST Client at https://apigee-restclient.appspot.com/.

GCP

1. Create Service Account with logs write permission
2. Create Credential key

Apigee

1. Enable extension Google Stackdriver logger
2. Deploy test environment with crendential key
3. Create API proxy -> reverse proxy -> no auth
4. Add policies to your API: API proxy -> Develop
    - Select the Spike Arrest policy from the TRAFFIC MANAGEMENT category.
    - Next, add the "Quota" policy to follow the "Verify API Key" policy. Keep the default Name and Display Name values when adding the policy.
    - Select the Quota policy from the designer view so its configuration appears in the code view below. Replace the codel with the following:
    - click Extension Callout. Select your extension name from the Extension dropdown menu. Select Log from the Actions dropdown. Click Add.
5. Create API product
6. Create developer app

## Leverage Apigee to Modernize Exposure & Secure Access

1. Create SOAP Service: https://hipster-soap.apigee.io/hipster?wsdl
2. Export your API Proxy
3. Create a cloud storage bucket & upload your app data in it
4. Make your bucket publicly accessible
5. Securely access the REST API and trace the call flow

## Self Service API Discovery & Sign Up Experience

Save and use the apikey for this lab: eRlOKErRKNHWH35FF2exp8r7EEzLwSPb

## Content Aggregation via Apigee - Bring in Google Hosted API Content

## Conditional Routing of APIs Based on Feature Flag

Save and use the apikey for this lab: Na9BFTMFL5LAyAUQpJvV89ibAEdpmdkX
Fgq3kp6yI90EGwxACG4EuQ4Rb5K4hJIy