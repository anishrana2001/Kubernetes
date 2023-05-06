# LAB for configMap

## How to insert the configMap as an environment variables?

### yaml file for configMap and deployment.
```
cat <<EOF>> configmap-env.yaml
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: example-configmap 
data:
  NGINX_PORT: "8080"
  language: "english"
  environments: production, development, test
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: env-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: env-deploy
  template:
    metadata:
      labels:
        app: env-deploy
    spec:
      containers:
      - name: env-cont
        image: nginx
        envFrom:
        - configMapRef:
            name: example-configmap
EOF
```
### Create the objects from yaml file.
```
kubectl apply -f configmap-env.yaml
```

### Explore the configMap through "describe" sub command.

```
kubectl describe configmaps example-configmap
```

### command output for your references.
```
[root@master1 volume]# kubectl describe configmaps example-configmap 
Name:         example-configmap
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
NGINX_PORT:
----
8080
environments:
----
production, development, test
language:
----
english

BinaryData
====

Events:  <none>
```

### Explore the configMap through "get" sub command.
```
kubectl get configmaps example-configmap
```

### command output for your references.
```
[root@master1 volume]# kubectl get configmaps example-configmap 
NAME                DATA   AGE
example-configmap   3      2m49s
```

### Login into one pod and check the environment variables. 
```
kubectl exec -it $(kubectl get pods | grep "env-deploy" | awk '{print $1}' | head -n 1) -- env
```

### command output for your references.
```
[root@master1 volume]# kubectl exec -it $(kubectl get pods | grep "env-deploy" | awk '{print $1}' | head -n 1) -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=env-deploy-698f6d8d98-5w87z
NGINX_VERSION=1.23.4
NJS_VERSION=0.7.11
PKG_RELEASE=1~bullseye
NGINX_PORT=8080
environments=production, development, test
language=english
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
TERM=xterm
HOME=/root
```
  
## clear the lab
```
kubectl delete -f configmap-env.yaml --grace-period=0 --force
rm -rf configmap-env.yaml
```
  
