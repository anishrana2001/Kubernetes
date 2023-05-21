# LAB for secret with **immutable**   
## Encrypt the variables
```
echo -n database | base64
```
``` 
echo -n Mysql123 | base64
```

### For your references. 
[root@master1 volume]# echo -n database | base64 
ZGF0YWJhc2U=
[root@master1 volume]# echo -n Mysql123 | base64 
TXlzcWwxMjM=
[root@master1 volume]# 


## How to create Namespace from Yaml file?
Step 1: 
```yaml 
cat <<EOF>>tiger.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: tiger
EOF
```
Step 2:
```
kubectl create -f tiger.yaml
```
## Create a secret
### Step 1: Create a secret under tiger namespace with encrypted data (base64).

```
cat <<EOF>>prod-db-secret3.yaml
apiVersion: v1 
kind: Secret 
metadata: 
    name: prod-db-secret3
    namespace: tiger
type: Opaque 
data: 
   username: ZGF0YWJhc2U=
   password: TXlzcWwxMjM=
immutable: true
EOF
```
### Step 2: Create a secret by using kubectl apply option.
```
kubectl apply -f prod-db-secret3.yaml
```
## How to get more details of this secret?
### Step 1: With the help of Discribe coommand.
```
kubectl describe secrets -n tiger prod-db-secret3
```
### Step 2: With the help of "get" sub command.
```
kubectl -n tiger get secrets prod-db-secret3
```
## 1. POD consuming secret by volume.

### Step 1: Creating a POD under namespace **tiger** and insert the variables from **volume**.
```
cat <<EOF>>secret-volume-pod3.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod3
  namespace: tiger
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: prod-db-secret3
      optional: true
EOF
```
### Step 2: Create the POD
```
kubectl apply -f secret-volume-pod3.yaml
```
```
kubectl get pods -n tiger
```
### Chekc the POD

#### Login into the POD and check the files are created under "/etc/foo" directory.
```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-volume-pod3) -- ls -ltr /etc/foo/ ; echo
```
#### use the command "cat" to list the content of username file.
```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-volume-pod3) -- cat /etc/foo/username ; echo
```
#### use the command "cat" to list the content of password file.
```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-volume-pod3) -- cat /etc/foo/password ; echo
```
## 2. POD consuming secret by env option.
### Creating a POD under "tiger" namespace by using **ENV** option for using secret.
```yaml
cat <<EOF>>secret-env-pod3.yaml	  
apiVersion: v1 
kind: Pod 
metadata: 
  name: secret-env-pod3
  namespace: tiger
spec: 
  containers: 
  - name: secret-env-pod3
    image: redis 
    env: 
      - name: SECRET_USERNAME 
        valueFrom: 
          secretKeyRef: 
            name: prod-db-secret3
            key: username 
      - name: SECRET_PASSWORD 
        valueFrom: 
          secretKeyRef: 
            name: prod-db-secret3
            key: password
EOF
```
```
kubectl create -f secret-env-pod3.yaml
```
### List the POD under tiger namespace.
```
kubectl get pods -n tiger
```

### Chekc the POD

#### Login into the POD and check if the environment varibales are available at container level?.
```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-env-pod3) -- env ; echo
```

### For your references.
```
[root@master1 volume]# kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-env-pod3) -- env ; echo
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=secret-env-pod3
GOSU_VERSION=1.16
REDIS_VERSION=7.0.11
REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-7.0.11.tar.gz
REDIS_DOWNLOAD_SHA=ce250d1fba042c613de38a15d40889b78f7cb6d5461a27e35017ba39b07221e3
############################
SECRET_USERNAME=database
SECRET_PASSWORD=Mysql123
#############################
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
TERM=xterm
HOME=/root
```

## Now, edit the secret. 
```
echo -n Mysql789 | base64 
```
```
kubectl -n tiger  edit secrets prod-db-secret3
```


## Clear the Lab
```
kubectl delete -n tiger pods/secret-env-pod3 pods/secret-volume-pod3
kubectl delete secrets -n tiger prod-db-secret3 
rm -f prod-db-secret3.yaml secret-volume-pod3.yaml secret-env-pod3.yaml
kubectl get all -n tiger
```

   
