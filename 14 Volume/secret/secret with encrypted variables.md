# LAB for secret with encrypted variables.   

### For your references. 
[root@master1 volume]# echo -n  database | base64 
ZGF0YWJhc2U=
[root@master1 volume]# echo -n Mysql123 | base64 
TXlzcWwxMjM=
[root@master1 volume]# 


## How to create Namespace from Yaml file?
```yaml 
cat <<EOF>>tiger.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: tiger
EOF
```
```
kubectl create -f tiger.yaml
```
### Create a secret under tiger namespace with encrypted data (base64).

```
cat <<EOF>>prod-db-secret2.yaml
apiVersion: v1 
kind: Secret 
metadata: 
    name: prod-db-secret2
    namespace: tiger
type: Opaque 
data: 
   username: ZGF0YWJhc2U=
   password: TXlzcWwxMjM=
EOF
```
### Create a secret by using kubectl apply option.
```
kubectl apply -f prod-db-secret2.yaml
```

### Discribe the secret.
```
kubectl describe secrets -n tiger prod-db-secret2
```

```
kubectl -n tiger get secrets prod-db-secret2
```




## Creating a POD under namespace tiger and insert the variables from volume.
```
cat <<EOF>>secret-volume-pod2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod2
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
      secretName: prod-db-secret2
      optional: true
EOF
```
### Create the POD
```
kubectl apply -f secret-volume-pod2.yaml
```
```
kubectl get pods -n tiger
```

## Login into the POD and check the files are created under "/etc/foo"
```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-volume-pod2) -- ls -ltr /etc/foo/ ; echo
```

```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-volume-pod2) -- cat /etc/foo/username ; echo
```
```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-volume-pod2) -- cat /etc/foo/password ; echo
```

## Creating a POD under "tiger" namespace by using ENV option for using secret.
```
cat <<EOF>>secret-env-pod2.yaml	  
apiVersion: v1 
kind: Pod 
metadata: 
  name: secret-env-pod2
  namespace: tiger
spec: 
  containers: 
  - name: secret-env-pod2
    image: redis 
    env: 
      - name: SECRET_USERNAME 
        valueFrom: 
          secretKeyRef: 
            name: prod-db-secret2
            key: username 
      - name: SECRET_PASSWORD 
        valueFrom: 
          secretKeyRef: 
            name: prod-db-secret2
            key: password
EOF
```
```
kubectl create -f secret-env-pod2.yaml
```

```
kubectl get pods -n tiger
```

```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-env-pod2) -- env ; echo
```

### For your references.
```
[root@master1 volume]# kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-env-pod2) -- env ; echo
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=secret-env-pod2
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

## Now, edit the secret and check if our container is updated ?
```
echo -n Mysql789 | base64 
```
```
kubectl -n tiger  edit secrets prod-db-secret2
```
```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-env-pod2) -- env | grep SECRET
```

```
[root@master1 volume]# echo -n Mysql789 | base64 
TXlzcWw3ODk=
[root@master1 volume]# kubectl -n tiger  edit secrets prod-db-secret2 
secret/prod-db-secret2 edited
[root@master1 volume]# kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-env-pod2) -- env | grep SECRET
SECRET_USERNAME=database
SECRET_PASSWORD=Mysql123
[root@master1 volume]# 
```
## We have to delete the exsisting POD then again create the pods to get the modified values.

```
kubectl -n tiger edit secrets prod-db-secret2
```
## Create the pod again.
```
kubectl create -f secret-env-pod2.yaml
```
## Now, check the keys of secret.
```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-env-pod2) -- env | grep SECRET
```
## For your references.
[root@master1 volume]# kubectl delete -n tiger pods secret-env-pod2
pod "secret-env-pod2" deleted
[root@master1 volume]# kubectl create -f secret-env-pod2.yaml
pod/secret-env-pod2 created
[root@master1 volume]# kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-env-pod2) -- env | grep SECRET
SECRET_USERNAME=database
SECRET_PASSWORD=Mysql789

## Clear the Lab
```
kubectl delete -n tiger pods/secret-env-pod2 pods/secret-volume-pod2
kubectl delete secrets -n tiger prod-db-secret2
kubectl delete namespaces tiger
rm -f prod-db-secret2.yaml secret-volume-pod2.yaml secret-env-pod2.yaml tiger.yaml
kubectl get all -n tiger
```

