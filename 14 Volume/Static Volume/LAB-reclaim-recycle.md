# Lab for PV reclaim policy 

## Create the PV, PVC and Deployment.
```yaml
cat <<EOF>> reclaim-policy-recycle.yaml
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: nfs-pv
spec:
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle   #### Data will be deleted.
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
kubectl apply -f reclaim-policy-recycle.yaml
```


## Check the status of PV, PVC and Deployment.
```
[root@master1 volume]# kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/nfs-pv   3Gi        RWX            Recycle          Bound    default/task-pv-claim   nfs                     6s

NAME                                  STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/task-pv-claim   Bound    nfs-pv   3Gi        RWX            nfs            6s
```

## Check the status of POD
```
kubectl get pods
```
```
[root@master1 volume]# kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
static-deployment1-88b7d5d8c-cqf8f   1/1     Running   0          22s
```

### Login into the POD. 
```
kubectl exec -it $(kubectl get pod | grep static-deployment1 | awk '{print $1}') -- /bin/bash
```

### Now, create one file and check this file on NFS server. 

```
[root@master1 volume]# kubectl exec -it $(kubectl get pod | grep static-deployment1 | awk '{print $1}') -- /bin/bash
root@static-deployment1-88b7d5d8c-cqf8f:/# cd /var/www/html/

root@static-deployment1-88b7d5d8c-cqf8f:/var/www/html# echo "1st line update from static-nfs-pod" > relaim-recycle.txt

root@static-deployment1-88b7d5d8c-cqf8f:/var/www/html# cat relaim-recycle.txt 
1st line update from static-nfs-pod

root@static-deployment1-88b7d5d8c-cqf8f:/var/www/html# exit
exit

[root@server1 nfsshare]# ls -ltr
total 4
-rw-r--r--. 1 root root 36 Apr 25 20:26 relaim-recycle.txt

[root@server1 nfsshare]# cat relaim-recycle.txt 
1st line update from static-nfs-pod
[root@server1 nfsshare]# 


```

### Its time to delete the deployment first and check the status of PV,PVC and NFS server.

```
kubectl delete deployments.apps static-deployment1
```

```
[root@master1 volume]# kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/nfs-pv   3Gi        RWX            Recycle          Bound    default/task-pv-claim   nfs                     4m37s

NAME                                  STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/task-pv-claim   Bound    nfs-pv   3Gi        RWX            nfs            4m37s


[root@server1 nfsshare]# cat relaim-recycle.txt 
1st line update from static-nfs-pod

```

## Now, delete the PVC
```
kubectl delete pvc/task-pv-claim
kubectl get pv,pvc
```
### Observe the column Status of of PV, it should be Released. But this time, I am not able to see/read this file on NFS server. It means that if we select the reclaim policy to Recycle in PV config file, it will delete the data from the external storage server if we delete the PVC.

```
[root@master1 volume]# kubectl delete pvc/task-pv-claim

[root@master1 volume]# kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/nfs-pv   3Gi        RWX            Recycle          Released   default/task-pv-claim   nfs                     5m21s


[root@server1 nfsshare]# cat relaim-recycle.txt 
cat: relaim-recycle.txt: No such file or directory

```


## Clear the lab from master server.
```
kubectl delete pv/nfs-pv
```
