****************************************************GSP343: Optimize Costs for Google Kubernetes Engine************************************************************************

------------------------------------------------------Task 1: Create cluster and deploy an app ------------------------------------------

ZONE=us-central1-a

gcloud container clusters create onlineboutique-cluster \
   --project=${DEVSHELL_PROJECT_ID} --zone=${ZONE} \
   --machine-type=n1-standard-2 --num-nodes=2

kubectl create namespace dev

kubectl create namespace prod

kubectl config set-context --current --namespace dev

git clone https://github.com/GoogleCloudPlatform/microservices-demo.git

cd microservices-demo

kubectl apply -f ./release/kubernetes-manifests.yaml --namespace dev

kubectl get svc -w --namespace dev

--------->--------->Open http://<EXTERNAL_IP> in a new tab.  homepage of the Online Boutique application will appear.----<----------<---------

---------------------------------------------------------------------------------------------------------------------------------------------------------------

------------------------------------------------------Task 2 : Migrate to an Optimized Nodepool ------------------------------------------

gcloud container node-pools create optimized-pool \
   --cluster=onlineboutique-cluster \
   --machine-type=custom-2-3584 \
   --num-nodes=2 \
   --zone=$ZONE

for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do
   kubectl cordon "$node";
done

for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do
   kubectl drain --force --ignore-daemonsets --delete-local-data --grace-period=10 "$node";
done

kubectl get pods -o=wide --namespace dev

gcloud container node-pools delete default-pool \
   --cluster onlineboutique-cluster --zone $ZONE



---------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------Task 3 : Apply a Frontend Update------------------------------------------


kubectl create poddisruptionbudget onlineboutique-frontend-pdb \
--selector app=frontend --min-available 1  --namespace dev

KUBE_EDITOR="nano" kubectl edit deployment/frontend --namespace dev

---------------->--Change the following--<---------------
   image to gcr.io/qwiklabs-resources/onlineboutique-frontend:v2.1

   imagePullPolicy to Always

**************save it using ctrl+o ---> Enter ---> ctrl + X************

---------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------Task 4 : Autoscale from Estimated Traffic------------------------------------------

kubectl autoscale deployment frontend --cpu-percent=50 \
   --min=1 --max=13 --namespace dev

kubectl get hpa --namespace dev

gcloud beta container clusters update onlineboutique-cluster \
   --enable-autoscaling --min-nodes 1 --max-nodes 6 --zone $ZONE

kubectl exec $(kubectl get pod --namespace=dev | grep 'loadgenerator' | cut -f1 -d ' ') \
   -it --namespace=dev -- bash -c "export USERS=8000; sh ./loadgen.sh"
   
---------------------------------------------------------------------------------------------------------------------------------------------------------------



-----------------------------------------------------------------------CONGRATULATIONS--------------------------------------------------------------------------
