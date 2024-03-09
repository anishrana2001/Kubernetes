# We can also add both the probes in a single manisfest file.

## First, create the configmaps. For this lab, please perform the liveness-probe lab. 
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
  name: live-readi
spec:
  containers:
  - name: live-readi-container
    image: nginx
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600	
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
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
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
kubectl get pods/live-readi
```

```
kubectl describe pods/live-readi
```

