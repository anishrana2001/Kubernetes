## How to insert the configMap from volume?

## Create the namespace
```
kubectl create namespace core
```
### yaml file for configMap and deployment.
```
cat <<EOF>>configmap-volume.yaml
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: example-configmap1
  namespace: core
data:
  NGINX_PORT: "8080"
  language: "english"
  environments: production, development, test
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: core
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        configMap:
         name: example-configmap1
EOF
```

### Create the objects from yaml file.
```
kubectl apply -f configmap-volume.yaml
```

### Explore the configMap through "describe" sub command.

```
kubectl -n core  describe configmaps example-configmap1 
```

### command output for your references.
```
[root@master1 volume]# kubectl -n core  describe configmaps example-configmap1 
Name:         example-configmap1
Namespace:    core
Labels:       <none>
Annotations:  <none>

Data
====
environments:
----
production, development, test
language:
----
english
NGINX_PORT:
----
8080

BinaryData
====
```

### Explore the configMap through "get" sub command.
```
kubectl -n core get configmaps example-configmap1
```

### Verify our pods are created?
```
kubectl get pods
```

### command output for your references.
```
[root@master1 volume]# kubectl -n core get configmaps example-configmap1 
NAME                 DATA   AGE
example-configmap1   3      2m55s
```
### Login into one pod and check the environment variables. 
```
kubectl -n core  exec -it $(kubectl -n core get pods | grep "volume-deploy" | awk '{print $1}' | head -n 1) -- env
```

### You will not observe key-values in the environment.
```
[root@master1 volume]# kubectl -n core  exec -it $(kubectl -n core get pods | grep "volume-deploy" | awk '{print $1}' | head -n 1) -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=volume-deploy-66df575d5b-25qvm
NGINX_VERSION=1.23.4
NJS_VERSION=0.7.11
PKG_RELEASE=1~bullseye
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
TERM=xterm
HOME=/root
```

### You will observe, 3 files created under /etc/config/ direcotry.
```
kubectl -n core  exec -it $(kubectl -n core get pods | grep "volume-deploy" | awk '{print $1}' | head -n 1) -- ls -ltr /etc/config
```
### command output for your references.
```
[root@master1 volume]# kubectl -n core  exec -it $(kubectl -n core get pods | grep "volume-deploy" | awk '{print $1}' | head -n 1) -- ls -ltr /etc/config
total 0
lrwxrwxrwx 1 root root 15 May  5 17:28 language -> ..data/language
lrwxrwxrwx 1 root root 19 May  5 17:28 environments -> ..data/environments
lrwxrwxrwx 1 root root 17 May  5 17:28 NGINX_PORT -> ..data/NGINX_PORT
```

### We can also open this file by using cat command.
```
kubectl -n core  exec -it $(kubectl -n core get pods | grep "volume-deploy" | awk '{print $1}' | head -n 1) -- cat /etc/config/language ; echo
```
### command output for your references.
```
[root@master1 volume]# kubectl -n core  exec -it $(kubectl -n core get pods | grep "volume-deploy" | awk '{print $1}' | head -n 1) -- cat /etc/config/language ; echo
english
```

## Clear the lab
```
kubectl delete -f configmap-volume.yaml --grace-period=0 --force
rm -rf configmap-volume.yaml
```


## Question: 

You are tasked to create a ConfigMap and consume the ConfigMap in a pod using a volume mount. 
Please complete the following.
- Create a ConfigMAP named another-config containing the key/value pare. Key3/value2.
- Start a pod named nginx-configmap containing a single container using the nginx image.
- and mount the key you just created into the pod under directory /some/path


```
kubectl create configmap another-config --from-literal=key3=value2
```
## Create a POD, name=nginx-configmap ; image: nginx ; mountPath: "/some/path" ; configMap: name: another-confi
```
cat <<EOF>>question-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-configmap
spec:
  containers:
  - name: mypod
    image: nginx
    volumeMounts:
    - name: foo
      mountPath: "/some/path"
  volumes:
  - name: foo
    configMap:
      name: another-config
 EOF
 ```
 
 ```
 kubectl create -f question-configmap.yaml
 ```
 
 ## Check if file is correctly mounted on the container?
 
 ```
 kubectl exec -it nginx-configmap -- ls -l /some/path
 ```
 ## Open the file and verify the value.
 ```
 kubectl exec -it nginx-configmap -- cat /some/path/key3 ; echo 
 ```
 
 
 ## How to clear the lab + question?
 
 ```
 kubectl delete pod/nginx-configmap  --grace-period=0 --force
 kubectl delete  configmaps another-config
 rm -f question-configmap.yaml
 ```

