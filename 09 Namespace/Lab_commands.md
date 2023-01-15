# LAB Qestions

## How to check the Namespace 
```
kubectl get namespaces
kubectl  get ns
```

## How to create Namespace with Yaml file?

```
cat > development.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    name: production
```
