
# LAB 1

```
prerequisite
kubectl create namespace fubar
kubectl create namespace tiger
kubectl create namespace kdp1003

```


## How to create ClusterIP Service?
### From Yaml file.
```
cat <<EOF>> clusterip1.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
EOF
```
```
kubectl apply -f clusterip1.yaml
```

## How to get more information about the service?
```
kubectl get all
```

```
kubectl describe service my-service
```
```
kubectl describe endpoints my-service
```
```
kubectl get service my-service
```
```
kubectl get endpoints my-service
```

## How to create service under namespace?

```
kubectl create deployment test1 --image=nginx --replicas=3 -n fubar
```
```
kubectl get pods -o wide -n fubar --show-labels
```
```
kubectl get all -n fubar
```
```
cat <<EOF>> clusterip-fubar.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: fubar
spec:
  type: ClusterIP
  selector:
    app: test1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
EOF
```


```
kubectl create -f clusterip-fubar.yaml
```

### How to check the service inside the Namespace?
```
kubectl get service -n fubar
```
```
kubectl get endpoints -n fubar
```
```
kubectl get pods -o wide -n fubar --show-labels
```
```
curl http://10.107.4.233
```
```
curl -LI http://10.107.4.233 -o /dev/null -w '%{http_code}\n' -s
```


### Impact on deleting the PODS from Deployment.
#### Execute the below command on one terminal
```
for i in {1..1000000} ; do  curl -LI http://10.107.4.233 -o /dev/null -w '%{http_code}\n' -s ; done | grep -v 200
```

```
kubectl get pods -o wide -n fubar
```
#### complete the POD names by pressing the tap button.
```
kubectl delete -n fubar pod/test1-
```


## How to create NodePort Service?


```
cat <<EOF>> my-nodeport-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
  namespace: fubar
spec:
  type: NodePort
  selector:
    app: test1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30000
EOF
```
```
kubectl apply -f my-nodeport-service.yaml
```

### How to get more information about the NodePort Service?
```
kubectl -n fubar get service/my-nodeport-service
```
```
kubectl -n fubar get endpoints/my-nodeport-service
```
```
kubectl -n fubar describe service/my-nodeport-service
```


```
ClusterIP=`kubectl -n fubar get service/my-nodeport-service | awk '{print $3}' | grep -v CLUSTER`; curl http://$ClusterIP
```

### Create a Deployment test1-deploy under kdp1003 namespace with image nginx and should have 3 replicas. Expose the individual pods via a NodePort on the node on which they are scheduled.
```
kubectl -n kdp1003 create deployment test1-deploy --image=nginx
```
```
kubectl -n kdp1003 expose deployment test1-deploy --name=test1-deploy-svc  --port=80 --target-port=80 --protocol=TCP --type=NodePort
```
```
kubectl -n kdp1003 get all
```



## How to create LoadBalancer?


```
cat <<EOF>> my-lb-service1.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-lb-service1
  namespace: tiger
spec:
  type: LoadBalancer
  selector:
    app: test1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30002
EOF
```
```
kubectl create -f my-lb-service1.yaml 
```
```
kubectl get service --namespace tiger 
```

### How to create Address polling for MetalLB?

```
cat <<EOF>> IPAddressPool.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.100-192.168.1.110
EOF
 
```







# Clear the lab 

```
kubectl delete -n core DaemonSets/test1-daemonset
kubectl delete namespaces core --grace-period=0 --force
kubectl delete -n  kube-system ds/fluentd-elasticsearch
```

