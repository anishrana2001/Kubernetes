# Liveness probs can be defined by 4 types. If checks fails then pos must be restarted.
# 1. HTTP (httpGet)
# 2. command (exec)
# 3. TCP (tcpSocket)
# 4. gRPC 


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
```
kubectl apply -f liveness-http.yaml 
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

## Let's move forward towards liveness command probs.

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


## Define a TCP liveness probe
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
      initialDelaySeconds: 2         # First attempt will start after 2 seconds.
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
### Check the status of this pod.
```
[root@master1 data]# kubectl get pods/liveness-tcp 
NAME           READY   STATUS    RESTARTS   AGE
liveness-tcp   1/1     Running   0          105s

[root@master1 data]# kubectl exec liveness-tcp -- service nginx stop
command terminated with exit code 137

[root@master1 data]# kubectl get pods/liveness-tcp 
NAME           READY   STATUS    RESTARTS     AGE
liveness-tcp   1/1     Running   1 (4s ago)   115s
[root@master1 data]#
```
### One can observe that pod restarted 1 time.

