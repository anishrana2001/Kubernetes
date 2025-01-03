# We can also add both the probes in a single manisfest file.

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
### Create the configmaps.  
```
kubectl create configmap healthz --from-file=healthz
kubectl create configmap nginx-conf --from-file=default.conf
```


```
cat <<EOF>> liveness-readiness-http.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: live-readi-http
spec:
  containers:
  - name: live-readi-container
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
      initialDelaySeconds: 2
      periodSeconds: 10
      failureThreshold: 1
    readinessProbe:
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 2
      periodSeconds: 10
      failureThreshold: 1
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
kubectl apply -f liveness-readiness-http.yaml
```
## Check the pod status

```
kubectl get pods/live-readi-http
```

```
kubectl describe pods/live-readi-http | tail
```
### Use the "-w" (watch) option to see the progress of this pod. Actually, we are going to stop the nginx service and will observe the behavior of this pod. It should restart the pod again. 
```
kubectl get pods/live-readi-http  -w
```
### Now, open the 2nd terminal and login into this pod.

### Stop the nginx service.
```
kubectl exec live-readi-http -- service nginx stop
```
### In the first terminal, we should see that our pod must restart. IF this is the case then we have done the lab for live-readi HTTP probs successfully .
## Clear the Lab.
```
kubectl delete -f  liveness-readiness-http.yaml
kubectl delete configmaps nginx-conf healthz
 rm -rf liveness-readiness-http.yaml healthz default.conf
```

