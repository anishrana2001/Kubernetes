## HeadLess service LAB

---
### _NOTE: In the Previous lab, we have already created the headless servie._
### _In this lab, we will explore the headless service._
---
```
https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/
```

### How to check the pod name entries on DNS server?

### Create a dnsutils pod.
```
kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
```


### Verify the pod.
```
kubectl get pods dnsutils
```

### Check the services in the Kubernetes cluster.
```
kubectl get service
```

```
kubectl exec -i -t dnsutils -- nslookup kubernetes.default
```

### Identify the resolve.conf file on this pod.


```
kubectl exec -ti dnsutils -- cat /etc/resolv.conf
```
### Check the "nginx-sts-0" Pod IP address and the perform the reverse lookup.

```
kubectl get pods -o wide
```
```
kubectl exec -i -t dnsutils -- nslookup  IP_Address_StateFulSet_POD
``` 
### Copy the name of pod and then perform the nslookup.
```
kubectl exec -i -t dnsutils -- nslookup nginx-sts-0.svc-nginx-sts.default.svc.cluster.local.
```


### If I create a new deployment, and check the pod DNS name, it will not resolve. Let's take a look into this.

```
kubectl create deployment test-pod --image=nginx 
``` 

```
kubectl get pods -o wide
```

```
kubectl exec -i -t dnsutils -- nslookup  POD_IP_test-pod-*
``` 

```
kubectl exec -i -t dnsutils -- nslookup test-pod-6ddbc6dd5f-9pbr7.default.svc.cluster.local.
```


### We can also perform the nslookup for Headless service. You will observe 2 IPs of StatefulSet PODs.

```
kubectl exec -i -t dnsutils -- nslookup  svc-nginx-sts.default.svc.cluster.local.
```

### From this lab, we get the know that due to statefulset, our PODs DNS entry will be created and headless service will resolve the statefulset POD's IP address.

### Clear the LAB.

### Now, delete the dnsutils pod and deployment.
```
kubectl delete pod dnsutils 
kubectl delete deployments.apps test-pod 
```
