## Create variable file.
```
cat <<EOF>> NGINX_PORT
8080
EOF
```
### Create the configMap by using "--from-file" option.
```
kubectl create configmap env-file --from-file=NGINX_PORT 
```

### Create the deployment, options are same as "envFrom".
```
cat <<EOF>> env-file-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: env-deploy1
spec:
  replicas: 5
  selector:
    matchLabels:
      app: env-deploy1
  template:
    metadata:
      labels:
        app: env-deploy1
    spec:
      containers:
      - name: env-cont
        image: nginx
        envFrom:
        - configMapRef:
            name: env-file
EOF
```

### Checking the environment variables of the container.
```
kubectl   exec -it $(kubectl  get pods | grep "env-deploy1" | awk '{print $1}' | head -n 1) -- env
```


### command output for your references.
```
cat <<EOF>> NGINX_PORT
8080
EOF

[root@master1 volume]# kubectl create configmap env-file --from-file=NGINX_PORT 
configmap/env-file created

[root@master1 volume]# kubectl apply -f env-file-deploy.yaml 
deployment.apps/env-deploy1 created

[root@master1 volume]# kubectl   exec -it $(kubectl  get pods | grep "env-deploy1" | grep Running | awk '{print $1}' | head -n 1) -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=env-deploy1-86f77fc5b-4kncd
NGINX_VERSION=1.23.4
NJS_VERSION=0.7.11
PKG_RELEASE=1~bullseye
NGINX_PORT=8080   ################>>>>>

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

## How to clear the lab?
```
rm -f NGINX_PORT
kubectl delete -f env-file-deploy.yaml --grace-period=0 --force
rm -rf env-file-deploy.yaml
kubectl delete configmaps env-file
```


