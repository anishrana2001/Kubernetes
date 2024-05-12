
### Installing the Multus CNI
```
kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/master/deployments/multus-daemonset-thick.yml
```

### How to verify that Multus is installed on the Kubernetes Cluster?
## Check if our pods are running?
```
kubectl get pods --all-namespaces | grep -i multus
```

## Check the file is created?
```
ls -l /etc/cni/net.d/00-multus.conf
```
## We can also read the file.
```
cat /etc/cni/net.d/00-multus.conf | jq .
```
```
[root@master1 net.d]# cat 00-multus.conf | jq .
{
  "capabilities": {
    "bandwidth": true,
    "portMappings": true
  },
  "cniVersion": "0.3.1",
  "logLevel": "verbose",
  "logToStderr": true,
  "name": "multus-cni-network",
  "clusterNetwork": "/host/etc/cni/net.d/10-calico.conflist",
  "type": "multus-shim"
}
[root@master1 net.d]# 
```

## Check the current network details.

```
ifconfig
route -n
```

```
[root@master1 multus-cni]# ifconfig enp0s3
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.31  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::a00:27ff:fe22:8301  prefixlen 64  scopeid 0x20<link>
        inet6 2401:4900:1c53:1fd8:a00:27ff:fe22:8301  prefixlen 64  scopeid 0x0<global>
        ether 08:00:27:22:83:01  txqueuelen 1000  (Ethernet)
        RX packets 260998  bytes 299894032 (286.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 164910  bytes 37108161 (35.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@master1 multus-cni]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    100    0        0 enp0s3
172.16.14.64    192.168.1.33    255.255.255.192 UG    0      0        0 tunl0
172.16.68.0     0.0.0.0         255.255.255.192 U     0      0        0 *
172.16.68.25    0.0.0.0         255.255.255.255 UH    0      0        0 calib312017aa31
172.16.68.26    0.0.0.0         255.255.255.255 UH    0      0        0 calib12398bf93b
172.16.133.128  192.168.1.32    255.255.255.192 UG    0      0        0 tunl0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.1.0     0.0.0.0         255.255.255.0   U     100    0        0 enp0s3
[root@master1 multus-cni]# 
```

### Create a new Multus configuration.
```
cat <<EOF | kubectl create -f -
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: macvlan-conf
spec:
  config: '{
      "cniVersion": "0.3.0",
      "type": "macvlan",
      "master": "enp0s3",
      "mode": "bridge",
      "ipam": {
        "type": "host-local",
        "subnet": "192.168.1.0/24",
        "rangeStart": "192.168.1.200",
        "rangeEnd": "192.168.1.216",
        "routes": [
          { "dst": "0.0.0.0/0" }
        ],
        "gateway": "192.168.1.1"
      }
    }'
EOF
```

### Perform the Post checks.
```
kubectl get network-attachment-definitions
```
```
kubectl describe network-attachment-definitions.k8s.cni.cncf.io macvlan-conf 
```

```
[root@master1 multus-cni]# kubectl get network-attachment-definitions
NAME           AGE
macvlan-conf   11s


[root@master1 multus-cni]# kubectl describe network-attachment-definitions.k8s.cni.cncf.io macvlan-conf 
```

### Create a pod with using Annotation.

```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: samplepod
  annotations:
    k8s.v1.cni.cncf.io/networks: macvlan-conf
spec:
  containers:
  - name: samplepod
    command: ["/bin/ash", "-c", "trap : TERM INT; sleep infinity & wait"]
    image: alpine
EOF
```

### Check the post checks for PODs.
```
kubectl get pods samplepod 
```

### Login into this pod and check the new interfaces.
```
kubectl exec -it samplepod -- /bin/sh
```

### If we want to add 2 interfaces in the Pod, then use "network-attachment-definitions" name twice
```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: samplepod2
  annotations:
    k8s.v1.cni.cncf.io/networks: macvlan-conf,macvlan-conf
spec:
  containers:
  - name: samplepod
    command: ["/bin/ash", "-c", "trap : TERM INT; sleep infinity & wait"]
    image: alpine
EOF
```
### Check the pods if it is running.
```
kubectl get pods samplepod2
```

### Check the 2 interfaces in the pod
```
kubectl exec pods/samplepod2 -- ifconfig
```

### Clear the Lab. 
```
kubectl delete  pods/samplepod pods/samplepod2 network-attachment-definitions.k8s.cni.cncf.io macvlan-conf 
kubectl delete  -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/master/deployments/multus-daemonset-thick.yml
rm -rf /etc/cni/net.d/00-multus.conf
```
