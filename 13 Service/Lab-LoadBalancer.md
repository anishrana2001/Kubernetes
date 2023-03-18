
## Connect a Frontend to a Backend Using LoadBalancer Service.
### Create the yaml files for Frontend deployment, frontend service, backend deployment and backend service.

```
cat <<EOF>> service_project.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
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
---
apiVersion: v1
kind: Service
metadata:
  name: hello-srv
spec:
  selector:
    app: hellokubec
    tier: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: http
---
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
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  selector:
    matchLabels:
      app: hello
      tier: frontend
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
kubectl apply -f service_project.yaml
```

## Install the External LoadBalancer MetalLB
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.9/config/manifests/metallb-native.yaml
```

### Configure the IP range for MetalLB.
```
cat <<EOF>> IPAddressPool.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.100-192.168.1.110
EOF
```

```
kubectl create -f IPAddressPool.yaml
```
### Now, External LB assigned the IP address, Look at External -IP coloumn.
```
[root@master1 data-service]# kubectl get service
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
frontend-srv   LoadBalancer   10.100.56.80    192.168.1.100   80:30747/TCP   9s
hello-srv      ClusterIP      10.110.117.12   <none>          80/TCP         9s
kubernetes     ClusterIP      10.96.0.1       <none>          443/TCP        100d
```

 ### We can now, curl the external IP and get the desired result.
```
[root@master1 data-service]# curl http://192.168.1.100
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### Let's, forward the traffic to backend server, so modify the Nginx's configuration.
```
[root@master1 data-service]# kubectl exec -it frontend-deployment-8cc58b98f-5tkff -- /bin/sh

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
# ls -ltr  
total 8
-rw-r--r-- 1 root root 1093 Mar 18 16:03 default.conf
-rw-r--r-- 1 root root  381 Mar 18 16:05 frontend.conf                                                                                                         ^C        n	  

bash
root@frontend-deployment-8cc58b98f-5tkff:/etc/nginx/conf.d# mv default.conf /tmp/
root@frontend-deployment-8cc58b98f-5tkff:/etc/nginx/conf.d# nginx -s reload
2023/03/18 16:06:42 [notice] 41#41: signal process started
root@frontend-deployment-8cc58b98f-5tkff:/etc/nginx/conf.d# exit
exit
exit
```
### After the modification, we can now have the desired result.
```
[root@master1 data-service]# curl http://192.168.1.100
{"message":"Hello"}
[root@master1 data-service]# kubectl get service
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
frontend-srv   LoadBalancer   10.100.56.80    192.168.1.100   80:30747/TCP   3m50s
hello-srv      ClusterIP      10.110.117.12   <none>          80/TCP         3m50s
kubernetes     ClusterIP      10.96.0.1       <none>          443/TCP        100d
[root@master1 data-service]# 
```

