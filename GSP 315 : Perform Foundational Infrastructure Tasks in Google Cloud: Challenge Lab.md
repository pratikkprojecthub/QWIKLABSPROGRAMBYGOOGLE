GSP 315 : Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab

-------------------TASK 1: Remove the overly permissive rules--------------------
IN CLOUD SHELL: 
gcloud auth list

gcloud config list project

gcloud compute firewall-rules delete open-access

----------------------------------------------------------------------------------

-------------------TASK 2: Start the bastion host instance--------------

or use 

gcloud compute instances start bastion

----------------------------------------------------------------------------------------------------------------------------------------------

-------------------TASK 3: Create a firewall rule that allows SSH (tcp/22) from the IAP service--------------------


gcloud compute firewall-rules create ssh-ingress --allow=tcp:22 --source-ranges 35.235.240.0/20 --target-tags ssh-ingress --network acme-vpc


gcloud compute instances add-tags bastion --tags=ssh-ingress --zone=us-central1-b

----------------------------------------------------------------------------------------------------------------------------------------------------

-------------------TASK 4: Create a firewall rule that allows traffic on HTTP (tcp/80) to any address and add network tag on juice-shop-------------


gcloud compute firewall-rules create http-ingress --allow=tcp:80 --source-ranges 0.0.0.0/0 --target-tags http-ingress --network acme-vpc


gcloud compute instances add-tags juice-shop --tags=http-ingress --zone=us-central1-b

------------------------------------------------------------------------------------------------------------------------------------------------------

-------------------TASK 5 : Create a firewall rule that allows traffic on SSH (tcp/22) from acme-mgmt-subnet-----------------

gcloud compute firewall-rules create internal-ssh-ingress --allow=tcp:22 --source-ranges 192.168.10.0/24 --target-tags internal-ssh-ingress --network acme-vpc

gcloud compute instances add-tags juice-shop --tags=internal-ssh-ingress --zone=us-central1-b


--------------------------------------------------------------

-------------------TASK 6: SSH to bastion host via IAP and juice-shop via bastion-------------------------------------------------------------------------


click the SSH button for the bastion host. Once connected, SSH to juice-shop.

ssh [Internal IP address of juice-shop]

------------------------------------------------------------------------------------------------------------------------------------------------------------
