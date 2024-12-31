
### How to scale up the StatefulSet ?
```
kubectl scale statefulset nginx-sts --replicas=3
```

```
kubectl get pods
```

### How to scale down the StatefulSet ?
```
kubectl scale statefulset nginx-sts --replicas=2
```

```
kubectl get pods
```
