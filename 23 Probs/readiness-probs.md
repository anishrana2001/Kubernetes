Lilke Liveness probs, readiness probs can also be defined by 4 types. If checks fails then pos must be restarted.
1. HTTP (httpGet)
2. command (exec)
3. TCP (tcpSocket)
4. gRPC

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
cat <<EOF>> readiness-http.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: readiness-http
spec:
  containers:
  - name: readiness
    image: nginx
    volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d/
        - name: healthz
          mountPath: /usr/share/nginx/html/   
    readinessProbe:
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
kubectl apply -f readiness-http.yaml 
```

### Validate if our pod is in running state?

```
kubectl get pods/readiness-http 
```

### We can also check the description of this pod.

```
kubectl describe pods/readiness-http 
```

### Use the "-w" (watch) option to see the progress of this pod. Actually, we are going to stop the nginx service and will observe the behavior of this pod. It should restart the pod again. 
### Now, open the 2nd terminal and login into this pod.
```
kubectl exec -it readiness-http -- /bin/bash
```
### Stop the nginx service.
```
service nginx stop
```
### In the first terminal, we should see that our pod must restart. IF this is the case then we have done the lab for readiness HTTP probs successfully .

```
[root@master1 data]# kubectl get pods
NAME            READY   STATUS             RESTARTS        AGE
readiness-http   1/1     Running            3 (59s ago)     137m
```

## 2. command (exec)   

### Create the pod yaml file.
```yaml
cat <<EOF>> readiness-exec.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: readiness-exec
spec:
  containers:
  - name: readiness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    readinessProbe:
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
kubectl apply -f readiness-exec.yaml
```
### We must see, our pod is in running state.
```
[root@master1 data]# kubectl get pods
NAME            READY   STATUS    RESTARTS        AGE
readiness-exec   1/1     Running   0               27s
```
### describe this pod.

```
[root@master1 data]# kubectl describe pod/readiness-exec | tail
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  61s                default-scheduler  Successfully assigned default/readiness-exec to workernode1.example.com
  Normal   Pulling    61s                kubelet            Pulling image "registry.k8s.io/busybox"
  Normal   Pulled     59s                kubelet            Successfully pulled image "registry.k8s.io/busybox" in 1.887795573s
  Normal   Created    59s                kubelet            Created container readiness
  Normal   Started    59s                kubelet            Started container readiness
  Warning  Unhealthy  16s (x3 over 26s)  kubelet            readiness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    16s                kubelet            Container readiness failed readiness probe, will be restarted
[root@master1 data]#

```

### You will observe that pod is restarted. If this is the case, then we have completed this task. Let's check.

```
[root@master1 data]# kubectl get pods
NAME            READY   STATUS    RESTARTS        AGE
readiness-exec   1/1     Running   1 (25s ago)     100s
```


## 3. TCP (tcpSocket)
### A third type of readiness probe uses a TCP socket. With this configuration, the kubelet will attempt to open a socket to your container on the specified port. If it can establish a connection, it means that container is healthy, if not then container is not healthy.

### We can use the readiness-http config file as we have already created the configMaps. 
```yaml
cat <<EOF>> readiness-tcp.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: readiness-tcp
spec:
  containers:
  - name: readiness
    image: nginx
    volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d/
        - name: healthz
          mountPath: /usr/share/nginx/html/   
    readinessProbe:                   # readiness probs config start
      tcpSocket:                     # tcpSocket
        port: 80                     # Our nginx service is running on port 80, so this readiness probs can reach to port 80.
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
kubectl create -f readiness-tcp.yaml
```
### Check the status of this pod. Please notice the "RESTART" column. It should be 0.
```
[root@master1 data]# kubectl get pods/readiness-tcp 
NAME           READY   STATUS    RESTARTS   AGE
readiness-tcp   1/1     Running   0          105s
```
### Stop the NGINX service once. We should see that our pod will be restarted and working again.
```
[root@master1 data]# kubectl exec readiness-tcp -- service nginx stop
command terminated with exit code 137
```
### One can observe that pod restarted 1 time.
```
[root@master1 data]# kubectl get pods/readiness-tcp 
NAME           READY   STATUS    RESTARTS     AGE
readiness-tcp   1/1     Running   1 (4s ago)   115s
[root@master1 data]#
```

## 4. gRPC 

### Official link of gRPC is https://github.com/grpc/grpc/blob/master/doc/health-checking.md
### In this, we check directly the service is running on the configured port. For that reason, we can use the readiness-http config and modify it as per our requirement. 
```yaml
cat <<EOF>> readiness-rpc.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: readiness-rpc
spec:
  containers:
  - name: readiness
    image: nginx
    volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d/
        - name: healthz
          mountPath: /usr/share/nginx/html/   
    readinessProbe:                  # readiness probs config start
      grpc:                         # grpc config start.
        port: 80                    # In this example, the readiness probe sends a gRPC request to port 80 of the container.
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
kubectl apply -f readiness-rpc.yaml
```
### Check the pod status.
```
kubectl get pods/readiness-rpc
```
### If we reload the service, we will observe pod is restarted.
```
kubectl exec readiness-rpc -- service nginx reload
```
```
kubectl get pods/readiness-rpc 
```


### Below is the reference for you.
```
[root@master1 data]# kubectl get pods/readiness-rpc 
NAME           READY   STATUS    RESTARTS   AGE
readiness-rpc   1/1     Running   0          7s
[root@master1 data]# kubectl exec readiness-rpc -- service nginx reload
Reloading nginx: nginx.
[root@master1 data]# kubectl get pods/readiness-rpc 
NAME           READY   STATUS    RESTARTS      AGE
readiness-rpc   1/1     Running   1 (12s ago)   43s
[root@master1 data]# kubectl exec readiness-rpc -- service nginx stop
command terminated with exit code 137
[root@master1 data]# kubectl get pods/readiness-rpc 
NAME           READY   STATUS      RESTARTS      AGE
readiness-rpc   0/1     Completed   1 (22s ago)   53s
[root@master1 data]# kubectl get pods/readiness-rpc 
NAME           READY   STATUS    RESTARTS      AGE
readiness-rpc   1/1     Running   2 (16s ago)   65s
[root@master1 data]# 
```
