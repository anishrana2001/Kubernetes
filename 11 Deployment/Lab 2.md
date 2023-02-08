# Exam Questions patterns

## Question 1

Create a new deployment for running nginx with the following parameters. 
- Run the deploymnet in the kdp2001 namespace. The namespace has alrady been created. 
- Name the deployment nginx and configure with 5 replicas
- Configure the pod with a container image of ifccnf/nginx:1.13.7-alpine
- Set an environment variable of NGINX_Port=8080 and also expose that port for the container above.



## Question 2

As a kubernetes application developer you will often find yourself needing to update a runing application. Please complete the following:
-  Update the web deployment in the kdpd002021 namespace with a maxSurge of 10% and a maxUnavailable of 5%
-  Perform a rolling update of the web deployment changing the lfccnf/nginx image version to 1.13
-  Rollback the web deployment to the previous version 


## Question 3

A deployment is failing on the cluster. Locate the deployment, and fix the problem.
