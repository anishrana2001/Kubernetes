```
kubectl create namespace core 
```

### Role for developement Team (ReadWrite)
```yaml
cat <<EOF>> dev-role1.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dev-role-01
  namespace: core
rules:
- apiGroups:
  - ""
  resources: ["*"]
  verbs:
  - get
  - list
  - watch
  - create
  - update
  - delete
EOF
```

```
kubectl apply -f dev-role1.yaml
```
### RoleBinding for developement Team 
```yaml
cat <<EOF>> dev-rb-readwrite-01.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
 name: dev-rb-readwrite-01
 namespace: core
subjects:
- kind: User
  name: dev
  apiGroup: rbac.authorization.k8s.io
roleRef:
 kind: Role
 name: dev-role-01
 apiGroup: rbac.authorization.k8s.io
EOF
```
```
kubectl apply -f dev-rb-readwrite-01.yaml
```

## How to check if dev user has the privileges?
```
kubectl auth can-i list secrets --namespace core --as dev
```

[root@master1 rbac1]# kubectl auth can-i  **list secrets** --namespace core --as **dev**
**yes**
[root@master1 rbac1]#

[root@master1 rbac1]# kubectl auth can-i **list secrets** --namespace core --as **raja**
**no**

## Role for Opereation Team (ReadOnly)
```yaml
cat <<EOF>> ops-role1.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: core
  name: ops-role-01
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log", "deployment"]
  verbs: ["get", "list", "watch"]
EOF
```
```
kubectl create -f ops-role1.yaml
```
### RoleBinding for Operation Team 
```yaml
cat <<EOF>> ops-rb-readonly-01.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
 name: ops-rb-readonly-01
 namespace: core
subjects:
- kind: User
  name: raja
  apiGroup: rbac.authorization.k8s.io
roleRef:
 kind: Role
 name: ops-role-01
 apiGroup: rbac.authorization.k8s.io
EOF
```
```
kubectl create -f ops-rb-readonly-01.yaml
```

## How to check if **raja** user has the privileges?

[root@master1 rbac1]# kubectl auth can-i **list pods** --namespace core --as raja
**yes**
[root@master1 rbac1]# kubectl auth can-i **list secret** --namespace core --as raja
**no**

## How to clear the lab
```
kubectl delete -f dev-role1.yaml -f dev-rb-readwrite-01.yaml -f ops-rbac1.yaml -f ops-rb-readonly-01.yaml
kubectl delete namespaces core
```
