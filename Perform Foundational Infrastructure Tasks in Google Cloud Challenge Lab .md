## `Lab Name` - *Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab [GSP315]*
## `Lab Link` - [Click Here](https://www.cloudskillsboost.google/focuses/10379?parent=catalog)


## Task 1: Create a bucket

* *Navigation menu* > `Cloud Storage` > Browser > `Create Bucket`

* *Name your bucket* > Enter the Bucket name provided in your lab > Continue

* Use default for the remaining

* Click on `Create`

## OR

* You can use the below command to create a `Cloud Storage Bucket`.

```
export BUCKET_NAME=
```

```
gsutil mb gs://$BUCKET_NAME
```

## Task 2: Create a Pub/Sub topic


* *Navigation menu* > `Pub/Sub` > `Topics`

* *Create Topic* > Name: the name provided in your lab > `Create Topic`

## OR

* You can use the below command to create a `Pub/Sub - Topic`.

```
export YOUR_TOPIC_NAME=
```

```
gcloud pubsub topics create $YOUR_TOPIC_NAME
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

* You will see an another file just created with the name - `Image_Name-64x64-thumbnail` (Like this).

## OR

* You can use the following commands to create and deploy the Cloud Function.

```
export REGION=

export FUNCTION_NAME=
```

```
gcloud config set compute/region $REGION

mkdir gcf_hello_world

cd gcf_hello_world

nano index.js
```
* Put the below code into your `index.js` file.

```
/* globals exports, require */
//jshint strict: false
//jshint esversion: 6
"use strict";
const crc32 = require("fast-crc32c");
const { Storage } = require('@google-cloud/storage');
const gcs = new Storage();
const { PubSub } = require('@google-cloud/pubsub');
const imagemagick = require("imagemagick-stream");
exports.thumbnail = (event, context) => {
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64"
  const bucket = gcs.bucket(bucketName);
  const topicName = "$YOUR_TOPIC_NAME";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      let newFilename = filename_without_ext + size + '_thumbnail.' + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
              // set the content-type
              gcsNewObject.setMetadata(
              {
                contentType: 'image/'+ filename_ext.toLowerCase()
              }, function(err, apiResponse) {});
              pubsub
                .topic(topicName)
                .publisher()
                .publish(Buffer.from(newFilename))
                .then(messageId => {
                  console.log(`Message ${messageId} published.`);
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });
          });
      });
    }
    else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  }
  else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
};
```

* This will create and open your `package.json` file.

```
nano package.json
```

* Put the below code in your `package.json` file.

```
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/pubsub": "^2.0.0",
    "@google-cloud/storage": "^5.0.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}
```

* It's time to deploy your Cloud Function.

```
gcloud functions deploy $FUNCTION_NAME \
  --region $REGION \
  --stage-bucket $BUCKET_NAME \
  --trigger-topic $YOUR_TOPIC_NAME \
  --runtime nodejs14 \
  --entry-point thumbnail
```

## Task 4: Remove the previous cloud engineer

* *Navigation menu* > `IAM & Admin` > `IAM`

* Search for the "Username 2" > *Edit* > `Delete Role`
 

# Congratulations🎉! You're all done with this Challenge Lab.