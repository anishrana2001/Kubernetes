## Question : 

## You have asked to create a role $\color[rgb]{1,0,1}config-role$, which allow only user $\color[rgb]{1,0,1}suraj$ to create only ConfigMap on $\color[rgb]{1,0,1}app-config$ namespace. Consume this ConfigMap in a pod using a volume mount. Please complete the following.
### - Create a ConfigMAP named $\color[rgb]{1,0,1}another-config$ containing the key/value pair, $\color[rgb]{1,0,1}key30/redcolour$ .
### - Start a pod named $\color[rgb]{1,0,1}nginx-configmap$ containing a single container using the $\color[rgb]{1,0,1}nginx$ image and mount the key you just created into the pod under directory $\color[rgb]{1,0,1}/some/path$
### - This pod must be created only those nodes which has label $\color[rgb]{1,0,1}disktype=ssd$

### Solution: 
#### Namespace = app-config
#### Role    =  config-role
#### verb    = create
#### resource = ConfigMap
#### user   = suraj

#### Pod name  = nginx-configmap
#### configMap = another-config
#### Key/value = key30/redcolour
#### image     = nginx
#### mountpath = /some/path
#### nodeselector = disktype=ssd



```
kubectl get nodes --show-labels
```
```
kubectl label nodes workernode1.example.com disktype=ssd
```
```
kubectl label nodes workernode2.example.com disktype=ssd
```
```
kubectl get nodes --show-labels
```
```
kubectl create namespace app-config
```
```
kubectl create configmap another-config --from-literal=key30=redcolour -n app-config
```

#### In order to create Role, we can open the Kubernetes web page.

#### https://kubernetes.io

#### Documentation --> Search -> Role
```
cat <<EOF>> question1.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: app-config
  name: config-role
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deploy-b
  namespace: app-config
subjects:
- kind: User
  name: suraj
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: config-role
  apiGroup: rbac.authorization.k8s.io
EOF
```
#### For creating configmap , we can copy the yaml file from Kubernetes web page.
#### https://kubernetes.io

#### Documentation --> Search -> volumes -> configmap

```
cat <<EOF>> pods.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-configmap
  namespace: app-config
spec:
  nodeSelector:        ## NodeSelector 
    disktype: ssd       ## NodeSelector disktype: ssd
  containers:
    - name: test
      image: nginx
      volumeMounts:
        - name: config-vol
          mountPath: /some/path
  volumes:
    - name: config-vol
      configMap:
        name: another-config
        items:
          - key: key30
            path: key30
EOF
```

```
kubectl create -f question1.yaml 
```
```
kubectl create -f pods.yaml 
```
```
kubectl get pods -n app-config 
```
```
kubectl auth can-i create configmaps --as suraj --namespace app-config
```
```
kubectl auth can-i create configmaps --as suraj 
```

## Now, create a system user suraj and also create a certificate for him. 

```
openssl genrsa -out suraj.key 2048
```
```
openssl req -new -key suraj.key -out suraj.csr -subj "/CN=suraj/O=dev/O=example.org"
```
```
openssl x509 -req -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -days 730 -in suraj.csr -out suraj.crt
```
```
kubectl config set-credentials suraj --client-certificate=/root/rbac/suraj.crt --client-key=/root/rbac/suraj.key --embed-certs=true
```
```
kubectl config get-contexts
```
```
kubectl config set-context suraj-context --cluster=kubernetes  --namespace=app-config --user=suraj 
```
```
kubectl config get-contexts
```
```
kubectl config use-context suraj-context
```
```
kubectl auth can-i create configmaps
```
## How to switch back to Admin user?
```
kubectl config use-context kubernetes-admin@kubernetes
```
## How to delete Role & RoleBinding?

### First check the role and rolebinding.
```
kubectl -n app-config get role/config-role rolebindings.rbac.authorization.k8s.io/deploy-b
```
### After examine the names, we can delete it by executing below command.
```
kubectl delete -n app-config role/config-role rolebindings.rbac.authorization.k8s.io/deploy-b
```

## How to delete context?

### Identify if you are on correct context.

kubectl config current-context 

### List all the contexts.
```
kubectl config get-contexts
```
### Check the current config.
```
kubectl config view
```

### Now, delete the context.
```
kubectl config delete-context suraj-context 
```
```
kubectl config delete-user suraj
```

### Post Check the current config.
```
kubectl config view
```

### Delete all the certificates.
```
cd /root/rbac
rm -f suraj.*
```

