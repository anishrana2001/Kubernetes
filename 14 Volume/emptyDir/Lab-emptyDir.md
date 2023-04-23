# Lab for emptyDir 

## Pod yaml file for emptyDir volume
```
cat <<EOF>> pod-emptyDir.yaml
kind: Namespace
apiVersion: v1
metadata:
  name: core1
---
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod2
  namespace: core1
spec:
  volumes:
  - name: vol1-emptydir
    emptyDir:
      sizeLimit: 500Mi
  containers:
  - image: nginx
    name: emptydir-container2
    volumeMounts:
    - mountPath: /cache
      name: vol1-emptydir
EOF
```

## Create a POD with this yaml file.

```
kubectl apply -f pod-emptyDir.yaml 
```

## Check the pod status
```
[root@master1 volume]# kubectl -n core1 get pods -owide
NAME            READY   STATUS    RESTARTS   AGE   IP             NODE                      NOMINATED NODE   READINESS GATES
emptydir-pod2   1/1     Running   0          19s   172.16.14.88   workernode2.example.com   <none>           <none>
```
## Check the directory on Workernode2

```
[root@master1 volume]# kubectl -n core1 exec -it emptydir-pod2 -- /bin/bash

root@emptydir-pod2:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay          14G  6.1G  8.0G  44% /
tmpfs            64M     0   64M   0% /dev
/dev/sda2        14G  6.1G  8.0G  44% /cache  #>>> This is the directory 
shm              64M     0   64M   0% /dev/shm
tmpfs           2.1G   12K  2.1G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs           1.1G     0  1.1G   0% /proc/acpi
tmpfs           1.1G     0  1.1G   0% /proc/scsi
tmpfs           1.1G     0  1.1G   0% /sys/firmware
```
## let's create one file in this directory (/cache)
```
root@emptydir-pod2:/cache# echo "1st line updated from the container" > file1.txt
root@emptydir-pod2:/cache# cat file1.txt 
1st line updated from the container
```

## Describe the pod to know the workdernode2 path 
```
[root@master1 pods]# kubectl -n core1 get pod/emptydir-pod2 -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: 4ec38cbd55ff426661a904f18ec8d694e696efa7f61ae297a6d2c36a87a9b443
    cni.projectcalico.org/podIP: 172.16.14.88/32
    cni.projectcalico.org/podIPs: 172.16.14.88/32
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"emptydir-pod2","namespace":"core1"},"spec":{"containers":[{"image":"nginx","name":"emptydir-container2","volumeMounts":[{"mountPath":"/cache","name":"vol1-emptydir"}]}],"volumes":[{"emptyDir":{"sizeLimit":"500Mi"},"name":"vol1-emptydir"}]}}
  creationTimestamp: "2023-04-23T06:51:30Z"
  name: emptydir-pod2
  namespace: core1
  resourceVersion: "87970"
  uid: 41b7549e-3521-490c-9d6e-450346887aea   ---> ID of this pod.
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: emptydir-container2
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /cache
      name: vol1-emptydir
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount  
      name: kube-api-access-rn7dw
      readOnly: true
  dnsPolicy: ClusterFirst
  ```
  
  ## From the above command output, we get to know the id of pod
  ```
  41b7549e-3521-490c-9d6e-450346887aea
  ```
  
  ## On Workernode2, go to this directory "/var/lib/kubelet/pods/"
 ```
  [root@workernode2 ~]# cd /var/lib/kubelet/pods
[root@workernode2 pods]# ls -l 
total 0
drwxr-x---  5 root root 71 Apr 23 12:21 41b7549e-3521-490c-9d6e-450346887aea   >>>> This is the POD ID.
drwxr-x---  5 root root 71 Apr 23 08:11 5ee04802-b89f-408b-a5b3-9d40ae7ce359
drwxr-x---. 5 root root 71 Dec  7 22:24 dcf7999a-e8d5-437d-aecb-3d9c50938977
drwxr-x---. 5 root root 71 Dec  7 22:24 faee1ee3-43b5-4586-9787-b7c85883fcc9
[root@workernode2 pods]# 
 ```
 ```
[root@workernode2 pods]# cd 41b7549e-3521-490c-9d6e-450346887aea/
[root@workernode2 41b7549e-3521-490c-9d6e-450346887aea]# ls -ltr
total 4
drwxr-x--- 3 root root  37 Apr 23 12:21 plugins
drwxr-x--- 4 root root  68 Apr 23 12:21 volumes
-rw-r--r-- 1 root root 210 Apr 23 12:21 etc-hosts
drwxr-x--- 3 root root  33 Apr 23 12:21 containers
[root@workernode2 41b7549e-3521-490c-9d6e-450346887aea]# cd volumes/

[root@workernode2 volumes]# ls -ltr
total 0
drwxr-xr-x 3 root root 27 Apr 23 12:21 kubernetes.io~empty-dir
drwxr-xr-x 3 root root 35 Apr 23 12:21 kubernetes.io~projected

[root@workernode2 volumes]# cd kubernetes.io~empty-dir/

[root@workernode2 kubernetes.io~empty-dir]# ls -ltr
total 0
drwxrwxrwx 2 root root 23 Apr 23 13:06 vol1-emptydir

[root@workernode2 kubernetes.io~empty-dir]# cd vol1-emptydir/

[root@workernode2 vol1-emptydir]# ls
file1.txt

[root@workernode2 vol1-emptydir]# cat file1.txt 
1st line updated from the container
[root@workernode2 vol1-emptydir]# 

```

## Update the file on workernode2 

```
[root@workernode2 vol1-emptydir]# echo "2nd line updated from workernode2" >> file1.txt 
[root@workernode2 vol1-emptydir]# cat file1.txt 
1st line updated from the container
2nd line updated from workernode2
[root@workernode2 vol1-emptydir]# 
```

## Check the file if it is updated?

```
root@emptydir-pod2:/cache# cat file1.txt 
1st line updated from the container
2nd line updated from workernode2
root@emptydir-pod2:/cache# 
```

## Now, removed this pod and check if file1.txt will be exist on workernode2?

```
[root@master1 volume]# kubectl delete -f pod-emptyDir.yaml 
namespace "core1" deleted
pod "emptydir-pod2" deleted
[root@master1 volume]# 
```

## You will notice, POD directory removed, thus file1.txt also get deleted. 

```
[root@workernode2 ~]# ls -ltr /var/lib/kubelet/pods/
total 0
drwxr-x---. 5 root root 71 Dec  7 22:24 dcf7999a-e8d5-437d-aecb-3d9c50938977
drwxr-x---. 5 root root 71 Dec  7 22:24 faee1ee3-43b5-4586-9787-b7c85883fcc9
drwxr-x---  5 root root 71 Apr 23 08:11 5ee04802-b89f-408b-a5b3-9d40ae7ce359
[root@workernode2 ~]#
```

## Clear the lab 
```
kubectl delete -f pod-emptyDir.yaml
```



