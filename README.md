# Create-and-Manage-Cloud-Resources-Challenge-Lab

# Task 1: Create a project jumphost instance
gcloud compute instances create nucleus-jumphost --machine-type f1-micro --zone us-east1-b


# Task 2: Create a Kubernetes service cluster

gcloud config set compute/zone us-east1-b

gcloud container clusters create nucleus-webserver1

gcloud container clusters get-credentials nucleus-webserver1

kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0

kubectl expose deployment hello-app --type=LoadBalancer --port 8080

kubectl get service 


# Task 3: Setup an HTTP load balancer

cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF

<h4>1 .Create an instance template :</h4>

gcloud compute instance-templates create nginx-template \
--metadata-from-file startup-script=startup.sh

<h4>2 .Create a target pool :</h4>

gcloud compute target-pools create nginx-pool

<h4>3 .Create a managed instance group :</h4>

gcloud compute instance-groups managed create nginx-group \
--base-instance-name nginx \
--size 2 \
--template nginx-template \
--target-pool nginx-pool

gcloud compute instances list

<h4>4 .Create a firewall rule to allow traffic (80/tcp) :</h4>

gcloud compute firewall-rules create www-firewall --allow tcp:80

gcloud compute forwarding-rules create nginx-lb \
--region us-east1 \
--ports=80 \
--target-pool nginx-pool

gcloud compute forwarding-rules list

<h4>5 .Create a health check :</h4>

gcloud compute http-health-checks create http-basic-check

gcloud compute instance-groups managed \
set-named-ports nginx-group \
--named-ports http:80

<h4>6 .Create a backend service and attach the manged instance group :</h4>

gcloud compute backend-services create nginx-backend \
--protocol HTTP --http-health-checks http-basic-check --global

gcloud compute backend-services add-backend nginx-backend \
--instance-group nginx-group \
--instance-group-zone us-east1-b \
--global

<h4>7 .Create a URL map and target HTTP proxy to route requests to your URL map :</h4>

gcloud compute url-maps create web-map \
--default-service nginx-backend

gcloud compute target-http-proxies create http-lb-proxy \
--url-map web-map

<h4>8 .Create a forwarding rule :</h4>

gcloud compute forwarding-rules create http-content-rule \
--global \
--target-http-proxy http-lb-proxy \
--ports 80

gcloud compute forwarding-rules list
