# Lab for PV reclaim policy 

## Create the PV, PVC and Deployment.
```yaml
cat <<EOF>> reclaim-policy-Delete.yaml
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: nfs-pv
spec:
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Delete   #### Data will be deleted??
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
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: nfs
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 3Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: static-deployment1
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: static-deployment1
  template:
    metadata:
      labels:
        app: static-deployment1
    spec:
      containers:
        - name: static-nfs-container1
          image: nginx
          volumeMounts:
            - name: static-nfs-volume
              mountPath: "/var/www/html"
      volumes:
        - name: static-nfs-volume
          persistentVolumeClaim: 
            claimName: task-pv-claim
EOF
```

## Apply the yaml file.
```
kubectl apply -f reclaim-policy-Delete.yaml
```


## Check the status of PV, PVC and Deployment.
```
[root@master1 volume]# kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/nfs-pv   3Gi        RWX            Delete           Bound    default/task-pv-claim   nfs                     16s

NAME                                  STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/task-pv-claim   Bound    nfs-pv   3Gi        RWX            nfs            16s
```

## Check the status of POD
```
kubectl get pods
```
```
[root@master1 volume]# kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
static-deployment1-88b7d5d8c-62plb   1/1     Running   0          68s
```

### Login into the POD. 
```
kubectl exec -it static-deployment1-88b7d5d8c-62plb -- /bin/bash
```

### Now, create one file and also updated that file from NFS server. 

```
root@static-deployment1-88b7d5d8c-f5tvh:/# cd /var/www/html/

root@static-deployment1-88b7d5d8c-62plb:/var/www/html# echo "1st line update from static-nfs-pod" > relaim-delete.txt

[root@server1 nfsshare]# echo "2nd line updated from NFS server" >> relaim-delete.txt

root@static-deployment1-88b7d5d8c-62plb:/var/www/html# cat relaim-delete.txt 
1st line update from static-nfs-pod
2nd line updated from NFS server

root@static-deployment1-88b7d5d8c-62plb:/var/www/html# exit

```

### Its time to delete the deployment first and check the status of PV,PVC and NFS server.

```
kubectl delete deployments.apps static-deployment1
```

```
[root@master1 volume]# kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/nfs-pv   3Gi        RWX            Delete           Bound    default/task-pv-claim   nfs                     3m23s

NAME                                  STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/task-pv-claim   Bound    nfs-pv   3Gi        RWX            nfs            3m23s


[root@server1 nfsshare]# cat relaim-delete.txt 
1st line update from static-nfs-pod
2nd line updated from NFS server

```

## Now, delete the PVC
```
kubectl delete pvc/task-pv-claim
kubectl get pv,pvc
```
### Observe the Status of of PV, it should be Failed. But still I can read my file on NFS server. 

### For volume plugins that support the Delete reclaim policy, deletion removes both the PersistentVolume object from Kubernetes, as well as the associated storage asset in the external infrastructure, such as an AWS EBS, GCE PD, Azure Disk, or Cinder volume.


```
[root@master1 volume]# kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/nfs-pv   3Gi        RWX            Delete           Failed   default/task-pv-claim   nfs                     4m50s

[root@server1 nfsshare]# cat relaim-delete.txt 
1st line update from static-nfs-pod
2nd line updated from NFS server

```


## Let's delete the PV 
```
kubectl delete pv nfs-pv
```


### Still, I can see and read this file on NFS. It means that "reclaim policy = Delete" is not supported for NFS in Static PV.
```
[root@server1 nfsshare]# cat relaim-delete.txt 
1st line update from static-nfs-pod
2nd line updated from NFS server
[root@server1 nfsshare]# 
```

## Clear the lab from master server.
```
kubectl delete -f reclaim-policy-Delete.yaml
rm -f reclaim-policy-Delete.yaml
```
### Remove the file from NFS server
```
 rm -f relaim-delete.txt
```
