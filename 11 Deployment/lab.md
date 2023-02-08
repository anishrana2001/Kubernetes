- ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) `#f03c15`
- ![#c5f015](https://placehold.co/15x15/c5f015/c5f015.png) `#c5f015`
- ![#1589F0](https://placehold.co/15x15/1589F0/1589F0.png) `Anish`

# LAB
## How to create Deployment?
### From Kubernetes.io web site
```
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
```
### From command line
```
Kubectl  create deployment  nginx  --image=nginx:1.14.2
```

## How to create Yaml file for Deployment from command?
```
Kubectl create deployment nginx --image=nginx:1.14.2 --dry-run=client -o yaml > deployment1.yaml
```
```
kubectl create -f deployment1.yaml
```

## How to check the Deployment?
### Below command should show two deployment, 1 from Kubernetes.io & 2nd from command line.
```
kubectl get deploy
```

### If we want to see only "nginx-deployment" deployment then
```
kubectl get deployment nginx-deployment
```

### If we want to create a deployment in the namespace (core)

```
kubectl create namespace core
kubectl create deployment nginx --image=nginx:1.14.2 -n core
kubectl get -n core deployments/nginx
```

## If we want to list the ReplicaSets created by deployment?

```
kubectl get rs --show-labels
```

```
kubectl get rs/nginx-deployment-
```
### You can also list all the ReplicaSets
```
kubectl get rs 
```

### To identify the pods accquired by this deployment?

```
kubectl get pods --show-labels
```

```
kubectl get pods --show-labels | grep nginx-deployment
```


### For more verbose output?
```
kubectl describe deployment nginx-deployment
```


## How to upgrade the deployment?

### To identify the container name
```
kubectl get deployments.apps nginx-deployment -o yaml
```

### Upgrade the container from 1.14.2 to 1.16.1
```
kubectl `#1589F0`set image deployment nginx-deployment nginx=nginx:1.16.1
```

### To see the status of pods
```
kubectl get pods -w
```

### Now, it should be 2 ReplicaSets for this deployment
```
kubectl  get rs --show-labels
```

### How can we check if our upgrade is successful?
```
kubectl rollout status deployment/nginx-deployment
```

```
kubectl describe deployments.apps nginx-deployment | grep 1.16
```

```
kubectl exec -it nginx-deployment-68fc675d59-7sdv4 -- env | grep -i nginx
```


### We can also edit the deployment yaml file also.
```
kubectl edit deployments.apps nginx-deployment 
```

### Old ReplicaSet must be use now.
```
kubectl get rs
```

### How to check if this edit feature is working?
```
kubectl exec -it nginx-deployment-7fb96c846b-2b6rp -- env | grep -i nginx
```

## How to rolling  back to a previous revision?

### We can also rolling back to previous revision from command line,too. 
```
kubectl get rs
```

```
kubectl rollout undo deployment/nginx-deployment 
```

```
kubectl get pods --show-labels
```

```
kubectl get pods --show-labels
```

```
kubectl get rs --show-labels
```



## How to scale UP/Down the deployment?

```
kubectl get deployments.apps 
```
```
kubectl scale deployment/nginx-deployment --replicas=5
```
```
kubectl get deployments.apps 
```

```
kubectl autoscale deployment/nginx-deployment --min=5 --max=10 --cpu-percent=70
```

### We can also scale down the number of pods.
```
kubectl scale deployment/nginx-deployment --replicas=2

```

```
kubectl  get deployments/nginx-deployment 
```

### We can also scale up/down the pods by directly editing the yaml file.
```
kubectl edit deployments.apps nginx-deployment 
```
```
kubectl  get deployments/nginx-deployment 
```

### How to delete the Deployment?
```
kubectl delete deployments.apps nginx-deployment
```

```
kubectl get pods --show-labels | grep nginx
```

```
kubectl get deployments.apps 
```



# Clear the lab 

```
kubectl delete -n core deployments/nginx
kubectl delete namespaces core --grace-period=0 --force
kubectl delete deployment/nginx-deployment --grace-period=0 --force
kubectl delete deployments/nginx --grace-period=0 --force
```

