
# Connect a Frontend to a Backend Using Services

## Create the yaml files for Backend service, named "hello-srv".
```
cat <<EOF>> hello-srv.yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-srv
spec:
  selector:
    app: hello
    tier: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: http
EOF
```
## Create the yaml files for Backend Deployment.
```
cat <<EOF>> backend-srv
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-srv
spec:
  selector:
    matchLabels:
      app: hello
      tier: backend
  replicas: 3
  template:
    metadata:
      labels:
        app: hello
        tier: backend
    spec:
      containers:
        - name: backend-container
          image: "gcr.io/google-samples/hello-go-gke:1.0"
          ports:
            - name: http
              containerPort: 80
EOF
```
## Create the yaml files for Frontend service, named "frontend-srv".
```
cat <<EOF>> frontend-srv.yaml
apiVersion: v1     
kind: Service
metadata:
  name: frontend-srv       
spec:
  selector:
    app: hello
    tier: frontend
  ports:
  - protocol: "TCP"
    port: 80
    targetPort: 80
  type: NodePort
EOF
```

## Create the yaml files for Frontend Deployment.
```
cat <<EOF>> frontend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  selector:
    matchLabels:
      app: hello
      tier: frontend
      track: stable
  replicas: 1
  template:
    metadata:
      labels:
        app: hello
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx
EOF
```
## Create the Frontend & Backend deployments and Services.
```
kubectl apply -f hello-srv.yaml -f frontend-srv.yaml -f frontend-deployment.yaml -f backend-deployment.yaml
```

## Explore the deployments that we created.
```
kubectl get deployments.apps
```
## Explore the services that we created.
```
kubectl get service -o wide
```
## Explore the endspoints that we created.
```
kubectl get endpoints
```

### Let's try to open the Frontend container web GUI. 

```
curl http://
```

```
var_frontend_srv=`kubectl get endpoints | grep frontend | awk '{print $2}' | grep -v END` ; curl http://$var_frontend_srv
```

### Differnet page should be open when we curl the backend container.

```
var_hello_srv=`kubectl get endpoints | grep hello-srv | awk '{print $2}' | grep -v END | cut -d "," -f1` ; curl http://$var_hello_srv
```

## Login to Frontend Container 
### Now, we are going to modify the Frontend container to forward the traffic to backend service.
```
kubectl exec -it frontend-deployment-8cc58b98f-js4m5 -- /bin/bash
cd /etc/nginx/conf.d/
cat <<EOF>> frontend.conf
# The identifier Backend is internal to nginx, and used to name this specific upstream
upstream Backend {
    # hello is the internal DNS name used by the backend Service inside Kubernetes
    server hello-srv;
}
server {
    listen 80;

    location / {
        # The following statement will proxy traffic to the upstream named Backend
        proxy_pass http://Backend;
    }
}
EOF

nginx -help
nginx -s reload
exit

```


## Now, check the frontend container.
### This time, when we curl the frontent container, we should get the response from the backend container.

```
var_frontend_srv=`kubectl get endpoints | grep frontend | awk '{print $2}' | grep -v END` ; curl http://$var_frontend_srv
```
## Check the VM IPs 
```
ifconfig
kubectl get service -o wide

VM_IP:PORT_NUMBER
```
