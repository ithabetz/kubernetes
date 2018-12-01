Docker:
first, I create the image of drkiq and sidekiq from Dockerfile given in article throw docker-compose.yml file
then push them to public repo ithabetz/img:drkiq and ithabetz/img:sidekiq


kubernetes:
We have a kubernetes cluster contains master machine and one node machine

configMap:
I used this service to pass the env	variables to the container that contains the DB credentials and listener port and other variables

PostgreSQL:
create PV volume from path /drkiq-Postgres then create claim volume from PV volume this volume will store the data of PostgreSQL database
PostgreSQL service that communicates with the pod that generate container itself that will act as DB server

Redis:
create PV volume from path /drkiq-redis then create claim volume from PV volume this volume will store the data of redis
Redis service that communicates with the pod that generate container itself that will serve the drkiq/sidekiq to store data on it

Drkiq:
Create PV volume from path /drkiq/ then create claim volume from PV volume this volume will store the source code of the rails project to be able to edit in the code without needing to rebuild the image 
Drkiq service that communicates with the POD that generate container itself that will be the server of rails we can communicate with throw port 8000 the container will serve data from PV volume
Also will communicate with cache memory on redis container throw configMap to get the variables 
Also will communicate with DB on postgresSQL container throw configMap to get the variables 

Sidekiq:
Sidekiq will use the same volume of drkiq because this container will use the same repo (source code) that drkiq will use
Also, sidekiq will can communicate with redis and DB throw the same configMap


NOTES:
we need to isolate the source code of rails project outside the image itself (because if we made any change on the project we need to rebuild the image and update the kubernetes deployment)
so we can make the source  code in a separated repo that will be pulled in the directory that will be shared with drkiq and sidekiq deployment (so you must pull rails project in /drkiq/ on the node before start any deployment (you can change /drkiq/ on node but you need to update the drkiq PV yml file also))
we will skip this step because we made a demo only so we edit drkiq-deployment.yaml and sidekiq-deployment.yaml and change mountPath to - mountPath: /drkiq-pro/ or any other name that not document root

#To create all Services, Deployment, PV, PVCs and configmaps
kubectl create -f configMap.yaml,drkiq-pv.yaml,postgres-pvc.yaml,redis-deployment.yaml,redis-service.yaml,drkiq-deployment.yaml,drkiq-service.yaml,postgres-pv.yaml,redis-pvc.yaml,sidekiq-deployment.yaml,drkiq-pvc.yaml,postgres-deployment.yaml,postgres-service.yaml,redis-pv.yaml

#To get all Services, Deployment, PV, PVCs and configmaps
kubectl get service ; kubectl get deployment ;kubectl get pv ; kubectl get pvc ; kubectl get configmap

#To delete all Services, Deployment, PV, PVCs and configmaps
kubectl delete deployment drkiq postgres redis sidekiq ; kubectl delete service drkiq postgres redis ;kubectl delete pvc drkiq-postgres drkiq-redis drkiq sidekiq ; kubectl delete pv drkiq-postgres drkiq-redis drkiq sidekiq ;kubectl delete configmap vars  

