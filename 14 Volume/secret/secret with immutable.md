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

```yaml
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
```yaml
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
### Let's check the current encrypted value of password.

```
kubectl get -n tiger secrets prod-db-secret3 -o yaml
```
```
echo -n Mysql123 | base64
```


## Now, edit the secret. 
```
echo -n Mysql789 | base64 
```
```
kubectl -n tiger  edit secrets prod-db-secret3
```
### For your references.

```
[root@master1 volume]# kubectl -n tiger  edit secrets prod-db-secret3
error: secrets "prod-db-secret3" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-1752694574.yaml"
error: Edit cancelled, no valid changes were saved.
[root@master1 volume]# 
```

## Clear the Lab
```
kubectl delete -n tiger pods/secret-volume-pod3 secrets/prod-db-secret3
kubectl delete namespaces tiger
rm -f prod-db-secret3.yaml secret-volume-pod3.yaml  tiger.yaml
ls -ltr
```

   
