

# LAB for taking the ETCD snapshot, restore and validate it.

## First, we are going to create a new POD so that we can validate our restore process.

```
kubectl run pre-check-pod --image=nginx
```
```
kubectl get pods
```


##  Let's take the snapshot, now.

```
cat /etc/kubernetes/manifests/etcd.yaml 
```
## OR we can execute this command.
```
kubectl -n kube-system describe pod/etcd-master1.example.com
```
```
ETCDCTL_API=3 etcdctl                      \
--endpoints=https://127.0.0.1:2379         \
--cacert=/etc/kubernetes/pki/etcd/ca.crt   \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key  \
snapshot save /var/lib/etcd-snapshot.db
```

### Now again, we create a new pod. The reason behind is that after restore the snapshot, I should not see this pod.

```
kubectl run post-check-pod --image=nginx
```
```
kubectl get pods
```


## Now, how to restore it? 
### Let's assume that ETCD "data directory" is corrupted. In this case, we can restore the snapshot.
### Data directory is the directory, where all the ETCD's configuration and data files are present.
### How we can find the ETCD data directory?

```
cat /etc/kubernetes/manifests/etcd.yaml | egrep "data-dir"
```
``
--data-dir=/var/lib/etcd
``

## OR, we can execute this command.

```
kubectl -n kube-system describe pod/etcd-master1.example.com | grep "data-dir"
```

```
ls -ld /var/lib/etcd
```
### How to restore the ETCD snapshot?
### This time, we have to create a new data directory, thus, we will use "--data-dir="/var/lib/etcd-backup". We can use any directory name.
### Good thing is that, we have already knows the endpoints, cacert, cert and key.

```
--endpoints=https://127.0.0.1:2379           \
--cacert=/etc/kubernetes/pki/etcd/ca.crt     \
--cert=/etc/kubernetes/pki/etcd/server.crt   \
--key=/etc/kubernetes/pki/etcd/server.key    \
```

### Previously, we took the snapshot on directory "/var/lib/etcd-snapshot.db"


```
ETCDCTL_API=3 etcdctl                        \
--endpoints=https://127.0.0.1:2379           \
--cacert=/etc/kubernetes/pki/etcd/ca.crt     \
--cert=/etc/kubernetes/pki/etcd/server.crt   \
--key=/etc/kubernetes/pki/etcd/server.key    \
snapshot restore /var/lib/etcd-snapshot.db   \
--data-dir="/var/lib/etcd-backup"
```

```
ls -ld /var/lib/etcd
```

```
ls -ld /var/lib/etcd-backup
```

## Now, we have changed the directory "--data-dir", thus we have to change the ownership of this directory also. This step only applicable where we have standalone etcd server.

sudo chown -R etcd:etcd /var/lib/etcd-backup

sudo systemctl start etcd


### We have successfully restore the snapshot. Checking the pods status.

```
kubectl get pods
```

### Why we are observing these pods?
### We have restored the snapshot but in the ETCD static pod yaml file, data-dir is still the old one. Let's confirm it.

```
cat /etc/kubernetes/manifests/etcd.yaml | egrep "data-dir"
```

### We have to update the new data directory on all other sections too.

```
cat /etc/kubernetes/manifests/etcd.yaml | grep /var/lib/etcd
```

[root@master1 ~]# cat /etc/kubernetes/manifests/etcd.yaml | grep /var/lib/etcd

    - --data-dir=/var/lib/etcd

    - mountPath: /var/lib/etcd
    
      path: /var/lib/etcd

```
sed -i 's=/var/lib/etcd=/var/lib/etcd-backup=' /etc/kubernetes/manifests/etcd.yaml 
```
```
cat /etc/kubernetes/manifests/etcd.yaml | grep /var/lib/etcd
```

[root@master1 ~]# sed -i 's=/var/lib/etcd=/var/lib/etcd-backup=' /etc/kubernetes/manifests/etcd.yaml 

[root@master1 ~]# cat /etc/kubernetes/manifests/etcd.yaml | grep /var/lib/etcd

    - --data-dir=/var/lib/etcd-backup
    - mountPath: /var/lib/etcd-backup
      path: /var/lib/etcd-backup


### If we execute this command "kubectl get pods", it take sometime. 
```
kubectl get pods
```

[root@master1 ~]# kubectl get pods

``
No resources found in default namespace.
``
```
kubectl get pods -A | grep etcd
```

[root@master1 ~]# kubectl get pods -A | grep etcd

``
kube-system   etcd-master1.example.com                           0/1            Pending           0                15s
``

```
kubectl -n kube-system get pods/etcd-master1.example.com -o yaml > etc.yaml
```
```
kubectl -n kube-system delete pod/etcd-master1.example.com 
```
```
kubectl apply -f etc.yaml
```
```
kubectl get pods -A | grep etc
```
```
kubectl get pods
```


## clear the lab and back to the first stage.

```
rm -rvf /var/lib/etcd/

ETCDCTL_API=3 etcdctl                        \
--endpoints=https://127.0.0.1:2379           \
--cacert=/etc/kubernetes/pki/etcd/ca.crt     \
--cert=/etc/kubernetes/pki/etcd/server.crt   \
--key=/etc/kubernetes/pki/etcd/server.key    \
snapshot restore /var/lib/etcd-snapshot.db   \
--data-dir="/var/lib/etcd"

kubectl delete pods/pre-check-pod

rm -rvf /var/lib/etcd-backup
```
