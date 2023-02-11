# Exam Questions patterns

## Execute the below commands for lab setup. 

```
kubectl create namespace kdp2001
kubectl create namespace kdpd002021
kubectl create namespace project-tiger
kubectl -n kdpd002021 create deployment web --image=nginx:1.13
kubectl create deployment testing-deploy --image=nginx:1.14.7 --replicas=2
```

## Question 1

Create a deployment loadbalancer with image nginx:1.14.2 and it should have 2 replicas. 

## Question 2

Scale the deployment loadbalancer to 6 pods.


## Question 3

Create a Deployment named deploy-important with image nginx:1.17.6-alpine. It should contain 2 containers, the first named nginx with image nginx:1.17.6-alpine and the second one named redis with image redis. 


## Question 4

Create a new deployment for running nginx with the following parameters. 
- Run the deploymnet in the kdp2001 namespace. The namespace has alrady been created. 
- Name the deployment nginx and configure with 5 replicas
- Configure the pod with a container image of nginx:1.13.7-alpine
- Set an environment variable of NGINX_Port=8080 and also expose that port for the container above.



## Question 5

As a kubernetes application developer you will often find yourself needing to update a runing application. Please complete the following:
-  Update the web deployment in the kdpd002021 namespace with a maxSurge of 10% and a maxUnavailable of 5%
-  Perform a rolling update of the web deployment changing the nginx image version to 1.14.2
-  Rollback the web deployment to the previous version and it should record.


## Question 6

Use Namespace project-tiger for the following. Create a Deployment named deploy-important with label id=very-important (the Pods should also have this label) and 3 replicas. It should contain two containers, the first named container1 with image nginx:1.17.6-alpine and the second one named container2 with image kubernetes/pause.

Use the Node selector: disktype=ssd

## Question 7

A deployment is failing on the cluster. Locate the deployment, and fix the problem.


## Clean the lab 

```
kubectl delete namespace kdp2001   --grace-period=0 --force
kubectl delete namespace kdpd002021  --grace-period=0 --force
kubectl delete namespace project-tiger  --grace-period=0 --force
kubectl label nodes workernode1.example.com disktype-
kubectl label nodes workernode2.example.com disktype-
kubectl delete deployments.apps deploy-important
kubectl delete deployments.apps testing-deploy
kubectl delete deployment.apps/loadbalancer
```
