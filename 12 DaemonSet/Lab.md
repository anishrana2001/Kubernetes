
# LAB 1
## How to create DaemonSet?
### From Kubernetes.io web site
```
kubectl apply -f https://k8s.io/examples/controllers/daemonset.yaml
```


## How to create Yaml file for DaemonSet from command?
```
kubectl create deployment test1-daemonset --image=nginx:1.14.2 --dry-run=client -oyaml
```

```
cat <<EOF>> test1-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  creationTimestamp: null
  labels:
    app: test1-daemonset
  name: test1-daemonset
spec:
  selector:
    matchLabels:
      app: test1-daemonset
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test1-daemonset
    spec:
      containers:
      - image: nginx:1.14.2
        name: nginx
EOF

```

```
kubectl create -f test1-daemonset.yaml
```

## How to check the DaemonSet?
### Below command should show two DaemonSet, 1 from Kubernetes.io & 2nd from command line.
```
kubectl get ds -n kube-system
```
```
kubectl get ds
```
### If we want to see only "test1-daemonset" DaemonSet then
```
kubectl get daemonsets.apps 
```
```
kubectl get ds 
```
```
kubectl get daemonsets/test1-daemonset 
```



### If we want to create a DaemonSet in the namespace (core)

```
kubectl create namespace core
```
```
kubectl create -f test1-daemonset.yaml -n core
```
```
kubectl get -n core ds/test1-daemonset
```

## DaemonSet do not create a ReplicaSet.

```
kubectl get rs --show-labels
```

```
kubectl get -n core rs
```


### To identify the pods managed by this DaemonSet?

```
kubectl get nodes
```

```
kubectl get pods -o wide 
```

```
kubectl get pods -o wide | egrep "test1-daemonset|NAME"
```


### For more verbose output?
```
kubectl describe daemonsets  test1-daemonset 
```


## How to upgrade the DaemonSet?

### To identify the container name
```
kubectl get daemonset/test1-daemonset -o yaml
```

### Upgrade the container image from 1.14.2 to 1.16.1
### To see the status of pods
```
kubectl get pods
```
```
kubectl set image ds/test1-daemonset  nginx=nginx:1.16.1
```

### To see the status of pods
```
kubectl get pods
```

### How can we check if our upgrade is successful?
```
kubectl rollout status ds/test1-daemonset
```

```
kubectl describe daemonsets  test1-daemonset  | grep 1.16
```


```
 kubectl exec -it test1-daemonset-7wgsn -- env | grep -i nginx
```


### We can also edit the DaemonSet yaml file, just change the nginx:1.16.1 to nginx:1.14.2
```
kubectl edit daemonsets.apps/test1-daemonset
```



```
kubectl exec -it test1-daemonset-45cfc -- env | grep -i nginx
```

## How to rolling  back to a previous revision?

### We can also rolling back to previous revision from command line,too. 


```
kubectl rollout undo DaemonSet/test1-daemonset 
```

```
kubectl get pods --show-labels
```


### How to delete the DaemonSet?
```
kubectl delete DaemonSets.apps test1-daemonset
```

```
kubectl get pods --show-labels | grep test1-daemonset
```

```
kubectl get DaemonSets.apps
```



# Clear the lab 

```
kubectl delete -n core DaemonSets/test1-daemonset
kubectl delete namespaces core --grace-period=0 --force
kubectl delete -n  kube-system ds/fluentd-elasticsearch
```


