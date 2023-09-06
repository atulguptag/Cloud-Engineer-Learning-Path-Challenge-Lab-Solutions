## `Lab Name` - *Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab [GSP315]*
## `Lab Link` - [Click Here](https://www.cloudskillsboost.google/focuses/10379?parent=catalog)


Run the below commands in the cloud shell terminal


## Task 1: Create a bucket

* *Navigation menu* > `Cloud Storage` > Browser > `Create Bucket`

* *Name your bucket* > Enter the Bucket name provided in your lab > Continue

* Use default for the remaining

* Click on `Create`

## OR

* You can use the below command to create a `Cloud Storage Bucket`.

```
gsutil mb gs://YOUR_BUCKET_NAME
```

## Task 2: Create a Pub/Sub topic


* *Navigation menu* > `Pub/Sub` > `Topics`

* *Create Topic* > Name: the name provided in your lab > `Create Topic`

## OR

* You can use the below command to create a `Pub/Sub - Topic`.

```
gcloud pubsub topics create YOUR_TOPIC_NAME
```

## Task 3: Create the thumbnail Cloud Function

* *Navigation menu* > `Cloud Functions` > `Create Function`


### Use the following configuration:

* *Name*: The Name provided in your lab 

* *Region*: `us-east1`

* *Trigger*: `Cloud Storage`

* *Event type*: `Finalize/Create`

* *Bucket*: BROWSE > `Select the qwiklabs bucket`

* *Remaining default* > `Next`

* *Runtime*: `Node.js 14`

* *Entry point*: `thumbnail`

* Add the `index.js` and `package.json` code by yourself.

* In the `index.js` code replace the text `REPLACE_WITH_YOUR_TOPIC` ID with the Topic Name you created in task 2.

* Download the image from the URL provided in your lab or [Click Here](https://storage.googleapis.com/cloud-training/gsp315/map.jpg) to Download directly.

* *Navigation menu* > `Cloud Storage` > `Browser` > `Select your bucket` > `Upload files` > Upload the recent downloaded file. 

* Refresh bucket

* You will see an another file just created with the name - `Image_Name 64x64 thumbnail` (Like this).

## Task 4: Remove the previous cloud engineer

* *Navigation menu* > `IAM & Admin` > `IAM`

* Search for the "Username 2" > *Edit* > `Delete Role`
 

# Congratulations🎉! You're all done with this Challenge Lab.