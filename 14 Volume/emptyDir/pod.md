# emptyDir POD Yaml file

```
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod2
  namespace: core
spec:
  volumes:
  - name: vol1-emptydir
    emptyDir:
      sizeLimit: 500Mi
  containers:
  - image: nginx
    name: emptydir-container2
    volumeMounts:
    - mountPath: /cache
      name: vol1-emptydir
```
