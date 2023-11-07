# TOPIC
### 1. How to find the DNS pod in Kubernetes cluster?
### 2. DNS Deployment configuration file.
### 3. How PODs and Services are resoling in Kubernetes cluster?
### 4. Resolving the pods and services from inside the pods.
### 5. Resolving the records of other namespace's pods.
### 6. How to login into the DNS POD?

#
#

## 1. How to find the DNS pod in Kubernetes cluster?
```
kubectl -n kube-system get pods
```
```
kubectl get service -n kube-system
```
```
kubectl -n kube-system describe service/kube-dns
```
```
kubectl get all -n kube-system -o wide | grep core
```

## 2. DNS Deployment configuration file.
```
kubectl -n kube-system get configmap
```
```
kubectl -n kube-system get configmap coredns -oyaml
```
```
kubectl -n kube-system get po -o wide | grep coredns
```
curl http://kube-dns-POD-IP:8080/health ; echo
```
kubectl -n kube-system get pods -owide | grep core

```
OR
```
kubectl get pods/$(kubectl -n kube-system get pods | grep core | head -1 | awk '{print $1}') -n kube-system -o=jsonpath="{range .items[*]}{.status.podIP}" ; echo
```
```
curl http://$(kubectl get pods/$(kubectl -n kube-system get pods | grep core | head -1 | awk '{print $1}') -n kube-system -o=jsonpath="{range .items[*]}{.status.podIP}"):8080/health ; echo
```


## 3. How PODs and Services are resoling in Kubernetes cluster?
```
kubectl create namespace core
```
```
kubectl create deployment front-end --image=nginx --replicas=3 -n core
```
```
kubectl expose deployment front-end --name=front-end-svc --port=80 --target-port=80 --protocol=TCP --type=NodePort -n core
```
```
kubectl -n core get all -o wide
```
```
kubectl -n kube-system get po -o wide | grep coredns
```
```
kubectl get service -n kube-system 
```
### Let's try to resolve the Deployment's POD name.
```
dig @10.96.0.10 +short  front-end_POD_NAME
```
```
dig @10.96.0.10 +short -x  POD_IP
```
```
dig @10.96.0.10 +short POD-IP-with-hypen.core.pod.cluster.local
```
```
kubectl -n core get service
```
```
dig @10.96.0.10 +short -x IP_ADD_CLUSTER_IP_Service
```
```
dig @10.96.0.10 +short front-end-svc.core.svc.cluster.local.
```


## 4. Resolving the pods and services from inside the pods.

```
kubectl -n core get pods -o wide
```
```
kubectl -n core exec  -it $(kubectl -n core get pods | grep front | awk '{print $1}' | head -n 1) -- /bin/bash
```
```
apt-get update && apt-get install -y iputils-ping dnsutils
```
### Exit from the pod and execute below command again.

```
kubectl -n core get pods -o wide
```
```
kubectl -n core exec  -it $(kubectl -n core get pods | grep front | awk '{print $1}' | head -n 1) -- /bin/bash
```

```
dig +short POD1-IP.core.pod.cluster.local 
```
```
dig +short POD2-IP.core.pod.cluster.local 
```
```
dig +short -x POD2_IP    
```
```
dig +short front-end-svc.core.svc.cluster.local 
```
```
dig +short -x SERVICE_IP_CLUSTER 
```

## 5. Resolving the records of other namespace's pods.
```
kubectl run nginx --image=busybox -- /bin/sh -c 'sleep 20000'
```
```
kubectl exec -it pod/nginx -- /bin/sh
```
```
nslookup POD-IP.core.pod.cluster.local
```
```
cat   /etc/resolv.conf 
```
```
nslookup kubernetes
```
```
nslookup kubernetes.
```


## 6. How to login into the DNS POD?

```
kubectl -n kube-system get pods -o wide| grep core
```
```
kubectl get pods/coredns-565d847f94-m4pjz -n kube-system -o=jsonpath="{range .items[*]}{.spec.containers[].name}" ; echo
```
```
kubectl -n kube-system debug -it coredns-565d847f94-m4pjz --image=busybox:1.28 --target=coredns
```
```
ip a show eth0
```

## Clear the lab

```

kubectl -n core delete deployments.apps front-end
kubectl -n core delete service/front-end-svc
kubectl delete namespaces core
kubectl delete pods/nginx 
```
