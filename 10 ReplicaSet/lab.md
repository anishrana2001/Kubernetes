# LAB
## How to create replicaset from yaml file?
```
kubectl apply -f https://kubernetes.io/examples/controllers/frontend.yaml
```
## How to check the replicaset?
```
kubectl get replicasets
kubectl get rs
```
## How to check the pods?
```
kubectl get pods
```
## How to delete the replicaset's pod?
```
kubectl delete pod/
kubectl get pods
```
## How to check the information of replicaset?
```
kubectl describe rs/frontend
```

## How to delete the replicaset ?
```
kubectl delete replicasets  frontend 
```

### how to verify if replicaset is deleted?
```
kubectl get rs
kubectl get pods
```
