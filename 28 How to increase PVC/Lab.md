# **NFS Server Configuration**

## _Note_
---
- Lab Setup : (4 VMs)
-  1 master (192.168.1.31)
-  2 workernodes, (192.168.1.32/33)
-  1 NFS server (192.168.1.34)

---
### Login into NFS server, and check the connectivity with master server.
```
ping 192.168.1.31
```
### Create a new directory for the PV (Persistent Volume)
```
mkdir -p /var/nfs/database/pg-0
mkdir -p /var/nfs/database/pg-1
mkdir -p /var/nfs/database/pg-2
```

### Install the NFS packages, if not already installed.
```
dnf -y install nfs-utils
```
```
vi /etc/idmapd.conf 
```
```
Domain = example.com
```
```
vi /etc/exports
```
```
/var/nfs/database/pg-0   192.168.1.0/24(rw,no_root_squash)
/var/nfs/database/pg-1   192.168.1.0/24(rw,no_root_squash)
/var/nfs/database/pg-2   192.168.1.0/24(rw,no_root_squash)
```
```
systemctl enable --now rpcbind nfs-server
```

```
iptables -F
```
```
export -a
```
```
systemctl restart nfs-server
firewall-cmd --add-service=nfs 
firewall-cmd --runtime-to-permanent 
showmount -e localhost
```


# **StatefulSet**
### First, we need to install the **nfs-utils** package on all the nodes.
```
yum install nfs-utils -y 
```

## _Note_

---
- We will create the StatefulSet with 3 Pods replicas. 
- We will create 3 PVs
- We will create 3 PVCs
---


| POD Name          | StorageClass Name | PV Name   | PVC Name              |
| :---------------- | :---------------: | :-------: | :--------------------:|
| nginx-sts-0    |   nfs             | pv-sts-0  | pg-pvc-nginx-sts-0 |
| nginx-sts-1    |   nfs             | pv-sts-1  | pg-pvc-nginx-sts-1 |
| nginx-sts-2    |  nfs              | pv-sts-2  | pg-pvc-nginx-sts-2 |

### _PVC NAME_
-  <claim_template_name>-<statefulset_name>-<statefulset_number>



```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: pv-sts-0
  labels:
    type: nfs
spec:
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  capacity:
    storage: 3Gi
  volumeMode: Filesystem
  nfs:
    path: /var/nfs/database/pg-0
    server: 192.168.1.34   # NFS server IP address
  storageClassName: nfs
  mountOptions:
    - hard
    - nfsvers=4.1
---
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: pv-sts-1
  labels:
    type: nfs
spec:
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  capacity:
    storage: 3Gi
  volumeMode: Filesystem
  nfs:
    path: /var/nfs/database/pg-1
    server: 192.168.1.34   # NFS server IP address
  storageClassName: nfs
  mountOptions:
    - hard
    - nfsvers=4.1
---
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: pv-sts-2
  labels:
    type: nfs
spec:
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  capacity:
    storage: 3Gi
  volumeMode: Filesystem
  nfs:
    path: /var/nfs/database/pg-2
    server: 192.168.1.34   # NFS server IP address
  storageClassName: nfs
  mountOptions:
    - hard
    - nfsvers=4.1
EOF
```

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pg-pvc-nginx-sts-0 # <claim template name>-<stateful set name>-<stateful set number>
  namespace: default
spec:
  storageClassName: nfs # for the NFS
  volumeName: pv-sts-0
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pg-pvc-nginx-sts-1 # <claim template name>-<stateful set name>-<stateful set number>
  namespace: default
spec:
  storageClassName: nfs # for the NFS
  volumeName: pv-sts-1
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pg-pvc-nginx-sts-2 # <claim template name>-<stateful set name>-<stateful set number>
  namespace: default
spec:
  storageClassName: nfs # for the NFS
  volumeName: pv-sts-2
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
EOF
```


### Create the Statefulset with headless service.
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx-sts
  labels:
    app: nginx-sts
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx-sts
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-sts
spec:
  serviceName: "svc-nginx-sts"
  replicas: 2
  selector:
    matchLabels:
      app: nginx-sts
  template:
    metadata:
      labels:
        app: nginx-sts
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: pg-pvc
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: pg-pvc
    spec:
      storageClassName: nfs
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
EOF
```
### Check the pods
```
kubectl get pods
```

### How to check the PV,PVC
```
kubectl get pv,pvc
```


### How to check the pod name entries on DNS server?
```
kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
```

```
kubectl get pods dnsutils
```


```
kubectl exec -i -t dnsutils -- nslookup kubernetes.default
```


```
kubectl get service
```

```
kubectl exec -ti dnsutils -- cat /etc/resolv.conf
```


```
kubectl get pods -o wide
```
```
kubectl exec -i -t dnsutils -- nslookup  IP_Address_POD
``` 

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

### How to scale up the StatefulSet ?
```
kubectl scale statefulset nginx-sts --replicas=3
```

```
kubectl get pods
```


### How to clear the lab?

```
kubectl delete statefulsets.apps nginx-sts 
kubectl delete persistentvolumeclaim/pg-pvc-nginx-sts-0 persistentvolumeclaim/pg-pvc-nginx-sts-1 persistentvolumeclaim/pg-pvc-nginx-sts-2 persistentvolume/pv-sts-0 persistentvolume/pv-sts-1 persistentvolume/pv-sts-2
```
