# Question 1
## Create a new NetworkPolicy named abc-name in the existing namespace orange.
## Ensure that the new NetworkPolicy allows Pods in namespace core to connect to port 9000 of Pods in namespace orange.
## Further ensure that the new NetworkPolicy:
## ✑ does not allow access to Pods, which don't listen on port 9000
## ✑ does not allow access from Pods, which are not in namespace 


# Solution: 

## What is given in the question?

### namespace names : orange & core

```
kubectl create namespace orange 
kubectl create namespace core
```



### Let's create 3 pods, 2 on core namespace and 1 on orange namespace.

```
kubectl -n orange run --image=nginx --labels "app=orange" orange-pod1
```

```
kubectl -n core run --image=nginx --labels "app=core" core-pod1
```


```
kubectl -n core run --image=nginx --labels "app=core" core-pod2
```
 
```
cat <<EOF>> orange-pod1.conf
server {
    listen       9000;
    listen  [::]:9000;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  orange-pod1-index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
EOF
```


```
kubectl -n orange cp orange-pod1.conf orange-pod1:/etc/nginx/conf.d/ -c orange-pod1
```
```
kubectl -n orange cp orange-pod1-index.html orange-pod1:/usr/share/nginx/html/ -c orange-pod1
```
```
kubectl exec -it -n orange pods/orange-pod1 -- service nginx reload
```


```
kubectl -n orange get all --show-labels -owide
```


### You can also able to connect to this port from core pods/orange-pod1
```
kubectl -n core exec -it pods/core-pod1 -- curl http://172.16.14.114:9000
```

```
kubectl get ns/orange --show-labels
```


### Now, lets create the NetworkPolicy.

```
cat <<EOF>> abc-name.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: abc-name
  namespace: orange
spec:
  podSelector:
    matchLabels:
      app: orange
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: core
      ports:
        - protocol: TCP
          port: 9000
EOF
```

```
kubectl apply -f abc-name.yaml
```

```
kubectl describe -n orange netpol/abc-name
```


```
kubectl -n core exec -it pods/core-pod1 -- curl orange-pod1_IP:9000

```

## We can also create 1 more pod on Orange NS and allow port 2222 and check if core namespace pods can access it ?

```
cat <<EOF>> orange-pod1-index.html
orange-pod1 web server is up on port 9000
EOF
```
```
```
kubectl -n orange run --image=nginx --labels "app=orange" orange-pod2
```

cat <<EOF>> orange-pod2.conf
server {
    listen       2222;
    listen  [::]:2222;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  orange-pod2-index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
EOF
```

```
cat <<EOF>> orange-pod2-index.html
db2 web server is up on port 2222
EOF
```


```
kubectl -n orange cp orange-pod2.conf orange-pod2:/etc/nginx/conf.d/ -c orange-pod2
```
```
kubectl -n orange cp orange-pod2-index.html orange-pod2:/usr/share/nginx/html/ -c orange-pod2
```
```
kubectl exec -it -n orange pods/orange-pod2 -- service nginx reload
```

### You must able to connect on port 9000 from orange-pod2
```
kubectl -n orange exec -it pods/orange-pod2 -- curl http://orange-pod1_IP:9000
```
### 2222 port is also accessable from core namespace.
```
kubectl -n core exec -it pods/core-pod1 -- curl http://orange-pod1_IP:2222
```

```
kubectl -n core exec -it pods/core-pod1 -- curl http://orange-pod1_IP:2222
```


# How to clear the lab for question 1?
```
kubectl -n orange delete pods/orange-pod1 pods/orange-pod2 
kubectl -n core delete pods/core-pod1 pods/core-pod2
kubectl delete -f abc-name.yaml
kubectl delete namespaces/orange namespaces/core
rm -f abc-name.yaml orange-pod1.conf orange-pod1-index.html orange-pod2-index.html orange-pod2.conf
```


# Question 2: 
## There was a security incident where an hacker was able to access the whole cluster from a single hacked backend Pod (backend0).
## To curb this situation, Kubernetes administrator ask you to create a NetworkPolicy called portblock-np in Namespace project-app. It should allow the backend0 pod only to:
##  -  Connect to frontent pod on port 8080
##  -  Connect to nginx pod on port 9090

## Use the app label of Pods in your policy.
## After implementation, connections from backend0 pods to toolbox pod on port 3333 should for example no longer work.

# Solution: 

## Let's create the setup on our lab. Also, it would be helpful to you to setup your own lap for exam preparation.

```
kubectl create namespace project-app
```
```
kubectl -n project-app run --image=nginx --labels "app=frontent" frontent
```



```
cat <<EOF>> frontent.conf
server {
    listen       8080;
    listen  [::]:8080;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  frontent-index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
EOF
```

```
cat <<EOF>> frontent-index.html
frontent web server is up on port 8080
EOF
```
```
cat <<EOF>> nginx.conf
server {
    listen       9090;
    listen  [::]:9090;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  nginx-index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
EOF
```

```
cat <<EOF>> nginx-index.html
nginx web server is up on port 9090
EOF
```

```
cat <<EOF>> toolbox.conf
server {
    listen       3333;
    listen  [::]:3333;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  toolbox-index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
EOF
```
```
cat <<EOF>> toolbox-index.html
toolbox web server is up on port 3333
EOF
```
```
kubectl create namespace project-app
```
```
kubectl -n project-app run --image=nginx --labels "app=frontent" frontent
```
```
kubectl -n project-app run --image=nginx --labels "app=nginx" nginx
```
```
kubectl -n project-app run --image=nginx --labels "app=toolbox" toolbox
```
```
kubectl -n project-app run --image=nginx --labels "app=backend" backend0
```
```
kubectl get pods -n project-app --show-labels -o wide
```

```
kubectl -n project-app cp frontent.conf frontent:/etc/nginx/conf.d/ -c frontent
```
```
kubectl -n project-app cp frontent-index.html frontent:/usr/share/nginx/html/ -c frontent
```
```
kubectl exec -it -n project-app pods/frontent -- service nginx reload
```

```
kubectl -n project-app cp nginx.conf nginx:/etc/nginx/conf.d/ -c nginx
```
```
kubectl -n project-app cp nginx-index.html nginx:/usr/share/nginx/html/ -c nginx
```
```
kubectl exec -it -n project-app pods/nginx -- service nginx reload
```
```
kubectl -n project-app cp toolbox.conf toolbox:/etc/nginx/conf.d/ -c toolbox
```
```
kubectl -n project-app cp toolbox-index.html toolbox:/usr/share/nginx/html/ -c toolbox
```
```
kubectl exec -it -n project-app pods/toolbox -- service nginx reload
```
```
kubectl get pods -n project-app --show-labels -o wide
```
```
kubectl -n project-app exec -it  frontent -- curl  172.16.133.178:8080
```
```
kubectl -n project-app exec -it  nginx -- curl  $(kubectl -n project-app get pod/nginx -o wide | awk '{print $6}' | grep -v IP):9090
```
```
kubectl -n project-app exec -it toolbox -- curl  $(kubectl -n project-app get pod/toolbox -o wide | awk '{print $6}' | grep -v IP):3333
```
```
kubectl -n project-app exec -it backend0 -- curl $(kubectl -n project-app get pod/nginx -o wide | awk '{print $6}' | grep -v IP):9090
```
```
kubectl -n project-app exec -it backend0 -- curl $(kubectl -n project-app get pod/toolbox -o wide | awk '{print $6}' | grep -v IP):3333
```
```
cat <<EOF>> npbackup.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: project-app
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: frontent
      ports:
        - protocol: TCP
          port: 8080
    - to:
        - podSelector:
            matchLabels:
              app: nginx
      ports:
        - protocol: TCP
          port: 9090
EOF
```

```
kubectl -n project-app exec -it backend0 -- curl $(kubectl -n project-app get pod/frontent -o wide | awk '{print $6}' | grep -v IP):8080
```
```
kubectl -n project-app exec -it backend0 -- curl $(kubectl -n project-app get pod/nginx -o wide | awk '{print $6}' | grep -v IP):9090
```
```
kubectl -n project-app exec -it backend0 -- curl $(kubectl -n project-app get pod/toolbox -o wide | awk '{print $6}' | grep -v IP):3333
```
```
kubectl apply -f npbackup.yaml
```
```
kubectl -n project-app exec -it backend0 -- curl $(kubectl -n project-app get pod/frontent -o wide | awk '{print $6}' | grep -v IP):8080
```
```
kubectl -n project-app exec -it backend0 -- curl $(kubectl -n project-app get pod/nginx -o wide | awk '{print $6}' | grep -v IP):9090
```
```
kubectl -n project-app exec -it backend0 -- curl $(kubectl -n project-app get pod/toolbox -o wide | awk '{print $6}' | grep -v IP):3333
```

# How to clear the lab?
```
kubectl -n project-app delete pod/frontent 
kubectl -n project-app delete pod/nginx
kubectl -n project-app delete pod/toolbox 
kubectl -n project-app delete pod/backend0 
kubectl get all -n project-app
kubectl delete -f npbackup.yaml
rm -f frontent.conf nginx.conf toolbox.conf toolbox-index.html frontent-index.html nginx-index.html npbackup.yaml 
kubectl delete namespace project-app
```
