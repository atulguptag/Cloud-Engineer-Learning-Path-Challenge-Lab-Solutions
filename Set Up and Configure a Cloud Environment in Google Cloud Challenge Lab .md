## `Lab Name` - *Set Up and Configure a Cloud Environment in Google Cloud: Challenge Lab [GSP321]*
## `Lab Link` - [Click Here](https://www.cloudskillsboost.google/focuses/10603?parent=catalog)


* Run the below commands in the cloud shell terminal

```cmd
export USER_NAME2=
```


## Task 1. Create development VPC manually

```cmd
gcloud compute networks create griffin-dev-vpc --subnet-mode custom

gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region us-east1 --range=192.168.16.0/20

gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region us-east1 --range=192.168.32.0/20
```


## Task 2. Create production VPC manually

```cmd
gsutil cp -r gs://cloud-training/gsp321/dm .

cd dm

sed -i s/SET_REGION/us-east1/g prod-network.yaml

gcloud deployment-manager deployments create prod-network \
    --config=prod-network.yaml

cd ..
```

## Task 3. Create bastion host

```cmd
gcloud compute instances create bastion --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt  --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt --tags=ssh --zone=us-east1-b

gcloud compute firewall-rules create fw-ssh-dev --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-dev-vpc

gcloud compute firewall-rules create fw-ssh-prod --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-prod-vpc
```

## Task 4. Create and configure Cloud SQL Instance

```cmd
gcloud sql instances create griffin-dev-db --root-password password --region=us-east1 --database-version=MYSQL_5_7
```

```
gcloud sql connect griffin-dev-db
```
* For SQL PASSWORD - use `password`.
```
CREATE DATABASE wordpress;
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
FLUSH PRIVILEGES;
```

```
exit
```

## Task 5. Create Kubernetes cluster

* This will take approx 5-10 mins to create a Cluster, so be patience here.

```cmd
gcloud container clusters create griffin-dev \
  --network griffin-dev-vpc \
  --subnetwork griffin-dev-wp \
  --machine-type n1-standard-4 \
  --num-nodes 2  \
  --zone us-east1-b

gcloud container clusters get-credentials griffin-dev --zone us-east1-b

cd ~/

gsutil cp -r gs://cloud-training/gsp321/wp-k8s .
```

## Task 6. Prepare the Kubernetes cluster

```cmd
sed -i "s/username_goes_here/wp_user/g" wp-k8s/wp-env.yaml

sed -i "s/username_goes_here/stormwind_rules/g" wp-k8s/wp-env.yaml

cd wp-k8s

kubectl create -f wp-env.yaml

gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

## Task 7. Create a WordPress deployment

```cmd
sed -i "s/YOUR_SQL_INSTANCE_CONNECTION_NAME/griffin-dev-db/g" wp-deployment.yaml
```

```
kubectl create -f wp-deployment.yaml

kubectl create -f wp-service.yaml

kubectl get svc -w
```

## Task 8. Enable monitoring

* `FOLLOW THE VIDEO INSTRUCTION`

### OR

* Go to Monitoring > from the left panel click on the `Uptime Check` and then Click on *`+Create Uptime Check`* 

In the *Name Field* - `<Any Name>`

Click Next, 

Host name - `<External IP of your wordpress db>`

Path - `/`

Click Next, Next and then click on *`Create`*


## Task 9. Provide access for an additional engineer

```cmd
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member=user:$USER_NAME2 --role=roles/editor
```

# Congratulations🎉! You are Done with the Challenge Lab.  