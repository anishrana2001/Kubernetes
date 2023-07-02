Role and RoleBinding.

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
## How to check Role & RoleBinding are created successfully? 
### For Role only.
```
kubectl -n core get role
```
### For RoleBinding only.
```
kubectl -n core get rolebindings
```
### For both.
```
kubectl -n core get role,rolebinding
```

## Describe the Role
```
kubectl -n core describe role/dev-role-01
```

## Describe RoleBinding.
```
kubectl -n core describe rolebindings/dev-rb-readwrite-01
```

## How to check if dev user has the privileges?
```
kubectl auth can-i list secrets --namespace core --as dev
```

[root@master1 rbac1]# kubectl auth can-i  **list secrets** --namespace core --as **dev**

**yes**


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

## Question 1: You have been asked to create a new Role for a deployment pipeline and bind it to a user raja. 

### Task - Create a new Role named deployment-role, which only allows in the existing namespace app-team1 to create the following resource types:
#### ✑ Deployment
#### ✑ StatefulSet
#### ✑ DaemonSet


Solution: We have asked to create role "deployment-role" and indirectly asked to create a **RoleBinding**.

verb = **create**

resources = **Deployment,StatefulSet,DaemonSet**

user = **raja **

namespace = **app-team1**

### We can complete this question through command line as well as Yaml file. 

#### Command line solution.

#### Check if Namespace is already created or not.
```
kubectl get namespace 
```
####  If namespace is not there then create it manually. 
```
kubectl create namespace app-team1
```
#### Create a Role object "deployment-role" in the namespace "app-team1", verb should be "create", resources= Deployment,StatefulSet,DaemonSet
```
kubectl create -n app-team1 role deployment-role --verb=create --resource=Deployment,StatefulSet,DaemonSet
```
#### Create a RoleBinding object ( we can give if it is not mentioned in the question), and mention the Role object name "deployment-role" and then user = raja
```
kubectl create -n app-team1 rolebinding deploy-b --role=deployment-role --user=raja
```
#### We can describe the object rolebinding. 
```
kubectl -n app-team1 describe rolebindings.rbac.authorization.k8s.io/deploy-b
```
#### We can also describe the Role object.
```
kubectl -n app-team1 describe role deployment-role
```
#### We can also check if user raja has the privileges to create deployment or not.
```
kubectl auth can-i create  Deployment --as raja -n app-team1
```
#### What happen if I use pod instead of deployment?
```
kubectl auth can-i create  pods --as raja -n app-team1
```

### How to delete Role, RoleBinding and namespace?
#### First, we need to delete the Rolebinding because Rolebinding is bind with Role.
##### How to Delete the RoleBinding?

#### First, we must identify where is my rolebinding.
```
kubectl -n app-team1 get rolebindings.rbac.authorization.k8s.io/deploy-b
```
```
kubectl delete -n app-team1 rolebindings.rbac.authorization.k8s.io/deploy-b 
```

##### How to delete the Role?
```
kubectl -n app-team1 get role/deployment-role
```
```
kubectl -n app-team1 delete role/deployment-role
```
##### How to Delete the namespace?
##### First, we must check there is no object is running.
```
kubectl get -n app-team1 all
```
```
kubectl delete namespaces app-team1
```

