# hostPath Volume
## POD yaml file for hostPath volume
```
cat <<EOF>> hostPath-pod.yaml
kind: Namespace
apiVersion: v1
metadata:
  name: core
---
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod2
  namespace: core
spec:
  volumes:
  - name: vol1-hostpath
    hostPath:
      path: /home/nfssharedata
      type: DirectoryOrCreate
  containers:
  - image: nginx
    name: hostpath-container2
    volumeMounts:
    - mountPath: /vol1
      name: vol1-hostpath
    EOF
```

## create pod through yaml file

```
kubectl create -f hostPath-pod.yaml
```

## Check the status
```
kubectl -n core get all
```

```
[root@master1 volume]# kubectl -n core get all
NAME                READY   STATUS    RESTARTS   AGE
pod/hostpath-pod2   1/1     Running   0          10s
```

## Login to the pod 
```
kubectl -n core exec -it pods/hostpath-pod2 -- /bin/bash
```

## Verify the volume

```
root@hostpath-pod2:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay          14G  6.1G  8.0G  44% /
tmpfs            64M     0   64M   0% /dev
/dev/sda2        14G  6.1G  8.0G  44% /vol1  #########
shm              64M     0   64M   0% /dev/shm
tmpfs           2.1G   12K  2.1G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs           1.1G     0  1.1G   0% /proc/acpi
tmpfs           1.1G     0  1.1G   0% /proc/scsi
tmpfs           1.1G     0  1.1G   0% /sys/firmware
```

## Create the file under volume director in the container.
```
root@hostpath-pod2:/# cd /vol1/

root@hostpath-pod2:/vol1# ls -ltr
total 0

root@hostpath-pod2:/vol1# echo "Line udpated from container" > file1.txt

root@hostpath-pod2:/vol1# cat file1.txt 
Line udpated from container
root@hostpath-pod2:/vol1# 
```

## Check the pod on which workernode it is created.
```
[root@master1 volume]# kubectl -n core get pods -o wide
NAME            READY   STATUS    RESTARTS   AGE   IP             NODE                      NOMINATED NODE   READINESS GATES
hostpath-pod2   1/1     Running   0          23m   172.16.14.84   workernode2.example.com   <none>           <none>
[root@master1 volume]# 
```

## Go to the directory on this workernode2 and update the file1.txt

```
[root@**workernode2** ~]# ls -l /home/nfssharedata/
total 4
-rw-r--r-- 1 root root 28 Apr 23 11:30 file1.txt

[root@workernode2 ~]# cat /home/nfssharedata/file1.txt 
Line udpated from container

[root@workernode2 ~]# echo "2nd line updated from workernode2" >> /home/nfssharedata/file1.txt 

[root@workernode2 ~]# cat /home/nfssharedata/file1.txt 
Line udpated from container
2nd line updated from workernode2
[root@workernode2 ~]# 
```

## Check the udpate file in the container

```
root@hostpath-pod2:/vol1# cat file1.txt 
Line udpated from container
2nd line updated from workernode2
root@hostpath-pod2:/vol1#
```

## Now, delete the pod and check the file1.txt on workernode2
```
[root@master1 volume]# kubectl delete -f hostpath-pod.yaml 
namespace "core" deleted
pod "hostpath-pod2" deleted
[root@master1 volume]#
```

## I can still access the file
```
[root@workernode2 ~]# cat /home/nfssharedata/file1.txt 
Line udpated from container
2nd line updated from workernode2
[root@workernode2 ~]# 
```

## Create the POD againa & check if I can access to file "file1.txt"

```
[root@master1 volume]# kubectl apply -f hostpath-pod.yaml 
namespace/core created
pod/hostpath-pod2 created

[root@master1 volume]# kubectl -n core exec -it pods/hostpath-pod2 -- /bin/bash
root@hostpath-pod2:/# cd /vol1/

root@hostpath-pod2:/vol1# cat file1.txt 
Line udpated from container
2nd line updated from workernode2
root@hostpath-pod2:/vol1# 
```

