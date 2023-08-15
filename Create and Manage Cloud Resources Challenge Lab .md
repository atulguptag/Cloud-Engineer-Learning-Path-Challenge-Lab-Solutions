## `Lab Name` - *Create and Manage Cloud Resources: Challenge Lab Solution [GSP313]*
## `Lab Link` - [Click Here](https://www.cloudskillsboost.google/focuses/10258?parent=catalog)


### Simply copy the following code and paste it into your `cloud shell terminal`.

```cmd
export INSTANCE_NAME=
```
```cmd
export ZONAL=
```
```cmd
export PORT_NUMBER=
```
```cmd
export FIREWALL_RULE=
```

## Task 1. Create a project jumphost instance

```cmd
gcloud compute instances create $INSTANCE_NAME \
--zone=$ZONAL \
--machine-type=e2-micro \
--image=projects/debian-cloud/global/images/debian-10-buster-v20220519
```

## Task 2. Create a Kubernetes service cluster

```cmd
gcloud container clusters create nucleus-backend \
          --num-nodes 1 \
          --network nucleus-vpc \
          --region $ZONAL

gcloud container clusters get-credentials nucleus-backend \
          --region $ZONAL

kubectl create deployment hello-server \
          --image=gcr.io/google-samples/hello-app:2.0

kubectl expose deployment hello-server \
          --type=LoadBalancer \
          --port $PORT_NUMBER
```

## Task 3. Set up an HTTP load balancer

```cmd
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```

```cmd
gcloud compute instance-templates create web-server-template \
          --metadata-from-file startup-script=startup.sh \
          --network nucleus-vpc \
          --machine-type g1-small \
          --region us-east1

gcloud compute instance-groups managed create web-server-group \
          --base-instance-name web-server \
          --size 2 \
          --template web-server-template \
          --region us-east1

gcloud compute firewall-rules create $FIREWALL_RULE \
          --allow tcp:80 \
          --network nucleus-vpc

gcloud compute http-health-checks create http-basic-check

gcloud compute instance-groups managed \
          set-named-ports web-server-group \
          --named-ports http:80 \
          --region us-east1

gcloud compute backend-services create web-server-backend \
          --protocol HTTP \
          --http-health-checks http-basic-check \
          --global

gcloud compute backend-services add-backend web-server-backend \
          --instance-group web-server-group \
          --instance-group-region us-east1 \
          --global

gcloud compute url-maps create web-server-map \
          --default-service web-server-backend

gcloud compute target-http-proxies create http-lb-proxy \
          --url-map web-server-map

gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80
        
gcloud compute forwarding-rules list
```

# Congratulations🎉! You are done with the Challenge Lab.
