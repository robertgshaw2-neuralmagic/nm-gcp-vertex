# nm-gcp-vertex
Example using DeepSparse in a GCP Vertex model serving endpoint

## Setup Example Project in GCP

Create Project
```
gcloud projects create deepsparse-vertex --name="DeepSparse Vertex Example"
```

Switch to this Project in the gcloud SDK.
```
xxx
```

## Push Docker Container with DeepSparse to Artifact Repository

Authenticate to gcloud
```
gcloud auth configure-docker us-east1-docker.pkg.dev
```

Enable Artifact Repository API
```
gcloud services enable artifactregistry.googleapis.com
```
Wait a few minutes for this to propogate through GCP's systems.

Create Artifact Repository
```
gcloud artifacts repositories create deepsparse-server-images --repository-format=docker --location=us-east1
```

Build Docker Image
```
docker build -t deepsparse-gcp-vertex:latest us-east1-docker.pkg.dev/deepsparse-vertex/deepsparse-server-images/sentiment-analysis .
```

Push Docker Image to Artifact Repository
```
docker push us-east1-docker.pkg.dev/deepsparse-vertex/deepsparse-server-images/sentiment-analysis
```

## Setup Model and Endpoint

Enable Vertex AI

Navigate to the [Vertex AI Dashboard](https://console.cloud.google.com/vertex-ai?project=deepsparse-vertex) and click "Enable Recommended APIs."


Create a Model
```bash
gcloud ai models upload \
  --region=us-east1 \
  --display-name=sparse-model \ 
  --container-image-uri=us-east1-docker.pkg.dev/deepsparse-vertex/deepsparse-server-images/sentiment-analysis \
  --container-ports=5543 \
  --container-health-route=/health \ 
  --container-predict-route=/predict
```

Get Model ID
```bash
gcloud ai models list \
    --region=us-east1 \
    --display-name=sparse-model
```
Note the number that appears in the `MODEL_ID` column. This will be used below when we deploy the Model to the Endpoint.

Create an Endpoint
```bash
gcloud ai endpoints create \
    --region=us-east1 \ 
    --display-name=example-endpoint
```

Get Endpoint ID
```bash
gcloud ai endpoints list \
    --region=us-east1 \
    --filter=display_name=example-endpoint
```
Note the number that appears in the `ENDPOINT_ID` column. This will be used below when we deploy the Model to the Endpoint.

Deploy model to the Endpoint.

```bash
gcloud ai endpoints deploy-model ENDPOINT_ID \
    --region=us-east1 \
    --model=MODEL_ID \ 
    --display-name=sparse-model \
    --machine-type=n1-highcpu-8 \
    --min-replica-count=1
```
Note that our endpoint does not use any accelerators.

## Send Requests To The Server

Create a JSON file to send to the server. The Sentiment Analysis Pipeline expects an array of sequences.

```json
{"sequences": ["The man dislikes going to the store", "The man loves going to the store"]}
```

Send A Request to the Endpoint:

```
gcloud ai endpoints raw-predict ENDPOINT_ID \
    --region=us-east1 \
    --request=@request.json
```
