# Docker #
First part of the task dockerizng the rails project
Genrate Ruby on rails project
Create Dockerfile (Docker-Files/Dockerfile)
Create docker-compose.yml (Docker-Files/docker-compose.yml)

# AWS#
All this part complete on AWS EC2 and you can access this container from this link https://www.hostersstack.com/ that secured with SSL certificate that implement on ELB we can add auto-scale if havy traffic happened scale new container (but this just a demo :D )

# Countuse Integration #
# Jenkins #
All of this environment deployed from Jenkins CI that also trigger any repo updates then building this project to ensure that the stg srv run the latest version of the app
You can find the simple  Jenkins-Pipeline file that responsible for deploy new code on the stg srv (Jenkins/Pipeline)

# kubernetes #
We have a kubernetes cluster contains master machine and one node machine

# Docker Images#
First, I create the image of drkiq and sidekiq from Dockerfile given in article throw docker-compose.yml file
then push them to public repo ithabetz/img:drkiq and ithabetz/img:sidekiq

### configMap:
I used this service to pass the env	variables to the container that contains the DB credentials and listener port and other variables

### PostgreSQL:
create PV volume from path /drkiq-Postgres then create claim volume from PV volume this volume will store the data of PostgreSQL database
PostgreSQL service that communicates with the pod that generate container itself that will act as DB server

### Redis:
create PV volume from path /drkiq-redis then create claim volume from PV volume this volume will store the data of redis
Redis service that communicates with the pod that generate container itself that will serve the drkiq/sidekiq to store data on it

### Drkiq:
Create PV volume from path /drkiq/ then create claim volume from PV volume this volume will store the source code of the rails project to be able to edit in the code without needing to rebuild the image 
Drkiq service that communicates with the POD that generate container itself that will be the server of rails we can communicate with throw port 8000 the container will serve data from PV volume
Also will communicate with cache memory on redis container throw configMap to get the variables 
Also will communicate with DB on postgresSQL container throw configMap to get the variables 

### Sidekiq:
Sidekiq will use the same volume of drkiq because this container will use the same repo (source code) that drkiq will use
Also, sidekiq will can communicate with redis and DB throw the same configMap

## Rails Repo:
We need to exec some command to init our rails project we can attach handlers to Container lifecycle events. to exec some command after POD start
We need to reset and migrate DB 
We need to generate a Pages controller with a home and add a new job

## NOTES:
we need to isolate the source code of rails project outside the image itself (because if we made any change on the project we need to rebuild the image and update the kubernetes deployment)
so we can make the source  code in a separated repo that will be pulled in the directory that will be shared with drkiq and sidekiq deployment (so you must pull rails project in /drkiq/ on the node before start any deployment (you can change /drkiq/ on node but you need to update the drkiq PV yml file also))
we will skip this step because we made a demo only so we edit drkiq-deployment.yaml and sidekiq-deployment.yaml and change mountPath to - mountPath: /drkiq-pro/ or any other name that not document root

#To create all Services, Deployment, PV, PVCs and configmaps:
kubectl create -f configMap.yaml,drkiq-pv.yaml,postgres-pvc.yaml,redis-deployment.yaml,redis-service.yaml,drkiq-deployment.yaml,drkiq-service.yaml,postgres-pv.yaml,redis-pvc.yaml,sidekiq-deployment.yaml,drkiq-pvc.yaml,postgres-deployment.yaml,postgres-service.yaml,redis-pv.yaml

#To get all Services, Deployment, PV, PVCs and configmaps:
kubectl get service ; kubectl get deployment ;kubectl get pv ; kubectl get pvc ; kubectl get configmap

#To delete all Services, Deployment, PV, PVCs and configmaps:
kubectl delete deployment drkiq postgres redis sidekiq ; kubectl delete service drkiq postgres redis ;kubectl delete pvc drkiq-postgres drkiq-redis drkiq sidekiq ; kubectl delete pv drkiq-postgres drkiq-redis drkiq sidekiq ;kubectl delete configmap vars
