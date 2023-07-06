## Question : 

## You have asked to create a role $\color[rgb]{1,0,1}config-role$, which allow only user $\color[rgb]{1,0,1}raja$ to create only ConfigMap on $\color[rgb]{1,0,1}app-config$ namespace. Consume this ConfigMap in a pod using a volume mount. Please complete the following.
### - Create a ConfigMAP named $\color[rgb]{1,0,1}another-config$ containing the key/value pair, $\color[rgb]{1,0,1}key30/redcolour$ .
### - Start a pod named $\color[rgb]{1,0,1}nginx-configmap$ containing a single container using the $\color[rgb]{1,0,1}nginx$ image and mount the key you just created into the pod under directory $\color[rgb]{1,0,1}/some/path$
### - This pod must be created only those nodes which has label $\color[rgb]{1,0,1}disktype=ssd$

### Solution: 
#### Namespace = app-config
#### Role    =  config-role
#### verb    = create
#### resource = ConfigMap
#### user   = raja

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


https://kubernetes.io

Documentation --> Search -> Role
```
cat <<EOF>> qustion1.yaml
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
  name: raja
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: config-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

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
  nodeSelector:
    disktype: ssd
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
```

```
kubectl create -f qustion1.yaml 
```
```
kubectl create -f pods.yaml 
```
```
kubectl get pods -n app-config 
```
```
kubectl auth can-i create configmaps --as raja --namespace app-config
```
```
kubectl auth can-i create configmaps --as raja 
```

