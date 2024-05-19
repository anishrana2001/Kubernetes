# Configure Memory and CPU Quotas for a Namespace


#### Requests and limits

####  Requests: It means minimum amount of resource it can consume (request). Kubernetes Schedular will check the node if it can create a pod with this minimum resource?
#### .
####  Limits: It means, maximum amount of request container can demand. The kubelet (and container runtime) enforce the limit. 
#### If application inside the container demands more resources (example allowed amount of memory), then the system kernel terminates the process that attempted the allocation, with an out of memory (OOM) error.
#### 

## In order to perform this, first we will going to create a Namespace "ns-quota1"
```
kubectl create namespace ns-quota1
```
## After that, we neend to create a ResourceQuota inside the newly created Namespace "ns-quota1". In this manifest file, we have mentioned the requests and limit values under the spec.hard section.
```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
  namespace: ns-quota1
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
EOF
```


## We can also check this ResourceQuota by executing below commands.
```
kubectl -n ns-quota1 get resourcequota mem-cpu-demo -o yaml
```

```
kubectl -n ns-quota1 describe resourcequota mem-cpu-demo
```

## Now, its time to create a pod ,but not inside this namespace. Thus, this resourcequota values will be not changed.
```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "800m"
      requests:
        memory: "600Mi"
        cpu: "400m"
EOF
```

## Check the resourcequota information
```
kubectl -n ns-quota1 get resourcequota mem-cpu-demo -o yaml
```
## Now, delete this pod. There is no use of it.
```
kubectl delete pod/quota-mem-cpu-demo
```
## Create a pod under the namespace "ns-quota1". This time, our resourcequota will work.
```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo
  namespace: ns-quota1
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "800m"
      requests:
        memory: "600Mi"
        cpu: "400m"
EOF
```

##  Check the resourcequota information
```
kubectl -n ns-quota1 get resourcequota mem-cpu-demo  -o yaml
```
## A need and clean way to check.
```
kubectl -n ns-quota1 get resourcequota mem-cpu-demo -o jsonpath='{ .status.used }' | jq .
```

## If we again creae a POD with excede value, it should give the error message. Let's try.
```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo-2
  namespace: ns-quota1
spec:
  containers:
  - name: quota-mem-cpu-demo-2-ctr
    image: redis
    resources:
      limits:
        memory: "1Gi"
        cpu: "800m"
      requests:
        memory: "700Mi"
        cpu: "400m"
EOF
```

Error from server (Forbidden): error when creating "STDIN": pods "quota-mem-cpu-demo-2" is forbidden: exceeded quota: mem-cpu-demo, requested: requests.memory=700Mi, used: requests.memory=600Mi, limited: requests.memory=1Gi

# Clear the lab.
```
kubectl -n ns-quota1 delete pod/quota-mem-cpu-demo  resourcequota/mem-cpu-demo
kubectl delete  namespace ns-quota1
```

