

# 1. How to add / validate / delete the taints?
## Create a directory and go inside it.
```
mkdir  taint
cd taint
```
## How to check the taints on node?

### First, check the node name.
```
kubectl get nodes
```
```
[root@master1 ~]# kubectl get nodes
NAME                      STATUS   ROLES           AGE    VERSION
master1.example.com       Ready    control-plane   283d   v1.25.4
workernode1.example.com   Ready    <none>          283d   v1.25.4
workernode2.example.com   Ready    <none>          283d   v1.25.4
```

### We want to add taints on "workernode1" node. key=prod, value=green, effect=NoSchedule
```
kubectl taint node workernode1.example.com prod=green:NoSchedule
```

## How to check the taints is applied?
```
kubectl describe nodes workernode1.example.com | grep -i taints
```

## How to remove the taints from node?
```
kubectl taint node workernode1.example.com prod=green:NoSchedule-
```
## Validate the status of nodes.
```
kubectl describe nodes workernode1.example.com | grep -i taints
```


# 2. Explore the effect option.

## Effect = NoSchedule
### Let's create 2 deployments so that we can validate the effect "NoSchedule"
```
kubectl create deploy no-schedule-effect-deploy1 --image=nginx --replicas=3
kubectl create deploy no-schedule-effect-deploy2 --image=nginx --replicas=3
```

```
kubectl get deployments.apps
```
### Validate if pods are running on both workernodes.
```
kubectl get pods -owide
```

### Add the taints
```
kubectl taint node workernode1.example.com prod=green:NoSchedule
```
### Check the taints on workernode1.
```
kubectl describe nodes workernode1.example.com | grep -i taints
```

### Now, again validate if it impact the running pods?
#### It should not change any pods, becuase we use effect=NoSchedule
```
kubectl get pods -owide
```

### Add one more effect i.e. NoExecute on workernode2

```
kubectl taint node workernode2.example.com prod=green:NoExecute
```
### Check the taints on workernode2.
```
kubectl describe nodes workernode2.example.com | grep -i taints
```
### Let's, check the pods status.
```
kubectl get pods -owide
```

### Remove the taint from workernode2.
```
kubectl taint node workernode2.example.com prod=green:NoExecute-
```
### You will observe the pending pods, get the nodes.
```
kubectl get pods -owide
```

### Add one more effect i.e. PreferNoSchedule on workernode2

```
kubectl taint node workernode2.example.com prod=green:PreferNoSchedule
```

### Will it impact any thing on running pods?

```
kubectl get pods -owide
```
#### 1. What we learnt from this lab is, NoSchedule and PreferNoSchedule effects do not impact on existing pods. However, NoExecute effect will move the existing pods. 

#### 2. PreferNoSchedule effect is preference (1. preference, 2 preference like this) or "soft" version of NoSchedule.

### How to clear the lab?
```
kubectl delete deploy no-schedule-effect-deploy1
kubectl delete deploy no-schedule-effect-deploy2
kubectl taint node workernode2.example.com prod=green:PreferNoSchedule-
kubectl taint node workernode1.example.com prod=green:NoSchedule-
```




# 3. How to add and validate toleration?

## Taint match & use operator "Equal"

### Add the taint on nodes.
```
kubectl taint node workernode1.example.com prod=green:NoSchedule
```
### Pod yaml file for "Taint match & operator 'Equal' "
```
cat <<EOF>> taint-match-and-operator-equal.yaml
apiVersion: v1
kind: Pod
metadata:
  name: taint-match-op-equal
  labels:
    env: test
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: "prod"
      operator: "Equal"
      effect: "NoSchedule"
      value: "green"
EOF
```

```
kubectl apply -f taint-match-and-operator-equal.yaml 
```
#### Note: This pod can be schedule on workernode2 because, on workernode2 there is no restriction, i.e taint.

```
kubectl get pods -o wide
```

### Let's add the taints on workernode2. But this time, we will add the taint provided by Kubernetes.
### If we want to disable the scheduling on workernode2 then we can use "cordon"
```
kubectl get nodes
```
```
kubectl cordon workernode2.example.com
```
```
kubectl get nodes
```

### Delete the pods and recreate it, so that we can varify it.

```
kubectl delete pods taint-match-op-equal 
kubectl apply -f taint-match-and-operator-equal.yaml
kubectl get pods -owide
```

##  Taint match & operator "Exists"

```
cat <<EOF>> taint-match-and-operator-exists.yaml
apiVersion: v1
kind: Pod
metadata:
  name: taint-match-op-exists
  labels:
    env: test
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: prod
      operator: Exists
      effect: NoSchedule
EOF
```
```
kubectl apply -f taint-match-and-operator-exists.yaml
```
### You will observe pod is scheduled on workernode1. It means that we configured well. 

```
kubectl get pods -owide
```

##  Taint not match but effect: "NoSchedule" matches for operator "Equal"
### Node workernode1.example.com  "prod=green:NoSchedule"
```
cat <<EOF>> taint-not-match-but-operator-equal-match.yaml
apiVersion: v1
kind: Pod
metadata:
  name: taint-not-match-but-operator-equal-match
  labels:
    env: test
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: "prod"
      operator: "Equal"
      effect: "NoSchedule"
      value: "red"
EOF
```
```
kubectl apply -f taint-not-match-but-operator-equal-match.yaml
```

### This pod must be in pending state.
```
kubectl get pods -owide
```

##  Taint "key" not match but effect NoSchedule matches with operator "Exists", this time with "Exists" operator.
### Node workernode1.example.com  "prod=green:NoSchedule"
```
cat <<EOF>> taint-not-match-but-operator-exists-match.yaml
apiVersion: v1
kind: Pod
metadata:
  name: tain-match-not-effect-op-equal
  labels:
    env: test
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: "prod1"
      operator: "Exists"
      effect: "NoSchedule"
EOF
```
```
kubectl apply -f taint-not-match-but-operator-exists-match.yaml
```
### This pod must be in pending state too.
```
kubectl get pods -owide
```


### What we learnt that if taints values not match in pod configuraiton file then pod will not schedule on tainted node. But these pods can be schedule on non-tainted nodes.

##  Taint match but this time, but effect: NoExecute not match, with operator "Equal".
### Tainted Node workernode1.example.com ; "prod=green:NoSchedule"
```
cat <<EOF>> taint-match-but-operator-equal-not-match.yaml
apiVersion: v1
kind: Pod
metadata:
  name: taint-match-but-operator-equal-not-match
  labels:
    env: test
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: prod
      operator: Equal
      effect: NoExecute
EOF
```
```
kubectl apply -f taint-match-but-operator-equal-not-match.yaml
```
### This pod also not schedule on workernode1 node. 

```
kubectl get pods -owide --sort-by=.metadata.creationTimestamp
```

##  Taint match but effect: NoExecute not match, with operator "Exists"
### Node workernode1.example.com  "prod=green:NoSchedule"

```
cat <<EOF>> taint-match-but-operator-exists-not-match.yaml
apiVersion: v1
kind: Pod
metadata:
  name: taint-match-but-operator-exists-not-match
  labels:
    env: test
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: prod
      operator: Exists
      effect: NoExecute
EOF
```
```
kubectl apply -f taint-match-but-operator-exists-not-match.yaml
```
### This pod also not schedule on workernode1 node. 
```
kubectl get pods -owide --sort-by=.metadata.creationTimestamp
```

## How to disable scheduling on nodes?
```
kubectl uncordon workernode2.example.com
```
```
kubectl get nodes
```
### Now, you will observe that all pods created on workernode2. Because, there is no taints applied on this node.
```
kubectl get pods -owide --sort-by=.metadata.creationTimestamp
```

## How to clear the lab ?
```
kubectl delete -f taint-match-and-operator-equal.yaml -f taint-match-and-operator-exists.yaml -f taint-not-match-but-operator-equal-match.yaml -f taint-not-match-but-operator-exists-match.yaml -f taint-match-but-operator-equal-not-match.yaml -f taint-match-but-operator-exists-not-match.yaml
kubectl taint node workernode1.example.com prod=green:NoSchedule-
rm -f taint-*
cd ..
rm -rf taint/
```
