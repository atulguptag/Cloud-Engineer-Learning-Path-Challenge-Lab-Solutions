## `Lab Name` - *Perform Foundational Data, ML, and AI Tasks in Google Cloud: Challenge Lab [GSP323]*
## `Lab Link` - [Click Here](https://www.cloudskillsboost.google/focuses/11044?parent=catalog)

## [YouTube Solution Link]()

Run the below commands in the cloud shell terminal

```
REGION=
Dataset=
TABLE=
TASK_3=
TASK_4=
```

```
PROJECT_ID=$(gcloud config get-value project)
target=$Dataset.$TABLE
bucket_name=$PROJECT_ID-marking
```

## Task 1: Run a simple Dataflow job

```
bq mk $Dataset
gsutil mb gs://$bucket_name
```
```
cat > table.py <<EOF
from google.cloud import bigquery

# Construct a BigQuery client object.
client = bigquery.Client()

table_id = "$PROJECT_ID.$Dataset.TABLE"

schema = [
    bigquery.SchemaField("guid", "STRING", mode="NULLABLE"),
    bigquery.SchemaField("isActive", "BOOLEAN", mode="NULLABLE"),
    bigquery.SchemaField("firstname", "STRING", mode="NULLABLE"),
    bigquery.SchemaField("surname", "STRING", mode="NULLABLE"),
    bigquery.SchemaField("company", "STRING", mode="NULLABLE"),
    bigquery.SchemaField("email", "STRING", mode="NULLABLE"),
    bigquery.SchemaField("phone", "STRING", mode="NULLABLE"),
    bigquery.SchemaField("address", "STRING", mode="NULLABLE"),
    bigquery.SchemaField("about", "STRING", mode="NULLABLE"),
    bigquery.SchemaField("registered", "TIMESTAMP", mode="NULLABLE"),
    bigquery.SchemaField("latitude", "FLOAT", mode="NULLABLE"),
    bigquery.SchemaField("longitude", "FLOAT", mode="NULLABLE"),
]

table = bigquery.Table(table_id, schema=schema)
table = client.create_table(table)  # Make an API request.
print(
    "Created table {}.{}.{}".format(table.project, table.dataset_id, table.table_id)
)
EOF
```
## Task 2: Run a simple Dataproc job
```
python3 table.py

gcloud dataflow jobs run lab-transform --gcs-location gs://dataflow-templates-$REGION/latest/GCS_Text_to_BigQuery --worker-machine-type e2-standard-2 --region $REGION --staging-location gs://$PROJECT_ID-marking/temp --parameters javascriptTextTransformGcsPath=gs://cloud-training/gsp323/lab.js,JSONPath=gs://cloud-training/gsp323/lab.schema,javascriptTextTransformFunctionName=transform,outputTable=$PROJECT_ID:$Dataset.$TABLE,inputFilePattern=gs://cloud-training/gsp323/lab.csv,bigQueryLoadingTemporaryDirectory=gs://$PROJECT_ID-marking/bigquery_temp
```

## Task 3: Use the Google Cloud Speech API

```
gcloud dataproc clusters create cluster-f357 --region $REGION --zone $REGION-a --master-machine-type e2-standard-2 --master-boot-disk-size 500 --num-workers 2 --worker-machine-type e2-standard-2 --worker-boot-disk-size 500 --image-version 2.0-debian10 --project $PROJECT_ID
```

--------------------------------------------------------------------------------------------------------------------------
```
gcloud beta compute ssh cluster-f357-w-0 -- -vvv
```
--------------------------------------------------------------------------------------------------------------------------
```
hdfs dfs -cp gs://cloud-training/gsp323/data.txt /data.txt
```
--------------------------------------------------------------------------------------------------------------------------
```
gcloud config set dataproc/region $REGION
gcloud dataproc jobs submit spark --cluster cluster-f357 \
  --class org.apache.spark.examples.SparkPageRank \
  --cluster=cluster-f357 \
  --jars file:///usr/lib/spark/examples/jars/spark-examples.jar -- /data.txt
```
--------------------------------------------------------------------------------------------------------------------------
```
gcloud services enable apikeys.googleapis.com
gcloud alpha services api-keys create --display-name="testname" 
KEY_NAME=$(gcloud alpha services api-keys list --format="value(name)" --filter "displayName=testname")
API_KEY=$(gcloud alpha services api-keys get-key-string $KEY_NAME --format="value(keyString)")
echo $API_KEY
```
--------------------------------------------------------------------------------------------------------------------------
```
gcloud iam service-accounts create techvine \
  --display-name "my natural language service account"
gcloud iam service-accounts keys create ~/key.json \
  --iam-account techvine@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com
export GOOGLE_APPLICATION_CREDENTIALS="/home/$USER/key.json"
gcloud auth activate-service-account techvine@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com --key-file=$GOOGLE_APPLICATION_CREDENTIALS
gcloud ml language analyze-entities --content="Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." > result.json
gcloud auth login --no-launch-browser
```

--------------------------------------------------------------------------------------------------------------------------
```
gsutil cp result.json $TASK_4
cat > request.json <<EOF 
{
  "config": {
      "encoding":"FLAC",
      "languageCode": "en-US"
  },
  "audio": {
      "uri":"gs://cloud-training/gsp323/task3.flac"
  }
}
EOF
curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json \
"https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" > result.json
gsutil cp result.json $TASK_3
gcloud iam service-accounts create quickstart
gcloud iam service-accounts keys create key.json --iam-account quickstart@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com
gcloud auth activate-service-account --key-file key.json
export ACCESS_TOKEN=$(gcloud auth print-access-token)
```
--------------------------------------------------------------------------------------------------------------------------
```
cat > request.json <<EOF 
{
   "inputUri":"gs://spls/gsp154/video/train.mp4",
   "features": [
       "TEXT_DETECTION"
   ]
}
EOF

```
--------------------------------------------------------------------------------------------------------------------------

## Task 4. Use the Cloud Natural Language API
```
curl -s -H 'Content-Type: application/json' \
    -H "Authorization: Bearer $ACCESS_TOKEN" \
    'https://videointelligence.googleapis.com/v1/videos:annotate' \
    -d @request.json
curl -s -H 'Content-Type: application/json' -H "Authorization: Bearer $ACCESS_TOKEN" 'https://videointelligence.googleapis.com/v1/operations/OPERATION_FROM_PREVIOUS_REQUEST' > result1.json
```

# Congratulations🎉! You're all done with this Challenge Lab.