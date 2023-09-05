# LAB for taking snapshot

##  First, create a snapshot of the existing etcd instance running at https://127.0.0.1:2379, saving the snapshot to /var/lib/etcd-snapshot.db.
## Next, restore an existing, previous snapshot located at /var/lib/etcd-snapshot-previous.db.

## Solution, while using etcd static pod yaml file "/etc/kubernetes/manifests/etcd.yaml"

```
cat /etc/kubernetes/manifests/etcd.yaml 
```
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /var/lib/etcd-snapshot-previous.db
```

## Now, how to restore it?
```
kubectl -n kube-system describe pods/etcd-master1.example.com | egrep "data-dir"
```
``
--data-dir=/var/lib/etcd
``

```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot restore /var/lib/etcd-snapshot-previous.db --data-dir="/var/lib/etcd-backup"
```
```
ls -ld /var/lib/etcd-backup
```
```
kubectl run post-check-pod --image=nginx
```
```
kubectl get pods
```
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

### It take sometime, get the response. 
```
kubectl get pods
```

[root@master1 ~]# kubectl get pods
No resources found in default namespace.

```
kubectl get pods -A | grep etcd
```

[root@master1 ~]# kubectl get pods -A | grep etcd
kube-system   etcd-master1.example.com                      0/1     Pending   0              15s
[root@master1 ~]# 

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


