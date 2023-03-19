
# LAB 1

## prerequisite
```
mkdir test-service-dir1
cd test-service-dir1
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

### How to extract the details of service?
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
### Now, create a deployment named nginx and see the result again.
```
kubectl create deployment nginx --image=nginx
```
```
kubectl get endpoints my-service
```


### From yaml file under namespace.

#### Create the deployment "test1" first with image "nginx" and replicas =3 under fubar namespace.
```
kubectl create deployment test1 --image=nginx --replicas=3 -n fubar
```
#### How to check the running pods under "fubar" namespace and also showing the labels.
```
kubectl get pods -o wide -n fubar --show-labels
```
#### List all objects under namespace "fubar"
```
kubectl get all -n fubar
```

#### Yaml file for ClusterIP service with namespace.
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
#### How to list all endpoints?
```
kubectl get endpoints -n fubar
```
```
kubectl get pods -o wide -n fubar --show-labels
```

#### Access the pods ip with the help of CURL cummand.
```
curl http://ClusterIP
```
```
var_clusterIP=`kubectl get service/my-service -n fubar | awk '{print $3}' | grep -v CLUSTER` ; curl http://$var_clusterIP
```
#### How to check the HTTP return code of the URL?
```
curl -LI http://$var_clusterIP -o /dev/null -w '%{http_code}\n' -s
```


### Impact on deleting the PODS from Deployment. Open 2 terminals.
#### Execute the below command on 1st terminal
```
for i in {1..1000000} ; do  curl -LI http://$var_clusterIP -o /dev/null -w '%{http_code}\n' -s ; done | grep -v 200
```

#### Execute the below command on 2nd Terminal.
```
kubectl get pods -o wide -n fubar
```
#### complete the POD names by pressing the tap button on 2nd Terminal.
```
kubectl delete -n fubar pod/test1-
```


## How to create NodePort Service?

### From Yamml file
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

### Create the NodePort service from "apply" sub-command. 
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

### NodePort service from Command line
#### Create a Deployment test1-deploy under kdp1003 namespace with image nginx and should have 3 replicas. Expose the individual pods via a NodePort on the node on which they are scheduled.
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

### Create deployment so that we can access the pods.

```
kubectl create deployment test1 --image=nginx --replicas=3 -n tiger
```

## How to create External LoadBalancer "MetalLB"?
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.9/config/manifests/metallb-native.yaml
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

```
kubectl create -f IPAddressPool.yaml 
```

#### Now you will observe the External IP 
```
kubectl -n tiger get service/my-lb-service1 
```

```
curl http://LoadBalancer_IP
```
```
var1_loadbalacer=`kubectl -n tiger get service/my-lb-service1 | awk '{print $4}' | grep -v EXTERNAL` ; echo $var1_loadbalacer
curl http://$var1_loadbalacer
```
## Type ExternalName 

### Yaml file for ExternalName
```
cat <<EOF>> external.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ExternalName
  externalName: frontend-srv.default.svc.cluster.local
EOF
```

### Creating ExternalName name from Yaml file.
```
kubectl create -f external.yaml 
```

###  List the services.
```
kubectl get service
```

```
[root@master1 ~]# kubectl get service
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP                              PORT(S)        AGE
frontend-srv   LoadBalancer   10.97.40.240    192.168.1.100                            80:31675/TCP   163m
hello-srv      ClusterIP      10.102.52.221   <none>                                   80/TCP         163m
kubernetes     ClusterIP      10.96.0.1       <none>                                   443/TCP        101d
my-service     ExternalName   <none>          frontend-srv.default.svc.cluster.local   <none>         7s
```

###  CNAME Record is created.
```
kubectl exec -it backend-deployment-664dcf7b6f-kgskw -- nslookup my-service.default.svc.cluster.local
```

```
[root@master1 ~]# kubectl exec -it backend-deployment-664dcf7b6f-kgskw -- nslookup my-service.default.svc.cluster.local
Server:    (null)
Address 1: ::1 localhost
Address 2: 127.0.0.1 localhost

Name:      my-service.default.svc.cluster.local
Address 1: 10.97.40.240 frontend-srv.default.svc.cluster.local
```

# Clear the lab 

```
kubectl delete namespaces --timeout=0 --force  kdp1003
kubectl delete namespaces --timeout=0 --force  fubar
kubectl delete namespaces --timeout=0 --force  development
kubectl delete namespaces --timeout=0 --force metallb-system
kubectl delete namespaces --timeout=0 --force kdp1003
kubectl delete namespaces --timeout=0 --force tiger
kubectl delete service/my-service --force --timeout=0
kubectl delete deployment.apps/nginx --force --timeout=0
kubectl delete service/my-service created --force --timeout=0

rm -f test-service-dir1/clusterip1.yaml
rm -f test-service-dir1/clusterip-fubar.yaml
rm -f test-service-dir1/my-nodeport-service.yaml
rm -f test-service-dir1/my-lb-service1.yaml
rm -f test-service-dir1/IPAddressPool.yaml
rm -f test-service-dir1/external.yaml
```

