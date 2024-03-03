# Liveness probs can be defined by 4 types.
# 1. HTTP (httpGet)
# 2. command (exec)
# 3. TCP (tcpSocket)
# 4. gRPC 


## 1. HTTP (httpGet)

## Create a NGINX configuration file on local host (master node).

```yaml
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
## After that, we can create a page for health check.

```
cat <<EOF>> healthz 
Server is up and running fine.
EOF
```

## Create a configMap from these files.
```
kubectl create configmap healthz --from-file=healthz 
```
```
kubectl create configmap nginx-conf --from-file=default.conf
```
```
kubectl apply -f liveness-http.yaml 
```

## Create a pod yaml file.
```yaml
cat <<EOF>> liveness-http.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: nginx
    volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d/
        - name: healthz
          mountPath: /usr/share/nginx/html/   
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
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
## Let's create a pod from the above yaml file.

```
kubectl apply -f liveness-http.yaml 
```

## Validate if our pod is in running state?

```
kubectl get pods/liveness-http 
```

## We can also check the description of this pod.

```
kubectl describe pods/liveness-http 
```

## Use the "-w" (watch) option to see the progress of this pod. Actually, we are going to stop the nginx service and will observe the behaviour of this pod. It should restart the pod again. 
## Now, open the 2nd terminal and login in this pod.
```
kubectl exec -it liveness-http -- /bin/bash
```
## Stop the nginx service.
```
service nginx stop
```
## In the first terminal, we should see that our pod must restart. IF this is the case then we have done the lab for liveness HTTP probs successfully .




