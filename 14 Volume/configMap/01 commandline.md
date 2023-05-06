
## How to create the configMap from command line?
```
kubectl create configmap command-configmap --from-literal=NGINX_PORT=8080 --from-literal=language=english --from-literal=environments=production,development,test
```
## Create the deployment Yaml file.
```
cat <<EOF>> command-env.yaml
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
            name: command-configmap
EOF
```
## Create the deployment from above yaml file.
```
kubectl apply -f command-env.yaml
```

## Check the deployment 
```
kubectl get deployments.apps
```
## Check if variables are correct inserted?

```
kubectl exec -it $(kubectl get pods | grep env-deploy1 | grep Running | awk '{print $1}' | head -n 1) -- env
```

## How to clear the lab.
```
kubectl delete deployments.apps env-deploy1
kubectl delete configmap command-configmap
kubectl delete configmaps command-configmap
rm -f command-env.yaml
```

 
 
 
 
 
 
 
 
 
 
