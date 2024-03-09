
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
