## Lab for Auto scaling.
### We will bring the NGINX server web page up and running. After that we will send the traffic to one POD so that it will increase the CPU resources. Finally, HPA will create a new pod.

## _prerequisite.
---
- 01 Configure metric server.
- 02 StatefulSet Pod web page up and running.
- 03 Configure HPA.
---

### 01 How to configure the metrics server?

## Before install the metrics server, let's check if it is already installed or not?
```
kubectl top pod -A
```

### Apply the Metrics server yaml file from Github.
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Check the deployment from all namespace.
```
kubectl get deployments.apps -A
```

### We can also verify the newly created metrics pod.
```
kubectl -n kube-system get pods | grep metrics
```

### If you wish, you can also explore this pod with "describe" command.
```
kubectl describe pod metrics-server-75bf97fcc9-hghhm  -n kube-system 
```

### Check the logs.
```
kubectl -n kube-system logs pods/metrics-server-75bf97fcc9-hghhm
```
```
kubectl -n kube-system edit deployments.apps metrics-server
```
---------------------------------------
      containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server/metrics-server:v0.3.7
        imagePullPolicy: IfNotPresent
        args:
          - --kubelet-insecure-tls
---------------------------------------
```
kubectl -n kube-system logs pods/metrics-server-75bf97fcc9-hghhm  (new pod)
```
```
kubectl top pod -A
```

### 02 Modify the NGINX configuration file so that we can see the webpage. 
### Create a file.
```
echo "This is my first website, running on nginx-sts-0 pod" > index.html
```

### Now, copy this file into the pod. 
```
kubectl cp index.html nginx-sts-0://usr/share/nginx/html/
```
### Check the file, if it is copied correctly.
```
kubectl exec -it nginx-sts-0 -- cat /usr/share/nginx/html/index.html
```

### Check the pod IP address.

```
kubectl get pod nginx-sts-0 --template '{{.status.podIP}}' ; echo
```

### Open the website. 
```
kubectl exec -it nginx-sts-0 -- curl http://POD_IP
```

### 03 Create a HorizontalPodAutoscaler (HPA).
```
cat <<EOF | kubectl apply -f -
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-nginx-sts
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: StatefulSet
    name: nginx-sts
  minReplicas: 2
  maxReplicas: 3
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 20
EOF
```




### Open another terminal and execute this command. It will generate the traffic and increase the CPU.



```
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://nginx-sts-0_POD_IP; done"
```

### Go back to first terminal and then execute the below command.
```
kubectl top pods
```
