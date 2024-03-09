
## We can employ readiness and liveness probes for our containers but what if we have a container that hosts an application, such as a legacy application, which needs a more time just to start up. In this situation, we do not have a faulty application, we just have a slow starting application. Implementing a liveness probe would put it in a loop where the container is restarted before completing the start up process each time.
#
#
# Startup probs can be defined by 4 types. If checks fails then pos must be restarted.
# 1. HTTP (httpGet)
# 2. command (exec)
# 3. TCP (tcpSocket)
# 4. gRPC 
#
#

## 1. HTTP (httpGet)

### Create a NGINX configuration file on local host (master node).

```
cat <<EOF>> default.conf 
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

server { 
       location /healthz {
        access_log off;
        return 200;
		}
	}
EOF
```
### After that, we can create a page for health check.

```
cat <<EOF>> healthz 
Server is up and running fine.
EOF
```

### Create a configMap from these files.
```
kubectl create configmap healthz --from-file=healthz 
```
```
kubectl create configmap nginx-conf --from-file=default.conf
```

### Create a pod yaml file.

```
cat <<EOF>> startup-http.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: startup
  name: startup
spec:
  containers:
  - name: startup-http-container
    image: nginx
    volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d/
        - name: healthz
          mountPath: /usr/share/nginx/html/
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
      failureThreshold: 2
      periodSeconds: 10
    startupProbe:
      httpGet:
        path: /healthz
        port: 80
      failureThreshold: 2
      periodSeconds: 10
  volumes:
  - name: config
    configMap:
      name: nginx-conf
  - name: healthz
    configMap:
      name: healthz
EOF
```

```
kubectl apply -f startup-http.yaml
```

### Check the status of startup pod

```
kubectl get pods/startup
```
### Now, execute the below command with "-w" option. This time, we are going to stop the nginx service and will observe the behaviour of our startup pod.
```
kubectl get pods startup -w
```

### Open the another terminal, and stop the nginx service.

```
kubectl exec startup  -- service nginx stop
```


You can configure the startup probes in the same way what we have done for liveness or readiness probes.


## Clear the lab.
```
kubectl delete -f startup-http.yaml
kubectl delete configmaps nginx-conf healthz
rm -rf startup-http.yaml healthz default.conf
```
