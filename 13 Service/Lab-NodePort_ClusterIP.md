

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

```
kubectl apply -f hello-srv.yaml -f frontend-srv.yaml -f frontend-deployment.yaml -f backend-deployment.yaml
```

```
kubectl get deployments.apps
```

```
kubectl get service -o wide
```

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

### Differnet page will be open whene we curl the backend container.

```
var_hello_srv=`kubectl get endpoints | grep hello-srv | awk '{print $2}' | grep -v END | cut -d "," -f1` ; curl http://$var_hello_srv
```

### Login to Frontend Container 
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


### Now, check the frontend container

```
var_frontend_srv=`kubectl get endpoints | grep frontend | awk '{print $2}' | grep -v END` ; curl http://$var_frontend_srv
```
