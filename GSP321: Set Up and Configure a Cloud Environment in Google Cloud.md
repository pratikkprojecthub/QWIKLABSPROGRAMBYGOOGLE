GSP321: Set Up and Configure a Cloud Environment in Google Cloud

----------------------------------------------------------------Task 1:Create development VPC manually ------------------------------------------

gcloud auth list

gcloud compute networks create griffin-dev-vpc --subnet-mode custom

gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region us-east1 --range=192.168.16.0/20

gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region us-east1 --range=192.168.32.0/20


---------------------------------------------------------------------------------------------------------------------------------------------------------



----------------------------------------------------------------Task 2:Create production VPC manually --------------------------------------------------

gsutil cp -r gs://cloud-training/gsp321/dm .

cd dm

sed -i s/SET_REGION/us-east1/g prod-network.yaml

gcloud deployment-manager deployments create prod-network \
    --config=prod-network.yaml

cd ..

---------------------------------------------------------------------------------------------------------------------------------------------------------



-----------------------------------------------------------Task 3:Create bastion host ---------------------------------------------------------------------


gcloud compute instances create bastion --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt  --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt --tags=ssh --zone=us-east1-b


gcloud compute firewall-rules create fw-ssh-dev --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-dev-vpc


gcloud compute firewall-rules create fw-ssh-prod --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-prod-vpc


-----------------------------------------------------------------------------------------------------------------------------------------------------



----------------------------------------------------Task 4:Create and configure Cloud SQL Instance--------------------------------------------------

gcloud sql instances create griffin-dev-db --root-password password --region=us-east1

gcloud sql connect griffin-dev-db

CREATE DATABASE wordpress;

GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";

FLUSH PRIVILEGES;

exit


------------------------------------------------------------------------------------------------------------------------------------------------------



---------------------------------------------------------------Task 5:Create Kubernetes cluster-------------------------------------------------------


gcloud container clusters create griffin-dev \
  --network griffin-dev-vpc \
  --subnetwork griffin-dev-wp \
  --machine-type n1-standard-4 \
  --num-nodes 2  \
  --zone us-east1-b


gcloud container clusters get-credentials griffin-dev --zone us-east1-b

cd ~/

gsutil cp -r gs://cloud-training/gsp321/wp-k8s .


------------------------------------------------------------------------------------------------------------------------------------------------------



-----------------------------------------------------------Task 6:Prepare the Kubernetes cluster -----------------------------------------------------

Open Editor -> wp-k8s -> wp-env.yaml Change username and password to:


  name: database
  
type: Opaque

stringData:

  username: wp_user
  
  password: stormwind_rules

--------> -------> save the changes made<--------<--------

---------------------------------------------------------------------------------


cd wp-k8s


kubectl create -f wp-env.yaml


gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
    
    
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json


---------------------------------------------------------------------------------

-------------------------------------------------Task 7:Prepare the Kubernetes cluster -------------------------------------------------------


Open the editor: Go to wp-deployment.yaml 

---> replace YOUR_SQL_INSTANCE with griffin-dev-db.

the code will be present between 40 to 44 line
command: ["/cloud_sql_proxy",
                    "-instances=griffin-dev-db=tcp:3306",
                    "-credential_file=/secrets/cloudsql/key.json"]


--------> -------> save the changes made<--------<--------


kubectl create -f wp-deployment.yaml


kubectl create -f wp-service.yaml

---------------------------------------------------------------------------------

------------------------------------------------ Task 8 : Enable monitoring -------------------------------------------------------------------

	
Navigation Menu -> Kubernetes Engine -> Services and Ingress -> Copy Endpoint's address. 

for ex : 35.227.5.109

Navigation Menu 

---> Monitoring 

---> Uptime Checks 

---> +create uptime checks

Title : Wordpress Uptime

Next

Hostname : {Endpoint's address} 35.227.5.109

Path : /

Next

Test

Then Create

-------------------------------------------------------------------------------------------------------------------------------------------------------

--------------------------------------------------Task 9: Provide access for an additional engineer ---------------------------------------------------

Navigation Menu 

---> IAM & Admin 

---> IAM -> Add

New Member : {Put Username 2 from lab instruction page on lab left corner}

Role : Project ---> Basic ---> Editor

-------------------------------------------------------------------------------------------------------------------------------------------------------

-------------------------------------------------------------------Congratulations---------------------------------------------------------------------
