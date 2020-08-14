This page: [go/chc-api-samples](http://go/chc-api-samples)

# Sample Requests

## Setup

```shell
PROJECT=<project-name>
LOCATION=us-central1
DATASET=test-dataset
HL7STORE=test-store
FHIRSTORE=test-fhir-store
# See https://g3doc.corp.google.com/cloud/healthcare/g3doc/api/api.md#testing for details. You can also obtain access tokens with other methods.
TOKEN=`oauth2l fetch --json <CLIENT_SECRENT.json> https://www.googleapis.com/auth/cloud-healthcare`
API_ENDPOINT_PREFIX=https://autopush-healthcare.sandbox.googleapis.com/v1beta1
```

Make sure your `<project-name`> exists and is granted access for autopush. If
not, you can use your dev instance
https://***{{USERNAME}}***-healthcare.sandbox.googleapis.com assuming you have
it set up.

## Dataset

**Create**

```shell
curl -X POST --data "dataset_id=${DATASET}" "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets?access_token=${TOKEN}"
```

**Delete**

```shell
curl -X DELETE "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}?access_token=${TOKEN}"
```

## HL7v2

**Create**

```shell
curl -X POST --data "{\"notification_config\":{\"pubsub_topic\":\"projects/${PROJECT}/topics/test\"}}" "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/hl7V2Stores?access_token=${TOKEN}&hl7_v2_store_id=${HL7STORE}" -H "Content-Type: application/json;charset=utf-8"
```

Note that the pubsub topic in the notification config needs to exist for future
calls to succeed. You can check the existence of the topics and create one if
necessary by running:

```shell
gcloud pubsub topics list --project=${PROJECT}
gcloud pubsub topics create <topic-name> --project=${PROJECT}
```

Once the pubsub topic is created, you also need to add
`cloud-healthcare-eng@system.gserviceaccount.com` as a `Pub/Sub Publisher` to
this topic, otherwise the API won't be able to publish to this topic. If you
have created your own service account for your test project
([steps here](https://g3doc.corp.google.com/cloud/healthcare/g3doc/api/service_accounts.md#service-accounts-in-dev-personal-instances)),
grant the publisher permission to that service account instead.

**Update**

```shell
curl -X PATCH --data "{\"notification_config\":{\"pubsub_topic\":\"projects/${PROJECT}/topics/test\"}}" "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/hl7V2Stores/${HL7STORE}?access_token=${TOKEN}&update_mask=notification_config.pubsub_topic" -H "Content-Type: application/json;charset=utf-8"
```

**List**

```shell
curl "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/hl7V2Stores?access_token=${TOKEN}"
```

**Ingest**

```shell
curl -X POST -H "Content-Type: application/json" --data-binary @cloud/healthcare/hl7/message_store/manual_tests/ingest_request.json "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/hl7V2Stores/${HL7STORE}/messages:ingest?access_token=${TOKEN}"
```

**List Messages**

```shell
curl "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/hl7V2Stores/${HL7STORE}/messages?access_token=${TOKEN}&filter="message_type=\"ADT\"""
```

## FHIR

**Create**

```shell
curl -X POST --data "{\"enable_update_create\":\"true\",\"notification_config\":{\"pubsub_topic\":\"projects/${PROJECT}/topics/test\"}}" "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/fhirStores?access_token=${TOKEN}&fhir_store_id=${FHIRSTORE}" -H "Content-Type: application/fhir+json;charset=utf-8"
```

Note that the pubsub topic in the notification config needs to exist for future
calls to succeed. You can check the existence of the topics and create one if
necessary by running:

```shell
gcloud pubsub topics list --project=${PROJECT}
gcloud pubsub topics create <topic-name> --project=${PROJECT}
```

**Update**

```shell
curl -X PATCH --data "{\"enable_update_create\":\"false\"}" "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/fhirStores/${FHIRSTORE}?access_token=${TOKEN}&update_mask=enable_update_create" -H "Content-Type: application/fhir+json;charset=utf-8"
```

**List**

```shell
curl "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/fhirStores?access_token=${TOKEN}"
```

**Create Resource**

Save the following to resource.json, replace `<resource-id>` with the
corresponding resource id.

> { "language": "English", "name": [ { "use": "official", "family": "Armstrong",
> "given": [ "Joe", "William" ] } ], "resourceType": "Patient", "id":
> "`<resource-id`>" }

```shell
curl -X PUT --data @resource.json "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/fhirStores/${FHIRSTORE}/fhir/Patient?access_token=${TOKEN}" -H "Content-Type: application/fhir+json;charset=utf-8"
```

**Get Resource**

```shell
RESOURCE_TYPE="Patient"
RESOURCE_ID=1
curl "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/fhirStores/${FHIRSTORE}/fhir/${RESOURCE_TYPE}/${RESOURCE_ID}?access_token=${TOKEN}"
```

**Update Resource**

```shell
curl -X PUT --data @resource.json "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/fhirStores/${FHIRSTORE}/fhir/${RESOURCE_TYPE}/${RESOURCE_ID}?access_token=${TOKEN}" -H "Content-Type: application/fhir+json;charset=utf-8"
```

**Search Resources**

```shell
curl "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/fhirStores/${FHIRSTORE}/fhir/Patient?access_token=${TOKEN}&given:exact=Joe&given:exact=William"
```

**Delete Resource**

```shell
RESOURCE_TYPE="Encounter"
RESOURCE_ID=1
curl -X DELETE "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/fhirStores/${FHIRSTORE}/fhir/${RESOURCE_TYPE}/${RESOURCE_ID}?access_token=${TOKEN}"
```

**Transaction**

Save the following to bundle.json

> { "resourceType": "Bundle", "type": "transaction", "entry": [ { "resource": {
> "resourceType": "Patient", "name": [ { "family": "MOUSE", "given": [ "MICKEY"
> ] } ], "gender": "male", "address": [ { "line": [ "123 Main St." ], "city":
> "Lake Buena Vista", "state": "FL", "postalCode": "32830" } ] }, "request": {
> "method": "PUT" } } ] }

```shell
curl -X POST --data @bundle.json "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/fhirStores/${FHIRSTORE}/fhir?access_token=${TOKEN}" -H "Content-Type: application/fhir+json;charset=utf-8"
```

**List Resource Versions**

```shell
RESOURCE_TYPE="Encounter"
RESOURCE_ID=1
curl "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/fhirStores/${FHIRSTORE}/fhir/${RESOURCE_TYPE}/${RESOURCE_ID}/_history?access_token=${TOKEN}"
```

**Get Resource Version**

```shell
RESOURCE_TYPE="Encounter"
RESOURCE_ID=1
RESOURCE_VERSION_ID=2
curl "${API_ENDPOINT_PREFIX}/projects/${PROJECT}/locations/${LOCATION}/datasets/${DATASET}/fhirStores/${FHIRSTORE}/fhir/${RESOURCE_TYPE}/${RESOURCE_ID}/_history/${RESOURCE_VERSION_ID}?access_token=${TOKEN}"
```
