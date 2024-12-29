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
| postgres-sts-0    |   nfs             | pv-sts-0  | pg-pvc-postgres-sts-0 |
| postgres-sts-1    |   nfs             | pv-sts-1  | pg-pvc-postgres-sts-1 |
| postgres-sts-2    |  nfs              | pv-sts-2  | pg-pvc-postgres-sts-2 |

### _PVC NAME_
## - <claim template name>-<stateful set name>-<stateful set number>



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
  name: pg-pvc-postgres-sts-0 # <claim template name>-<stateful set name>-<stateful set number>
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
  name: pg-pvc-postgres-sts-1 # <claim template name>-<stateful set name>-<stateful set number>
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
  name: pg-pvc-postgres-sts-2 # <claim template name>-<stateful set name>-<stateful set number>
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
  name: nginx
  labels:
    app: postgres
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: postgres
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-sts
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.21
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

 ### How to scale up the StatefulSet ?
 ```
 kubectl scale statefulset postgres-sts --replicas=3
 ```

```
 kubectl get pods
 ```
