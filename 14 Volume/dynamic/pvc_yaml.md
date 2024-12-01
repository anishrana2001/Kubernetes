## How to create PVC 

```
cat <<EOF>>  dynamic-PersistentVolumeClaim.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
spec:
  storageClassName: nfs-client  ## This name must be same what we configured in StorageClass name
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
EOF
```

```
kubectl apply -f dynamic-PersistentVolumeClaim.yaml
```
