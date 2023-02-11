### Blockquotes

> Blockquotes

Paragraphs and Line Breaks
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
loadbalancer   2/2     2            2           7s
                    
> "Blockquotes Blockquotes"
```
kubectl create namespace kdp2001
kubectl create namespace kdpd002021
kubectl create namespace project-tiger
```
``
namespace/kdp2001 created
namespace/kdpd002021 created
namespace/project-tiger created
``

## Question 1 : 
### Create a deployment loadbalancer with image nginx:1.14.2 and it should have 2 replicas.
>[root@master1 ~]# kubectl create deployment loadbalancer --image=nginx:1.14.2 --replicas=2

>deployment.apps/loadbalancer created


[root@master1 ~]# kubectl get deployments.apps loadbalancer

`
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
loadbalancer   2/2     2            2           7s
`

## Question 2: 
### Scale the deployment loadbalancer to 6 pods.

[root@master1 ~]# kubectl scale deployment loadbalancer --replicas=6
``
deployment.apps/loadbalancer scaled
``
[root@master1 ~]# kubectl get deployments.apps loadbalancer
``
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
loadbalancer   6/6     6            6           24s
``
## Question 3: 
### Create a Deployment named deploy-important with image nginx:1.17.6-alpine. It should contain 2 containers, the first named nginx with image nginx:1.17.6-alpine and the second one named redis with image redis.

[root@master1 ~]# kubectl create deployment deploy-important  --image=nginx:1.17.6-alpine --dry-run=client -o yaml > deploy-important.yaml

[root@master1 ~]# vim deploy-important.yaml 


[root@master1 ~]# cat deploy-important.yaml
```yamlscript
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: deploy-important
  name: deploy-important
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deploy-important
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deploy-important
    spec:
      containers:
      - image: nginx:1.17.6-alpine
        name: nginx
      - image: redis
        name: redis
```


[root@master1 ~]# kubectl create -f deploy-important.yaml
``
deployment.apps/deploy-important created
``

[root@master1 ~]# kubectl get deployments.apps deploy-important 
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
deploy-important   1/1     1            1           19s

[root@master1 ~]# kubectl get pod/deploy-important-7fb555749-9mhjd 
NAME                               READY   STATUS    RESTARTS   AGE
deploy-important-7fb555749-9mhjd   2/2     Running   0          34s

[root@master1 ~]# kubectl describe pod/deploy-important-7fb555749-9mhjd 
Name:             deploy-important-7fb555749-9mhjd
Namespace:        default
Priority:         0
Service Account:  default
Node:             workernode1.example.com/192.168.1.32
Start Time:       Thu, 09 Feb 2023 20:17:50 +0530
Labels:           app=deploy-important
                  pod-template-hash=7fb555749
Annotations:      cni.projectcalico.org/containerID: 310fd81ee12aba8d1bf6fedf1cf0de28559aa8324e0f42843ba43110ec22a012
                  cni.projectcalico.org/podIP: 172.16.133.148/32
                  cni.projectcalico.org/podIPs: 172.16.133.148/32
Status:           Running
IP:               172.16.133.148
IPs:
  IP:           172.16.133.148
Controlled By:  ReplicaSet/deploy-important-7fb555749
Containers:
  nginx:
    Container ID:   containerd://fd40dd29f6c312ec07e95cf03fe080ed956ed3e20f30cc433b0c370ba01fee10
    Image:          nginx:1.17.6-alpine
    Image ID:       docker.io/library/nginx@sha256:0e61b143db3110f3b8ae29a67f107d5536b71a7c1f10afb14d4228711fc65a13
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 09 Feb 2023 20:17:59 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bhljn (ro)
  redis:
    Container ID:   containerd://4ed7efd9e78e440d42327862ca52ab527f36e82636915cfa793e85b1cea0b2e6
    Image:          redis
    Image ID:       docker.io/library/redis@sha256:6a59f1cbb8d28ac484176d52c473494859a512ddba3ea62a547258cf16c9b3ae
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 09 Feb 2023 20:18:06 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bhljn (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-bhljn:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  43s   default-scheduler  Successfully assigned default/deploy-important-7fb555749-9mhjd to workernode1.example.com
  Normal  Pulling    43s   kubelet            Pulling image "nginx:1.17.6-alpine"
  Normal  Pulled     35s   kubelet            Successfully pulled image "nginx:1.17.6-alpine" in 7.927245432s
  Normal  Created    35s   kubelet            Created container nginx
  Normal  Started    35s   kubelet            Started container nginx
  Normal  Pulling    35s   kubelet            Pulling image "redis"
  Normal  Pulled     28s   kubelet            Successfully pulled image "redis" in 6.909027261s
  Normal  Created    28s   kubelet            Created container redis
  Normal  Started    28s   kubelet            Started container redis



[root@master1 ~]# kubectl describe pod/deploy-important-7fb555749-9mhjd | egrep "nginx|redis"
  nginx:
    Image:          nginx:1.17.6-alpine
    Image ID:       docker.io/library/nginx@sha256:0e61b143db3110f3b8ae29a67f107d5536b71a7c1f10afb14d4228711fc65a13
  redis:
    Image:          redis
    Image ID:       docker.io/library/redis@sha256:6a59f1cbb8d28ac484176d52c473494859a512ddba3ea62a547258cf16c9b3ae
  Normal  Pulling    107s  kubelet            Pulling image "nginx:1.17.6-alpine"
  Normal  Pulled     99s   kubelet            Successfully pulled image "nginx:1.17.6-alpine" in 7.927245432s
  Normal  Created    99s   kubelet            Created container nginx
  Normal  Started    99s   kubelet            Started container nginx
  Normal  Pulling    99s   kubelet            Pulling image "redis"
  Normal  Pulled     92s   kubelet            Successfully pulled image "redis" in 6.909027261s
  Normal  Created    92s   kubelet            Created container redis
  Normal  Started    92s   kubelet            Started container redis
[root@master1 ~]# 


Press the double tap, you will see the container names.

[root@master1 ~]# kubectl logs pods/deploy-important-7fb555749-9mhjd 
nginx  redis  

## Question 4: 

### Create a new deployment for running nginx with the following parameters. 
### Run the deploymnet in the kdp2001 namespace. The namespace has alrady been created.
### Name the deployment nginx and configure with 5 replicas
### Configure the pod with a container image of ifccnf/nginx:1.13.7-alpine
### Set an environment variable of NGINX_Port=8080 and also expose that port for the container above.


[root@master1 ~]kubectl -n kdp2001  create deployment nginx --image=nginx:1.13.7-alpine --replicas=5 --dry-run=client -oyaml > nginx.yaml
[root@master1 ~]# vim nginx.yaml 
[root@master1 ~]# cat nginx.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
  namespace: kdp2001
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.13.7-alpine
        name: nginx
        env:
        - name: NGINX_Port
          value: "8080"
        resources: {}
status: {}
```
[root@master1 ~]# kubectl create -f nginx.yaml 
deployment.apps/nginx created

[root@master1 ~]# kubectl get deployments.apps -n kdp2001 nginx 
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   5/5     5            5           63s


[root@master1 ~]# kubectl  exec -n kdp2001 pods/nginx-747b75fccc-29f8r -- env | grep NGINX_Port
NGINX_Port=8080


## Question 5

### As a kubernetes application developer you will often find yourself needing to update a runing application. Please complete the following:

### Update the web deployment in the kdpd002021 namespace with a maxSurge of 10% and a maxUnavailable of 5%
### Perform a rolling update of the web deployment changing the nginx image version to 1.14.2
### Rollback the web deployment to the previous version and it should record.


[root@master1 ~]# kubectl -n kdpd002021 describe deployments.apps web 
Name:                   web
Namespace:              kdpd002021
CreationTimestamp:      Thu, 09 Feb 2023 23:00:15 +0530
Labels:                 app=web
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=web
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=web
  Containers:
   nginx:
    Image:        nginx:1.13
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   web-76b8d9869b (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  25s   deployment-controller  Scaled up replica set web-76b8d9869b to 1

[root@master1 ~]# kubectl -n kdpd002021 edit deployments.apps web 
deployment.apps/web edited

[root@master1 ~]# kubectl -n kdpd002021 describe deployments.apps web | grep RollingUpdateStrategy
RollingUpdateStrategy:  5% max unavailable, 10% max surge

[root@master1 ~]# kubectl -n kdpd002021 get replicasets.apps 
NAME             DESIRED   CURRENT   READY   AGE
web-76b8d9869b   1         1         1       97s

[root@master1 ~]# kubectl -n kdpd002021 set image deployment web nginx=nginx:1.14.2 --record
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/web image updated

[root@master1 ~]# kubectl rollout history -n kdpd002021 deployment web 
deployment.apps/web 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment web nginx=nginx:1.14.2 --namespace=kdpd002021 --record=true

[root@master1 ~]# kubectl -n kdpd002021 get replicasets.apps 
NAME             DESIRED   CURRENT   READY   AGE
web-668c4846c5   1         1         1       3m27s
web-76b8d9869b   0         0         0       5m10s


[root@master1 ~]# kubectl -n kdpd002021 rollout undo deployment web
deployment.apps/web rolled back

[root@master1 ~]# kubectl rollout history -n kdpd002021 deployment web 
deployment.apps/web 
REVISION  CHANGE-CAUSE
2         kubectl set image deployment web nginx=nginx:1.14.2 --namespace=kdpd002021 --record=true
3         <none>

[root@master1 ~]# kubectl -n kdpd002021 get replicasets.apps 
NAME             DESIRED   CURRENT   READY   AGE
web-668c4846c5   0         0         0       2m54s
web-76b8d9869b   1         1         1       4m37s


## Question 6

### Use Namespace project-tiger for the following. Create a Deployment named deploy-important with label id=very-important (the Pods should also have this label) and 3 replicas. It should contain two containers, the first named container1 with image nginx:1.17.6-alpine and the second one named container2 with image kubernetes/pause.
### Use the Node selector: disktype=ssd


[root@master1 ~]# kubectl get nodes --show-labels 
NAME                      STATUS   ROLES           AGE   VERSION   LABELS
master1.example.com       Ready    control-plane   64d   v1.25.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master1.example.com,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
workernode1.example.com   Ready    <none>          64d   v1.25.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=workernode1.example.com,kubernetes.io/os=linux
workernode2.example.com   Ready    <none>          64d   v1.25.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=workernode2.example.com,kubernetes.io/os=linux


### Below is the command to add the label on nodes.
[root@master1 ~]# kubectl label nodes workernode1.example.com disktype=ssd
node/workernode1.example.com labeled

[root@master1 ~]# kubectl label nodes workernode2.example.com disktype=ssd
node/workernode2.example.com labeled

### Now, you can see the labels = disktype=ssd
[root@master1 ~]# kubectl get nodes --show-labels 
NAME                      STATUS   ROLES           AGE   VERSION   LABELS
master1.example.com       Ready    control-plane   64d   v1.25.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master1.example.com,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
workernode1.example.com   Ready    <none>          64d   v1.25.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=workernode1.example.com,kubernetes.io/os=linux
workernode2.example.com   Ready    <none>          64d   v1.25.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=workernode2.example.com,kubernetes.io/os=linux


[root@master1 ~]# kubectl -n project-tiger create deployment deploy-important --image=nginx:1.17.6-alpine --replicas=3 --dry-run=client -oyaml > deploy-important.yaml

[root@master1 ~]# vim deploy-important.yaml 

[root@master1 ~]# cat deploy-important.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    id: very-important               # change the label
  name: deploy-important
  namespace: project-tiger
spec:
  replicas: 3
  selector:
    matchLabels:
      id: very-important              # change the label 
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: very-important             # change the label
    spec:
      containers:
      - image: nginx:1.17.6-alpine  
        name: container1               # update the container name
      - image: kubernetes/pause        # add the image name
        name: container2               # add the container name
      nodeSelector:                # added line
        disktype: ssd              # added line
```
[root@master1 ~]# kubectl create -f deploy-important.yaml 
deployment.apps/deploy-important created

[root@master1 ~]# kubectl -n project-tiger get deployments.apps -o wide --show-labels 
NAME               READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS              IMAGES                                 SELECTOR            LABELS
deploy-important   3/3     3            3           88s   container1,container2   nginx:1.17.6-alpine,kubernetes/pause   id=very-important   id=very-important

[root@master1 ~]# kubectl -n project-tiger get pods --show-labels -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP               NODE                      NOMINATED NODE   READINESS GATES   LABELS
deploy-important-7995fcd99d-56ffq   2/2     Running   0          62s   172.16.133.159   workernode1.example.com   <none>           <none>            id=very-important,pod-template-hash=7995fcd99d
deploy-important-7995fcd99d-s7g2n   2/2     Running   0          62s   172.16.14.100    workernode2.example.com   <none>           <none>            id=very-important,pod-template-hash=7995fcd99d
deploy-important-7995fcd99d-x75z9   2/2     Running   0          62s   172.16.14.95     workernode2.example.com   <none>           <none>            id=very-important,pod-template-hash=7995fcd99d


## Question 7

### A deployment is failing on the cluster. Locate the deployment, and fix the problem.


[root@master1 ~]# kubectl get deployments.apps -A
NAMESPACE       NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
default         deploy-important          1/1     1            1           3h16m
default         loadbalancer              6/6     6            6           3h19m
default         testing-deploy            0/2     2            0           19s       >>>>> This deployment has the problem
kdp2001         nginx                     5/5     5            5           167m
kdpd002021      web                       1/1     1            1           34m
kube-system     calico-kube-controllers   1/1     1            1           64d
kube-system     coredns                   2/2     2            2           64d
kube-system     metrics-server            1/1     1            1           6d2h
project-tiger   deploy-important          3/3     3            3           3m41s


[root@master1 ~]# kubectl describe deployments.apps testing-deploy 
Name:                   testing-deploy
Namespace:              default
CreationTimestamp:      Thu, 09 Feb 2023 23:34:00 +0530
Labels:                 app=testing-deploy
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=testing-deploy
Replicas:               2 desired | 2 updated | 2 total | 0 available | 2 unavailable   --> Pods are not running.
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=testing-deploy
  Containers:
   nginx:
    Image:        nginx:1.14.7
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   testing-deploy-6b9b844c9b (2/2 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m15s  deployment-controller  Scaled up replica set testing-deploy-6b9b844c9b to 2


[root@master1 ~]# kubectl get pods --show-labels | grep testing
testing-deploy-6b9b844c9b-7mhsm    0/1     ImagePullBackOff   0          5m25s   app=testing-deploy,pod-template-hash=6b9b844c9b
testing-deploy-6b9b844c9b-l78pk    0/1     ImagePullBackOff   0          5m25s   app=testing-deploy,pod-template-hash=6b9b844c9b


[root@master1 ~]# kubectl describe pod/testing-deploy-6b9b844c9b-7mhsm
Name:             testing-deploy-6b9b844c9b-7mhsm
Namespace:        default
Priority:         0
Service Account:  default
Node:             workernode1.example.com/192.168.1.32
Start Time:       Thu, 09 Feb 2023 23:34:00 +0530
Labels:           app=testing-deploy
                  pod-template-hash=6b9b844c9b
Annotations:      cni.projectcalico.org/containerID: cfd832c3a809659dc05ccba55ac21058f9bc171a2e1edb3c230f2075228f26b9
                  cni.projectcalico.org/podIP: 172.16.133.158/32
                  cni.projectcalico.org/podIPs: 172.16.133.158/32
Status:           Pending
IP:               172.16.133.158
IPs:
  IP:           172.16.133.158
Controlled By:  ReplicaSet/testing-deploy-6b9b844c9b
Containers:
  nginx:
    Container ID:   
    Image:          nginx:1.14.7
    Image ID:       
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-n6f7q (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-n6f7q:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
  Normal   Scheduled  5m54s                  default-scheduler  Successfully assigned default/testing-deploy-6b9b844c9b-7mhsm to workernode1.example.com
  Normal   Pulling    4m21s (x4 over 5m54s)  kubelet            Pulling image "nginx:1.14.7"
  Warning  Failed     4m19s (x4 over 5m51s)  kubelet            Failed to pull image "nginx:1.14.7": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/nginx:1.14.7": failed to resolve reference "docker.io/library/nginx:1.14.7": docker.io/library/nginx:1.14.7: not found
  Warning  Failed     4m19s (x4 over 5m51s)  kubelet            Error: ErrImagePull
  Warning  Failed     4m5s (x6 over 5m50s)   kubelet            Error: ImagePullBackOff
  Normal   BackOff    47s (x20 over 5m50s)   kubelet            Back-off pulling image "nginx:1.14.7"


Edit the image name from "nginx:1.14.7" to "nginx:1.14.2"
[root@master1 ~]# kubectl edit deployments.apps testing-deploy   
deployment.apps/testing-deploy edited

[root@master1 ~]# kubectl get deployments.apps testing-deploy 
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
testing-deploy   2/2     2            2           7m23s
