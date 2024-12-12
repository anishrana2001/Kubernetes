# **How to increase the PVC size**
---
* Prerequicite 
1. GIT
2. Kubernetes Cluster
3. csi-driver-host path

---
### How to `Install` GIT
---
### First, we will install the `git-all` package and then `clone` the git repository. 
```
dnf install git-all
mkdir ; /data; cd /data ; git clone https://github.com/kubernetes-csi/csi-driver-host-path.git
cd csi-driver-host-path/
```
### In order to install the `csi-driver-host-path`, we need to set the variable and after that we need to apply some yaml files.
#### Change to the latest supported snapshotter release branch
```
SNAPSHOTTER_BRANCH=release-6.3
```
### Apply VolumeSnapshot CRDs
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_BRANCH}/client/config/crd/snapshot.storage.k8s.io_volumesnapshotclasses.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_BRANCH}/client/config/crd/snapshot.storage.k8s.io_volumesnapshotcontents.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_BRANCH}/client/config/crd/snapshot.storage.k8s.io_volumesnapshots.yaml
```

### Change to the latest supported snapshotter version
```
SNAPSHOTTER_VERSION=v6.3.3
```
### Create snapshot controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/deploy/kubernetes/snapshot-controller/rbac-snapshot-controller.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/deploy/kubernetes/snapshot-controller/setup-snapshot-controller.yaml
```

### Check if any pods are running the snapshot-controller image:
```
kubectl get pods --all-namespaces -o=jsonpath='{range .items[*]}{"\n"}{range .spec.containers[*]}{.image}{", "}{end}{end}' | grep snapshot-controller
```


### Deploy hostpath driver
```
deploy/kubernetes-latest/deploy.sh
```
---
### Post Check! : 2 Pods must be in running state.
```
kubectl get pods | grep csi
```


---
### Let's create a StorageClass, PVC and pods. 
---
```
for i in ./examples/csi-storageclass.yaml ./examples/csi-pvc.yaml ./examples/csi-app.yaml; do kubectl apply -f $i; done
```



### **Post checks**
```
kubectl get sc,pv,pvc,pods
```

```
kubectl get pods my-csi-app
```

```
kubectl get pvc
```
### Now, our PVC is created and one pod is bound with this PVC.
### Its time to increase the PVC size.
```
kubectl edit pvc/csi-pvc
```
### We have edited the PVC, however it is still showing old value. The reason behind is that POD is still using this PVC. To make it effective, we have restart the pod.

#### _First, take the `backup` of this pod._
```
kubectl get pod my-csi-app -o  yaml > pod.yaml
```
### We can safely delete this pod.
```
kubectl delete pod my-csi-app 
```
### Create the pod again.
```
kubectl apply -f pod.yaml 
```
### Finally, check the PVC size, it should be increased. 
```
kubectl get pvc
```
```
kubectl describe pvc/csi-pvc 
```
