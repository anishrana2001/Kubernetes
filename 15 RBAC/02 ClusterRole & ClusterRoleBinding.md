# Cluster and ClusterRoleBinding

## How to check $\color[rgb]{1,0,1} default ClusterRole$ in the Kubernetes Cluster?
```
kubectl get clusterrole
```

```yaml
cat <<EOF>> admin-cr.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
 name: admin-cr
rules:
 - apiGroups:
   - ""
   - "apps"
   resources: ["*"]
   verbs:
   - get
   - list
   - watch
   - create
   - update
   - patch
   - delete
EOF
```

```
kubectl apply -f admin-cr.yaml
```

	  
	  
```yaml
cat <<EOF>> admin-crb-1.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
 name: readwrite-crb1
roleRef:
 kind: ClusterRole
 name: admin-cr
 apiGroup: rbac.authorization.k8s.io
subjects:
- kind: User
  name: ashok
  apiGroup: rbac.authorization.k8s.io
EOF
```

```
kubectl apply -f admin-crb-1.yaml
```

## How to check ClusterRole is created successfully?
```
kubectl describe clusterrole/admin-cr 
```
```
kubectl get  clusterrole/admin-cr 
```

## How to check ClusterRoleBinding is created successfully?
```
kubectl describe clusterrolebindings.rbac.authorization.k8s.io readwrite-crb1
```
```
kubectl get clusterrolebindings.rbac.authorization.k8s.io readwrite-crb1
```

### How to check if user $\color[rgb]{1,0,1} ashok$ has rights to create deployment?
```
kubectl auth can-i create daemonsets --namespace app-team1 --as ashok
```


## How to clear the Lab?

```
kubectl get clusterrole,clusterrolebindings.rbac.authorization.k8s.io | grep admin-cr
```
```
kubectl delete clusterrole/admin-cr clusterrolebindings/readwrite-crb1
```

## OR
```
kubectl delete -f admin-cr.yaml -f admin-crb-1.yaml
```

### Remove the fiels that we created earlier.
```
rm -rf admin-cr.yaml admin-crb-1.yaml
```






## Question 1: use context-k8s
### You have been asked to create a new ClusterRole for a deployment pipeline and bind it to a specific ServiceAccount scoped to a specific namespace.

### Task -
### Create a new ClusterRole named $\color[rgb]{1,0,1} deployment-app-clusterrole$, which only allows to $\color[rgb]{1,0,1} create$ the following resource types:
### Deployment
### Stateful Set
### DaemonSet
### Create a new ServiceAccount named $\color[rgb]{1,0,1} cicd-app$ in the existing namespace $\color[rgb]{1,0,1} app-team1$.
### Bind the new ClusterRole deployment-app-clusterrole to the new ServiceAccount cicd-app, limited to the namespace app-team1.

## **Solution:**
### ClusterRole = deployment-app-clusterrole
### ServiceAccount = cicd-app
### namespace = app-team1

### First, we need to create namespace for service account user.
```
kubectl create namespace app-team1
```
### Create ClusterRole $\color[rgb]{1,0,1} (deployment-app-clusterrole)$ with verb (create) and allow resources ($\color[rgb]{1,0,1} Deployment,StatefulSet,DaemonSet$)
```
kubectl create clusterrole deployment-app-clusterrole --verb=create --resource=Deployment,StatefulSet,DaemonSet
```
### Create service account ($\color[rgb]{1,0,1} cicd-app$ under namespace $\color[rgb]{1,0,1} app-team1$
```
kubectl create serviceaccount cicd-app -n app-team1
```
### Now, create ClusterRoleBinding and Bind with ClusterRole $\color[rgb]{1,0,1} deployment-app-clusterrole$ and allow System user i.e serviceaccount.

```
kubectl create clusterrolebinding deploy-b --clusterrole=deployment-app-clusterrole --serviceaccount=app-team1:cicd-app -n app-team1
```
### Describe the ClusterRole & check the values.
```
kubectl describe clusterrole deployment-app-clusterrole
```

### Describe the ClusterRoleBinding & check the values.
```
kubectl describe clusterrolebindings.rbac.authorization.k8s.io/deploy-b
```
### Its time to check, if service account user has the rights to create deloyment?
```
kubectl auth can-i create  Deployment --as system:serviceaccount:app-team1:cicd-app --namespace=app-team1
```
