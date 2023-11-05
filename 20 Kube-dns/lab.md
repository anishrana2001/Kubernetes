1. How to find the DNS pod in Kubernetes cluster?
2. DNS Deployment configuration file.
3. How PODs and Services are resoling in Kubernetes cluster?
4. Resolving the pods and services from inside the pods.
5. Resolving the records of other namespace's pods.
6. How to login into the DNS POD?



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
```
curl http://172.16.68.38:8080/health ; echo
```
```
kubectl get pods/coredns-565d847f94-m4pjz -n kube-system -o=jsonpath="{range .items[*]}{.status.podIP
```
```
curl http://$(kubectl get pods/coredns-565d847f94-m4pjz -n kube-system -o=jsonpath="{range .items[*]}{.status.podIP}"):8080/health ; echo
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
```
dig @10.96.0.10 +short  POD_NAME
```
```
dig @10.96.0.10  -x  +short POD_IP
```
```
dig @10.96.0.10 +short POD-IP-with-hypen.core.pod.cluster.local
```
```
kubectl -n core get service
```
```
dig @10.96.0.10   +short -x 10.100.177.112
```
```
dig @10.96.0.10   +short front-end-svc.core.svc.cluster.local.
```


## 4. Resolving the pods and services from inside the pods.
```
kubectl -n core exec  -it front-end-c446cd7f4-hqxtn -- /bin/bash
```
```
apt-get update && apt-get install -y iputils-ping dnsutils
```
```
dig 172-16-133-166.core.pod.cluster.local +short
```
```
dig 172-16-14-71.core.pod.cluster.local +short
```
```
dig -x  172.16.14.71    +short
```
```
dig front-end-svc.core.svc.cluster.local +short
```
```
dig -x 10.100.177.112 +short
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
