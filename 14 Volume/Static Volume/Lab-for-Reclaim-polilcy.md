# Lab for PV reclaim policy 

## persistentVolumeReclaimPolicy set to delete
```
[root@master1 volume]# cat PersistentVolume.yaml
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: nfs-pv
spec:
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Delete
  capacity:
    storage: 3Gi
  volumeMode: Filesystem
  nfs:
    path: /home/nfsshare
    server: 192.168.1.34   # NFS server IP address
  storageClassName: nfs
  mountOptions:
    - hard
    - nfsvers=4.1
```
## Create the PV and PVC. Please note previous PVC yaml file is being used here.
```
[root@master1 volume]# kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/nfs-pv   3Gi        RWX            Delete           Bound    default/task-pv-claim   nfs                     16s

NAME                                  STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/task-pv-claim   Bound    nfs-pv   3Gi        RWX            nfs            16s
```

## Now, delete the PVC
```
[root@master1 volume]# kubectl delete -f PersistentVolumeClaim.yaml 
persistentvolumeclaim "task-pv-claim" deleted
```
## Check the status coloumn of PV. You should observed Faiiled.
```
[root@master1 volume]# kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/nfs-pv   3Gi        RWX            Delete           Failed   default/task-pv-claim   nfs                     47s
```
## Let's delete the PV 
```
[root@master1 volume]# kubectl delete -f  PersistentVolume.yaml 
persistentvolume "nfs-pv" deleted
```

 ## LAB for Reclaim Policy for Retain option
```
[root@master1 volume]# cat PersistentVolume.yaml 
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: nfs-pv
spec:
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  capacity:
    storage: 3Gi
  volumeMode: Filesystem
  nfs:
    path: /home/nfsshare
    server: 192.168.1.34   # NFS server IP address
  storageClassName: nfs
  mountOptions:
    - hard
    - nfsvers=4.1
```
## Create the PV & PVC
```
[root@master1 volume]# kubectl apply -f PersistentVolumeClaim.yaml -f PersistentVolume.yaml
persistentvolumeclaim/task-pv-claim created
persistentvolume/nfs-pv created

```

## Check the status of PV & PVC
```
[root@master1 volume]# kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/nfs-pv   3Gi        RWX            Retain           Bound    default/task-pv-claim   nfs                     52s

NAME                                  STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/task-pv-claim   Bound    nfs-pv   3Gi        RWX            nfs            52s
```

## Now, delete the PersistentVolumeClaim and see the status coloumn of PV. It should be Released state. 
```
[root@master1 volume]# kubectl delete -f PersistentVolumeClaim.yaml 
persistentvolumeclaim "task-pv-claim" deleted

[root@master1 volume]# kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/nfs-pv   3Gi        RWX            Retain           Released   default/task-pv-claim   nfs                     2m8s
```

