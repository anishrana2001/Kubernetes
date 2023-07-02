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

### How to check if user $\color[rgb]{1,0,1} ashok $ has rights to create deployment?
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



$\color[rgb]{1,0,1} hello$
