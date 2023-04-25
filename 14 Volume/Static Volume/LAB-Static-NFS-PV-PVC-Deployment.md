# LAB for NFS Static Volume

## Create the PV, PVC and Deployment.
```yaml
cat <<EOF>> LAB-Static-NFS-PV-PVC-Deployment.yaml
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
kubectl apply -f LAB-Static-NFS-PV-PVC-Deployment.yaml
```
## Check the status of PV and PVC
```
kubectl get pv,pvc
```
```
[root@master1 volume]# kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/nfs-pv   3Gi        RWX            Retain           Bound    default/task-pv-claim   nfs                     33s

NAME                                  STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/task-pv-claim   Bound    nfs-pv   3Gi        RWX            nfs            34s

```

```
 kubectl get pods
 ```
```
[root@master1 volume]# kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
static-deployment1-88b7d5d8c-7r9rt   0/1     Running   0          26m
```
<pre>
root@static-deployment1-88b7d5d8c-7r9rt:/# df -h
Filesystem                   Size  Used Avail Use% Mounted on
overlay                       14G  6.1G  8.0G  44% /
tmpfs                         64M     0   64M   0% /dev
shm                           64M     0   64M   0% /dev/shm
/dev/sda2                     14G  6.1G  8.0G  44% /etc/hosts
192.168.1.34:/home/nfsshare   15G  4.8G   11G  32% **/var/www/html **  >>>>>>>
tmpfs                        2.1G   12K  2.1G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                        1.1G     0  1.1G   0% /proc/acpi
tmpfs                        1.1G     0  1.1G   0% /proc/scsi
tmpfs                        1.1G     0  1.1G   0% /sys/firmware
</pre>

```
[root@master1 volume]# kubectl exec -it pods/static-deployment1-88b7d5d8c-7r9rt -- /bin/bash
root@static-deployment1-88b7d5d8c-7n7k8:/# cd /var/www/html/
root@static-deployment1-88b7d5d8c-7n7k8:/var/www/html#
```

## Create the file in the pod.

```
root@static-deployment1-88b7d5d8c-7r9rt:/var/www/html# ls -ltr
total 0

root@static-deployment1-88b7d5d8c-7r9rt:/var/www/html# echo "1st line update from static-nfs-pod" > static-file1.txt

root@static-deployment1-88b7d5d8c-7r9rt:/var/www/html# cat static-file1.txt 
1st line update from static-nfs-pod

root@static-deployment1-88b7d5d8c-7r9rt:/var/www/html# 

```

## Check the NFS server on this particular location
```
[root@server1 ~]# cd /home/nfsshare/
[root@server1 nfsshare]# ls -ltr
total 4
-rw-r--r--. 1 root root 36 Apr 23 16:01 static-file1.txt
[root@server1 nfsshare]# cat static-file1.txt 
1st line update from static-nfs-pod
[root@server1 nfsshare]# 
```

## Update the file on NFS server.
```
[root@server1 nfsshare]# echo "2nd line update from NFS server" >> static-file1.txt
[root@server1 nfsshare]# cat static-file1.txt 
1st line update from static-nfs-pod
2nd line update from NFS server
[root@server1 nfsshare]# 
```

## Check the file on POD, if this file is also updated?
```
root@static-deployment1-88b7d5d8c-7r9rt:/var/www/html# cat static-file1.txt 
1st line update from static-nfs-pod
2nd line update from NFS server
root@static-deployment1-88b7d5d8c-7r9rt:/var/www/html# 
```

## Delete the POD and a new POD will be created, thanks to Deployment and verify this file is still exist?
```
[root@master1 volume]# kubectl delete pod/static-deployment1-88b7d5d8c-7r9rt 
pod "static-deployment1-88b7d5d8c-7r9rt" deleted
[root@master1 volume]# kubectl get pod
NAME                                 READY   STATUS    RESTARTS   AGE
static-deployment1-88b7d5d8c-4h9zn   1/1     Running   0          6s
[root@master1 volume]# kubectl exec -it static-deployment1-88b7d5d8c-4h9zn -- /bin/bash
root@static-deployment1-88b7d5d8c-4h9zn:/# cd /var/www/html/
root@static-deployment1-88b7d5d8c-4h9zn:/var/www/html# cat static-file1.txt 
1st line update from static-nfs-pod
2nd line update from NFS server
root@static-deployment1-88b7d5d8c-4h9zn:/var/www/html# 
```


## Now, delete the Deployment, PVC and PV and then create it again and check if we still see the file?
```
[root@master1 volume]# kubectl delete -f deployment.yaml -f PersistentVolumeClaim.yaml -f PersistentVolume.yaml
deployment.apps "static-deployment1" deleted
persistentvolumeclaim "task-pv-claim" deleted
persistentvolume "nfs-pv" deleted

[root@master1 volume]# kubectl apply -f deployment.yaml -f PersistentVolumeClaim.yaml -f PersistentVolume.yaml
deployment.apps/static-deployment1 created
persistentvolumeclaim/task-pv-claim created
persistentvolume/nfs-pv created

[root@master1 volume]# kubectl get pods 
NAME                                 READY   STATUS    RESTARTS   AGE
static-deployment1-88b7d5d8c-m7ztz   0/1     Running   0          12s

[root@master1 volume]# kubectl exec -it static-deployment1-88b7d5d8c-m7ztz -- /bin/bash
root@static-deployment1-88b7d5d8c-m7ztz:/# cd /var/www/html/
root@static-deployment1-88b7d5d8c-m7ztz:/var/www/html# cat static-file1.txt 
1st line update from static-nfs-pod
2nd line update from NFS server
root@static-deployment1-88b7d5d8c-m7ztz:/var/www/html# exit
```

## clear the lab
```
kubectl delete -f deployment.yaml -f PersistentVolumeClaim.yaml -f PersistentVolume.yaml
rm -f PersistentVolumeClaim.yaml deployment.yaml PersistentVolume.yaml
[root@server1 nfsshare]# rm -f static-file1.txt 
[root@server1 nfsshare]#
```
