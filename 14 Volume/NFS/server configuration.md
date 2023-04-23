## Commands to configure the NFS server
```
ping 192.168.1.31
mkdir /home/nfsshare
dnf -y install nfs-utils
vi /etc/idmapd.conf 
Domain = example.com
vi /etc/exports
/home/nfsshare   192.168.1.0/24(rw,no_root_squash)
systemctl enable --now rpcbind nfs-server
iptables -F
export -arv
cd /home/nfsshare/
ls -ltr
touch file.txt
systemctl enable --now rpcbind nfs-server 
firewall-cmd --add-service=nfs 
firewall-cmd --runtime-to-permanent 
```
## On Kubernetes cluster nodes execute below command
```
yum install -y nfs-utils
```
