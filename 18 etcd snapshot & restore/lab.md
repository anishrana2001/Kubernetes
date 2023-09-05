# LAB for taking snapshot

##  First, create a snapshot of the existing etcd instance running at https://127.0.0.1:2379, saving the snapshot to /var/lib/etcd-snapshot.db.
## Next, restore an existing, previous snapshot located at /var/lib/etcd-snapshot.db.

## Solution, while using etcd static pod yaml file "/etc/kubernetes/manifests/etcd.yaml"

```
cat /etc/kubernetes/manifests/etcd.yaml 
```
```
ETCDCTL_API=3 etcdctl                      \
--endpoints=https://127.0.0.1:2379         \
--cacert=/etc/kubernetes/pki/etcd/ca.crt   \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key  \
snapshot save /var/lib/etcd-snapshot.db
```

## Now, how to restore it?
```
kubectl -n kube-system describe pods/etcd-master1.example.com | egrep "data-dir"
```
``
--data-dir=/var/lib/etcd
``

```
ls -ld /var/lib/etcd
```

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
ls -ld /var/lib/etcd-backup
```

## If you observe Permission deny error message then, 
```
sudo ETCDCTL_API=3 etcdctl                   \
--endpoints=https://127.0.0.1:2379           \
--cacert=/etc/kubernetes/pki/etcd/ca.crt     \
--cert=/etc/kubernetes/pki/etcd/server.crt   \
--key=/etc/kubernetes/pki/etcd/server.key    \
snapshot restore /var/lib/etcd-snapshot.db   \
--data-dir="/var/lib/etcd-backup"
```
## If still observing error message, then check 
sudo chown -R etcd:etcd /var/lib/etcd
sudo systemctl start etcd
