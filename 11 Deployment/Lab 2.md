# Exam Questions patterns

## Execute the below commands for lab setup. 

```
kubectl create namespace kdp2001
kubectl create namespace kdpd002021
kubectl create namespace project-tiger
kubectl label nodes workernode1.example.com disktype=ssd
kubectl label nodes workernode2.example.com disktype=ssd
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
- Configure the pod with a container image of ifccnf/nginx:1.13.7-alpine
- Set an environment variable of NGINX_Port=8080 and also expose that port for the container above.



## Question 5

As a kubernetes application developer you will often find yourself needing to update a runing application. Please complete the following:
-  Update the web deployment in the kdpd002021 namespace with a maxSurge of 10% and a maxUnavailable of 5%
-  Perform a rolling update of the web deployment changing the lfccnf/nginx image version to 1.13
-  Rollback the web deployment to the previous version and it should record.


## Question 6

Use Namespace project-tiger for the following. Create a Deployment named deploy-important with label id=very-important (the Pods should also have this label) and 3 replicas. It should contain two containers, the first named container1 with image nginx:1.17.6-alpine and the second one named container2 with image kubernetes/pause.

There should be only ever one Pod of that Deployment running on one worker node. We have two worker nodes: workernode1.example.com and workernode1.example.com. Because the Deployment has three replicas the result should be that on both nodes one Pod is running. The third Pod won't be scheduled, unless a new worker node will be added.

In a way we kind of simulate the behaviour of a DaemonSet here, but using a Deployment and a fixed number of replicas.
Use the Node selector: disktype=ssd

## Question 7

A deployment is failing on the cluster. Locate the deployment, and fix the problem.


## Clean the lab 

```
kubectl delete namespace kdp2001
kubectl delete namespace kdpd002021
kubectl delete namespace project-tiger
kubectl label nodes workernode1.example.com disktype=ssd
kubectl label nodes workernode2.example.com disktype=ssd
kubectl label nodes workernode1.example.com disktype-
kubectl label nodes workernode2.example.com disktype-
```
