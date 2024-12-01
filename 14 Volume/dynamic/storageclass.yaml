## StorageClass yaml file.
```
cat <<EOF>> storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: k8s-sigs.io/nfs   # This name must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"
EOF
```
