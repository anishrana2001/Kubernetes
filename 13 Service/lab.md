
# LAB 1

## prerequisite
```
mkdir /root/test-service-dir1
cd /root/test-service-dir1
kubectl create namespace fubar
kubectl create namespace tiger
kubectl create namespace kdp1003

```


## 1.  How to create ClusterIP Service?
### 1.1. Under Default Namespace.
#### 1.1.1 Yaml file.
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

#### 1.1.2 Create the ClusterIP from yaml file.
```
kubectl apply -f clusterip1.yaml
```

#### 1.1.3 How to extract the details of service?
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
#### 1.1.4 Now, create a deployment named nginx and see the result again.
```
kubectl create deployment nginx --image=nginx
```
```
kubectl get endpoints my-service
```


### 1.2 Under "fubar" Namespace.

#### 1.2.1 Create the deployment "test1" first with image "nginx" and replicas =3 under fubar namespace.
```
kubectl create deployment test1 --image=nginx --replicas=3 -n fubar
```
#### 1.2.2 How to check the running pods under "fubar" namespace and also showing the labels.
```
kubectl get pods -o wide -n fubar --show-labels
```
#### 1.2.3 List all objects under namespace "fubar"
```
kubectl get all -n fubar
```

#### 1.2.4 Yaml file for ClusterIP service with namespace.
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

#### 1.2.5 Create the ClusterIP service under fubar Namespace.
```
kubectl create -f clusterip-fubar.yaml
```

#### 1.2.6 How to check the service inside the Namespace?
```
kubectl get service -n fubar
```
#### 1.2.7 How to list all endpoints?
```
kubectl get endpoints -n fubar
```
```
kubectl get pods -o wide -n fubar --show-labels
```

#### 1.2.8 Access the pods ip with the help of CURL cummand.
```
curl http://ClusterIP
```
```
var_clusterIP=`kubectl get service/my-service -n fubar | awk '{print $3}' | grep -v CLUSTER` ; curl http://$var_clusterIP
```
#### 1.2.9 How to check the HTTP return code of the URL?
```
curl -LI http://$var_clusterIP -o /dev/null -w '%{http_code}\n' -s
```


#### 1.2.10 Impact on deleting the PODS from Deployment. Open 2 terminals.
##### 1.2.10.1 Execute the below command on 1st terminal
```
for i in {1..1000000} ; do  curl -LI http://$var_clusterIP -o /dev/null -w '%{http_code}\n' -s ; done | grep -v 200
```

##### 1.2.10.2  Execute the below command on 2nd Terminal.
```
kubectl get pods -o wide -n fubar
```
#####  1.2.10.3 complete the POD names by pressing the tap button on 2nd Terminal.
```
kubectl delete -n fubar pod/test1-
```


### 2 How to create NodePort Service?

### 2.1 From Yamml file.
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

###  2.2 Create the NodePort service from "apply" sub-command. 
```
kubectl apply -f my-nodeport-service.yaml
```

###  2.3 How to get more information about the NodePort Service?
```
kubectl -n fubar get service/my-nodeport-service
```
```
kubectl -n fubar get endpoints/my-nodeport-service
```
```
kubectl -n fubar describe service/my-nodeport-service
```
###  2.4 Access the POD's web portal from ClusterIP.
```
ClusterIP=`kubectl -n fubar get service/my-nodeport-service | awk '{print $3}' | grep -v CLUSTER`; curl http://$ClusterIP
```
### 2.4.1 We can also access the POD web page from our Node_IP:port_number

![image](https://user-images.githubusercontent.com/93471182/226165066-a6ecadcb-a88c-4c67-9e8a-493ff323d0f2.png)


###  2.5 NodePort service from Command line
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



## 3 How to create LoadBalancer?

### 3.1 Yaml file for LoadBalancer.
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

### 3.2 Create the LoadBalancer service.
```
kubectl create -f my-lb-service1.yaml 
```

### 3.3 Explore the service.
```
kubectl get service --namespace tiger 
```

### 3.4 Create deployment so that we can access the pods.

```
kubectl create deployment test1 --image=nginx --replicas=3 -n tiger
```

### 3.5 How to create External LoadBalancer "MetalLB"?

### 3.5.1 Apply the Yaml file.
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.9/config/manifests/metallb-native.yaml
```
### 3.5.2 Yaml file for L2 layer configuration for MetalLB.
```
cat <<EOF>>  l2advertise.yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
EOF
```
### 3.5.3 Create the L2 advertise.
```
kubectl create -f l2advertise.yaml
```

### 3.5.4 How to create Address polling for MetalLB, so that we can get the EXTERNAL IP address?

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
### 3.5.5 Create the IPAddressPool.
```
kubectl create -f IPAddressPool.yaml 
```

### 3.5.6 Now, you will observe the External IP 
```
kubectl -n tiger get service/my-lb-service1 
```
### 3.5.7 You can able to access the External IP address.
```
curl http://LoadBalancer_IP
```
### 3.5.7 You can able to access the External IP address.
```
var1_loadbalacer=`kubectl -n tiger get service/my-lb-service1 | awk '{print $4}' | grep -v EXTERNAL` ; echo $var1_loadbalacer
curl http://$var1_loadbalacer
```

## 4 Type ExternalName 

### 4.1 Yaml file for ExternalName
```
cat <<EOF>> external.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service-external
spec:
  type: ExternalName
  externalName: my-service.default.svc.cluster.local
EOF
```

### 4.2 Creating ExternalName name from Yaml file.
```
kubectl create -f external.yaml 
```

### 4.3 List the services and CNAME Record is created.
```
kubectl get service
```

```
[root@master1 ~]# kubectl get service
NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP                              PORT(S)        AGE
frontend-srv            LoadBalancer   10.97.40.240    192.168.1.100                            80:31675/TCP   163m
hello-srv               ClusterIP      10.102.52.221   <none>                                   80/TCP         163m
kubernetes              ClusterIP      10.96.0.1       <none>                                   443/TCP        101d
my-service-external     ExternalName   <none>          frontend-srv.default.svc.cluster.local   <none>         7s
```

### 4.4  frontend-srv  will be resolve if we do nsloookup my-service-external
```
kubectl run --rm -it test1 --image=gcr.io/google-samples/hello-go-gke:1.0  -- nslookup my-service-external.default.svc.cluster.local
```

```
[root@master1 test-service-dir1]# kubectl run --rm -it test1 --image=gcr.io/google-samples/hello-go-gke:1.0  -- nslookup my-service-external.default.svc.cluster.local
Address 1: ::1 localhost
Address 2: 127.0.0.1 localhost

Name:      my-service-external.default.svc.cluster.local
Address 1: 10.103.138.72 my-service.default.svc.cluster.local
pod "test1" deleted
```


# Clear the lab 

```
kubectl delete deployment.apps/nginx --force --timeout=0
kubectl delete deployment.apps/test1 -n fubar --force --timeout=0
kubectl delete deployment.apps/test1-deploy -n kdp1003 --force --timeout=0
kubectl delete deployment.apps/test1 -n tiger --force --timeout=0
kubectl delete namespaces  kdp1003 --timeout=0 --force 
kubectl delete namespaces fubar --timeout=0 --force 
kubectl delete namespaces metallb-system --timeout=0 --force 
kubectl delete namespaces --timeout=0 --force tiger
kubectl delete service/my-service --force --timeout=0
kubectl delete service/my-service-external created --force --timeout=0

rm -f /root/test-service-dir1/clusterip1.yaml
rm -f /root/test-service-dir1/clusterip-fubar.yaml
rm -f /root/test-service-dir1/my-nodeport-service.yaml
rm -f /root/test-service-dir1/my-lb-service1.yaml
rm -f /root/test-service-dir1/IPAddressPool.yaml
rm -f /root/test-service-dir1/l2advertise.yaml
rm -f /root/test-service-dir1/external.yaml
rmdir /root/test-service-dir1/
```

