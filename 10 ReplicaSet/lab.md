# LAB
## How to create replicaset from yaml file?
```
kubectl apply -f https://kubernetes.io/examples/controllers/frontend.yaml
```
## How to check the replicaset?
```
kubectl get replicasets
```
```
kubectl get rs
```
## How to check the pods?
```
kubectl get pods
```
## How to delete the replicaset's pod?
```
kubectl delete pod/pod_name
```
```
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

### How to verify if replicaset is deleted?
```
kubectl get rs
```
```
kubectl get pods
```
## If one pod is already runing with same labels?
```
kubectl run rcpod  --image=gcr.io/google_samples/gb-frontend:v3 --labels=tier=frontend
```
```
kubectl get pods --show-labels
```
```
kubectl apply -f https://kubernetes.io/examples/controllers/frontend.yaml
```
```
kubectl describe replicasets.apps/frontend | grep -i replicas:
```
```
kubectl get pods --show-labels
```
```
