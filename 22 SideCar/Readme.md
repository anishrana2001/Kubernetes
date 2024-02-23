# SideCar

## Given a container that writes a log file in format A and a container that converts logs files form fromatA to foramtB, create a deployment that runs both containers such that the log files from the first container are converted by the second container, emitting logs in formatB.
## - Create a deployment named deployment-007 in default namespace, that- includes image busybox:1.28  container, named web-one
## - Includes a sidecar busybox:1.28 container, named sidecar-two
## - Mounts a share volume /tmp/log on both containers, which does not persist when the pod is deleted.
## - Instruct the web-one container to run the command
## ---
## while true; do 
## echo "i luv cncf" >> /tmp/log/input.log; 
## sleep 10;
## done

## which should output logs to /tmp/log/input.log in plain text format with example values.

## Solution :

### Google--> Kubernetes.io --> Search --> Logging Architecture

```yaml
[root@master1 ~]# kubectl create deployment deployment-007 --image=busybox:1.28 -o yaml --dry-run=client
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: deployment-007
  name: deployment-007
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deployment-007
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deployment-007
    spec:
      containers:
      - image: busybox:1.28
        name: busybox
        resources: {}
status: {}
[root@master1 ~]# 
```

![image](https://github.com/anishrana2001/Kubernetes/assets/93471182/4565623f-0dd6-4a7a-a104-1204dd6d92c5)

