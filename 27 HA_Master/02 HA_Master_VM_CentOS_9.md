
### Update the file "/etc/hosts/" on all nodes.
```
cat <<EOF>>  /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.30  master2.example.com      master2
192.168.1.31  master1.example.com      master1
192.168.1.32  workernode1.example.com  workernode1
192.168.1.33  workernode2.example.com  workernode2
192.168.1.34  loadbalancer.example.com loadbalancer
EOF
```

### Login into first master node "master1: 192.168.1.31"
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
EOF
```


### Checking the requirement for Kubernetes.
```
cat > /tmp/script.sh
```
```
echo -e "\033[44m###Kubernetes installation Preparation ###\033[m"
RAM=`cat /proc/meminfo | grep MemTotal | awk '{print ($2 / 1024) / 1024 ,"GiB"}'`
CPU=`cat /proc/cpuinfo | grep processor`

sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
swapoff -a
echo -e "\033[42mYour system RAM is $RAM\033[m"
echo -e "\033[42mYour system CPUs are
$CPU\033[m"
sleep 10

echo -e "\033[44m###Disable SELINUX ###\033[m"

cat /etc/sysconfig/selinux | grep SELINUX=
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i --follow-symlinks 's/SELINUX=permissive/SELINUX=disabled/g' /etc/sysconfig/selinux
cat /etc/sysconfig/selinux | grep SELINUX=
setenforce 0


echo -e "\033[44m###Make DNS local entries - Change it as per your requirement ###\033[m"

echo -e "\033[44m###Check the connectivity of your cluster nodes ###\033[m"
ping -c 2 master1
ping -c 2 workernode1
ping -c 2 workernode2


sleep 10 

echo -e "\033[44m###Disable the Firewall ###\033[m"
systemctl stop firewalld.service
systemctl disable firewalld
systemctl status firewalld


sleep 10 
echo -e "\033[44m###Preparation for Docker installation ###\033[m"
modprobe br_netfilter
lsmod | grep br_netfilter
modprobe overlay
echo "1"  > /proc/sys/net/bridge/bridge-nf-call-iptables
echo "1"  > /proc/sys/net/ipv6/conf/default/forwarding
sysctl -a | grep net.bridge.bridge-nf-call-iptables
echo ######################


sleep 10 
echo -e "\033[44m###Docker installation steps ###\033[m"
echo -e "\033[42mRemoving the old runtime engine such as Docker and Buildah ###\033[m"
echo -e "\033[42mInstalling the Docker engine\033[m"
sudo yum -y remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine buildah
echo -e "\033[42mInstall the yum-utils package\033[m"
yum install -y yum-utils
echo -e "\033[42mAdd repo\033[m"
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
echo -e "\033[42mInstall Docker packages :docker-ce docker-ce-cli containerd.io docker-compose-plugin\033[m"
yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
containerd config default | sudo tee /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
echo -e "\033[42mRestarting the docker service and enable it\033[m"
systemctl start docker
systemctl enable docker
docker run hello-world
echo ######################

sleep 10
echo -e "\033[44m###Kubernetes installation steps ###\033[m"
echo -e "\033[42mCreating a Kubernetes repo file\033[m"


echo -e "\033[42mInstalling the Kubernetes packages : kubelet kubeadm kubectl\033[m"
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
echo -e "\033[42mEnable and start the Kubelet service\033[m"
systemctl enable kubelet
systemctl start kubelet
echo #####################


swapoff -a
echo -e "\033[42mAdding in the kubernetes master\033[m"
```

```
ip a
echo ""
ip route
```

```
ip route add default via 192.168.1.0 dev enp0s3
```
### Create the cluster.
```
kubeadm init --control-plane-endpoint "loadbalancer:6443" --upload-certs --pod-network-cidr=10.244.0.0/16
```
### 

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### You can now join any number of the control-plane node running the following command on each as root:
### Check the output.
### 
### 
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
### Check the status of node.
```
kubectl get nodes
```

### Check the pods in kube-system namespace
```
kubectl get pods -n kube-system
```
## How to install the ETCD packages, so that we can check at the end for the status of etcd cluster.

```
etcdctl version
```
```
curl -s https://api.github.com/repos/etcd-io/etcd/releases/latest | grep browser_download_url  | grep linux-amd64  | cut -d '"' -f 4  | wget -qi -
```
```
tar xvf etcd-v*.tar.gz 
```
```
cd etcd-*/
```
```
sudo mv etcd* /usr/local/bin/ 
```
```
cd ~
```
```
 export PATH=/usr/local/bin:$PATH
```
```
etcdctl version
```
### Check the members.
```
 etcdctl --endpoints 192.168.1.31:2379   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key   --cacert=/etc/kubernetes/pki/etcd/ca.crt   member list
```
### we can also check, who is leader. This command will inform us more information at the end.
```
etcdctl --endpoints 192.168.1.31:2379   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key   --cacert=/etc/kubernetes/pki/etcd/ca.crt   endpoint status --cluster --write-out=table
```

### Once all are good. Switch to 2nd Master node.
### Login into first master node "master2: 192.168.1.30"

### Execute the below command and then run the script.
```
scp root@192.168.1.31:/tmp/script.sh /tmp/
```
```
scp /tmp/script.sh root@192.168.1.32:/tmp
```
```
scp /tmp/script.sh root@192.168.1.33:/tmp
```

Once all the nodes join the cluster, then execute the following command on both master nodes.
```
kubectl get nodes
```

### Now, observe the etcd pods by executing this command.
```
kubectl get pods -n kube-system
```

### Check the members.
```
 etcdctl --endpoints 192.168.1.31:2379   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key   --cacert=/etc/kubernetes/pki/etcd/ca.crt   member list
```
### Now, check the leader.
```
etcdctl --endpoints 192.168.1.31:2379   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key   --cacert=/etc/kubernetes/pki/etcd/ca.crt   endpoint status --cluster --write-out=table
```
