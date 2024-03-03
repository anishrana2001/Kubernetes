# Liveness probs can be defined by 4 types. If checks fails then pos must be restarted.
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
### Let's create a pod from the above yaml file.

```
kubectl apply -f liveness-http.yaml 
```

### Validate if our pod is in running state?

```
kubectl get pods/liveness-http 
```

### We can also check the description of this pod.

```
kubectl describe pods/liveness-http 
```

### Use the "-w" (watch) option to see the progress of this pod. Actually, we are going to stop the nginx service and will observe the behavior of this pod. It should restart the pod again. 
### Now, open the 2nd terminal and login into this pod.
```
kubectl exec -it liveness-http -- /bin/bash
```
### Stop the nginx service.
```
service nginx stop
```
### In the first terminal, we should see that our pod must restart. IF this is the case then we have done the lab for liveness HTTP probs successfully .

```
[root@master1 data]# kubectl get pods
NAME            READY   STATUS             RESTARTS        AGE
liveness-http   1/1     Running            3 (59s ago)     137m
```
#
#

## 2. command (exec)   

### Create the pod yaml file.
```yaml
cat <<EOF>> liveness-exec.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
EOF
```

### Create POD resource.
```
kubectl apply -f liveness-exec.yaml
```
### We must see, our pod is in running state.
```
[root@master1 data]# kubectl get pods
NAME            READY   STATUS    RESTARTS        AGE
liveness-exec   1/1     Running   0               27s
```
### describe this pod.

```
[root@master1 data]# kubectl describe pod/liveness-exec | tail
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  61s                default-scheduler  Successfully assigned default/liveness-exec to workernode1.example.com
  Normal   Pulling    61s                kubelet            Pulling image "registry.k8s.io/busybox"
  Normal   Pulled     59s                kubelet            Successfully pulled image "registry.k8s.io/busybox" in 1.887795573s
  Normal   Created    59s                kubelet            Created container liveness
  Normal   Started    59s                kubelet            Started container liveness
  Warning  Unhealthy  16s (x3 over 26s)  kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    16s                kubelet            Container liveness failed liveness probe, will be restarted
[root@master1 data]#

```

### You will observe that pod is restarted. If this is the case, then we have completed this task. Let's check.

```
[root@master1 data]# kubectl get pods
NAME            READY   STATUS    RESTARTS        AGE
liveness-exec   1/1     Running   1 (25s ago)     100s
```

#
#

## 3. TCP (tcpSocket)
### A third type of liveness probe uses a TCP socket. With this configuration, the kubelet will attempt to open a socket to your container on the specified port. If it can establish a connection, it means that container is healthy, if not then container is not healthy.

### We can use the liveness-http config file as we have already created the configMaps. 
```yaml
cat <<EOF>> liveness-tcp.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-tcp
spec:
  containers:
  - name: liveness
    image: nginx
    volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d/
        - name: healthz
          mountPath: /usr/share/nginx/html/   
    livenessProbe:                   # Liveness probs config start
      tcpSocket:                     # tcpSocket
        port: 80                     # Our nginx service is running on port 80, so this liveness probs can reach to port 80.
      initialDelaySeconds: 2         # Once the container starts, it will wati for 2 second and then it send the request. 
      periodSeconds: 10              # Retry afer 10 seconds.
      failureThreshold: 3            # After 3 failures a container is classified as failed.
  volumes:
  - name: config
    configMap:
      name: nginx-conf
  - name: healthz
    configMap:
      name: healthz
EOF
```
### Create this POD.
```
kubectl create -f liveness-tcp.yaml
```
### Check the status of this pod. Please notice the "RESTART" column. It should be 0.
```
[root@master1 data]# kubectl get pods/liveness-tcp 
NAME           READY   STATUS    RESTARTS   AGE
liveness-tcp   1/1     Running   0          105s
```
### Stop the NGINX service once. We should see that our pod will be restarted and working again.
```
[root@master1 data]# kubectl exec liveness-tcp -- service nginx stop
command terminated with exit code 137
```
### One can observe that pod restarted 1 time.
```
[root@master1 data]# kubectl get pods/liveness-tcp 
NAME           READY   STATUS    RESTARTS     AGE
liveness-tcp   1/1     Running   1 (4s ago)   115s
[root@master1 data]#
```
#
#
## 4. gRPC 

### Official link of gRPC is "https://github.com/grpc/grpc/blob/master/doc/health-checking.md"
### In this, we check directly the service is running on the configured port. For that reason, we can use the liveness-http config and modify it as per our requirement. 
```yaml
cat <<EOF>> liveness-rpc.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-rpc
spec:
  containers:
  - name: liveness
    image: nginx
    volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d/
        - name: healthz
          mountPath: /usr/share/nginx/html/   
    livenessProbe:                  # Liveness probs config start
      grpc:                         # grpc config start.
        port: 80                    # In this example, the liveness probe sends a gRPC request to port 80 of the container.
      initialDelaySeconds: 2        # Once the container starts, it will wati for 2 second and then it send the request. 
      periodSeconds: 10             # Retry afer 10 seconds.
      failureThreshold: 3           # After 3 failures a container is classified as failed.
  volumes:
  - name: config
    configMap:
      name: nginx-conf
  - name: healthz
    configMap:
      name: healthz
EOF
```
### Create the pod.
```
kubectl apply -f liveness-rpc.yaml
```
### Check the pod status.
```
kubectl get pods/liveness-rpc
```
### If we reload the service, we will observe pod is restarted.
```
kubectl exec liveness-rpc -- service nginx reload
```
```
kubectl get pods/liveness-rpc 
```


### Below is the reference for you.
```
[root@master1 data]# kubectl get pods/liveness-rpc 
NAME           READY   STATUS    RESTARTS   AGE
liveness-rpc   1/1     Running   0          7s
[root@master1 data]# kubectl exec liveness-rpc -- service nginx reload
Reloading nginx: nginx.
[root@master1 data]# kubectl get pods/liveness-rpc 
NAME           READY   STATUS    RESTARTS      AGE
liveness-rpc   1/1     Running   1 (12s ago)   43s
[root@master1 data]# kubectl exec liveness-rpc -- service nginx stop
command terminated with exit code 137
[root@master1 data]# kubectl get pods/liveness-rpc 
NAME           READY   STATUS      RESTARTS      AGE
liveness-rpc   0/1     Completed   1 (22s ago)   53s
[root@master1 data]# kubectl get pods/liveness-rpc 
NAME           READY   STATUS    RESTARTS      AGE
liveness-rpc   1/1     Running   2 (16s ago)   65s
[root@master1 data]# 
```
