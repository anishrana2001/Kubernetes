# How to upgrade the Kubernetes Cluster 

```
[root@master1 ~]# kubectl get nodes
NAME                      STATUS   ROLES           AGE    VERSION
master1.example.com       Ready    control-plane   345d   v1.25.4
workernode1.example.com   Ready    <none>          345d   v1.25.4
workernode2.example.com   Ready    <none>          345d   v1.25.4
[root@master1 ~]# kubectl get pods -owide -n kube-system;  
NAME                                          READY   STATUS    RESTARTS       AGE    IP             NODE                      NOMINATED NODE   READINESS GATES
calico-kube-controllers-798cc86c47-phlnc      1/1     Running   32 (21m ago)   345d   172.16.68.63   master1.example.com       <none>           <none>
calico-node-672fx                             1/1     Running   29 (21m ago)   345d   192.168.1.31   master1.example.com       <none>           <none>
calico-node-c5kn4                             1/1     Running   45 (17m ago)   345d   192.168.1.32   workernode1.example.com   <none>           <none>
calico-node-tszjn                             1/1     Running   43 (13m ago)   345d   192.168.1.33   workernode2.example.com   <none>           <none>
coredns-565d847f94-m4pjz                      1/1     Running   25 (21m ago)   345d   172.16.68.62   master1.example.com       <none>           <none>
coredns-565d847f94-z2crd                      1/1     Running   24 (22m ago)   345d   172.16.68.61   master1.example.com       <none>           <none>
etcd-master1.example.com                      1/1     Running   18 (21m ago)   345d   192.168.1.31   master1.example.com       <none>           <none>
kube-apiserver-master1.example.com            1/1     Running   27 (17m ago)   345d   192.168.1.31   master1.example.com       <none>           <none>
kube-controller-manager-master1.example.com   1/1     Running   50 (16m ago)   345d   192.168.1.31   master1.example.com       <none>           <none>
kube-proxy-4nkf7                              1/1     Running   41 (17m ago)   345d   192.168.1.32   workernode1.example.com   <none>           <none>
kube-proxy-g2pwp                              1/1     Running   42 (13m ago)   345d   192.168.1.33   workernode2.example.com   <none>           <none>
kube-proxy-hv8wh                              1/1     Running   20 (22m ago)   345d   192.168.1.31   master1.example.com       <none>           <none>
kube-scheduler-master1.example.com            1/1     Running   49 (21m ago)   345d   192.168.1.31   master1.example.com       <none>           <none>

[root@master1 ~]# kubectl drain master1.example.com --ignore-daemonsets
node/master1.example.com cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/calico-node-672fx, kube-system/kube-proxy-hv8wh
evicting pod kube-system/calico-kube-controllers-798cc86c47-phlnc
evicting pod kube-system/coredns-565d847f94-z2crd
evicting pod kube-system/coredns-565d847f94-m4pjz
pod/calico-kube-controllers-798cc86c47-phlnc evicted
pod/coredns-565d847f94-z2crd evicted
pod/coredns-565d847f94-m4pjz evicted
node/master1.example.com drained


[root@master1 ~]#  kubectl get pods -owide -n kube-system
NAME                                          READY   STATUS    RESTARTS       AGE    IP               NODE                      NOMINATED NODE   READINESS GATES
kube-apiserver-master1.example.com            1/1     Running   27 (18m ago)   345d   192.168.1.31     master1.example.com       <none>           <none>
etcd-master1.example.com                      1/1     Running   18 (23m ago)   345d   192.168.1.31     master1.example.com       <none>           <none>
kube-scheduler-master1.example.com            1/1     Running   49 (23m ago)   345d   192.168.1.31     master1.example.com       <none>           <none>
kube-controller-manager-master1.example.com   1/1     Running   50 (17m ago)   345d   192.168.1.31     master1.example.com       <none>           <none>
kube-proxy-hv8wh                              1/1     Running   20 (23m ago)   345d   192.168.1.31     master1.example.com       <none>           <none>
calico-node-672fx                             1/1     Running   29 (23m ago)   345d   192.168.1.31     master1.example.com       <none>           <none>
calico-kube-controllers-798cc86c47-8j46s      1/1     Running   0              77s    172.16.133.190   workernode1.example.com   <none>           <none>
kube-proxy-4nkf7                              1/1     Running   41 (18m ago)   345d   192.168.1.32     workernode1.example.com   <none>           <none>
kube-proxy-g2pwp                              1/1     Running   42 (14m ago)   345d   192.168.1.33     workernode2.example.com   <none>           <none>
calico-node-c5kn4                             1/1     Running   45 (18m ago)   345d   192.168.1.32     workernode1.example.com   <none>           <none>
calico-node-tszjn                             1/1     Running   43 (14m ago)   345d   192.168.1.33     workernode2.example.com   <none>           <none>
coredns-565d847f94-7zgwp                      1/1     Running   0              77s    172.16.14.95     workernode2.example.com   <none>           <none>
coredns-565d847f94-f9vs4                      1/1     Running   0              77s    172.16.133.191   workernode1.example.com   <none>           <none>



Follow the URL : https://v1-27.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
In this I have selected the upgrade from 1.26 to 1.27. You can select yours. 
```
