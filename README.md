# BigQuery Question and Answer Bot

### What Problem does the app solve



# Prerequisites
The assumption is made that you have set up a google cloud project and activated billing for this project.

#### The following Google Cloud Project APIs need to be enabled
* IAM
* BigQuery
* Vertex AI
* Cloud Run


#### gcloud Setup
To initialize the gcloud CLI:
```shell
gcloud init
```

## Deploy Container to GCP Container Register

### Set an environment variables
```shell
export GOOGLE_CLOUD_PROJECT="<YOUR PROJECT ID>"
export CONTAINER_NAME='cnt-boilerplate-streamlit'
export APP_NAME='app-boilerplate-streamlit'
export REGION="us-central1"
export AR_NAME="ar-boilerplate-streamlit" 
export AR_URI=${REGION}-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/${AR_NAME}
#export SERVICE_ACCOUNT="<enter service account for cloud run"
```

```shell
gcloud auth application-default login
gcloud auth login
gcloud config set project ${GOOGLE_CLOUD_PROJECT}
```

### Create a repository in Artifact Registry
```shell
gcloud artifacts repositories create ${AR_NAME} --location=${REGION} --repository-format=docker
```
**Note**: If this command fails, run the below and try again: 
```shell
gcloud components update
```

### Create Docker file
Copy the following into a Dockerfile (if not exists)
```shell
FROM python:3.9-slim
WORKDIR /app
COPY . .
RUN pip3 install -r requirements.txt
EXPOSE 8501
ENTRYPOINT ["streamlit", "run", "app.py","--server.port=8501"]
```

### Setup Docker auth
```shell
gcloud auth configure-docker us-central1-docker.pkg.dev
```

### Create and tag docker image
```shell
docker build -t ${AR_URI}/${CONTAINER_NAME} .
```

### Push to Artifact Registry
```shell
docker push ${AR_URI}/${CONTAINER_NAME}
```

### Cloud from Service Identity
By default, Cloud Run revisions and jobs execute as the Compute Engine default service account. The Compute Engine default service account has the Project Editor IAM role which grants read and write permissions on all resources in your Google Cloud project.
While this may be convenient, rather than use the default service account, Google recommends creating your own user-managed service account with the most granular permissions and assigning that service account as your Cloud Run service or job identity. 

See the doce here for details [here](https://cloud.google.com/run/docs/securing/service-identity#gcloud)

### Deploy to Cloud Run
```shell
gcloud run deploy ${APP_NAME} \
--image=${AR_URI}/${CONTAINER_NAME} \
--platform=managed \
--port 8501 \
--service-account=${SERVICE_ACCOUNT} \
--region=${REGION} \
--cpu=2 --memory=1Gi \
--max-instances=2 \
--allow-unauthenticated
```

Note, remove this flag if you don't want to allow unauthenticated access:
```shell
--allow-unauthenticated
```

Access the Service URL when CLoud Run service has been deployed
```
Service URL: https://app-boilerplate-streamlit-random-suffix.a.run.app
```

See the details [here](https://cloud.google.com/run/docs/authenticating/public#gcloud)



