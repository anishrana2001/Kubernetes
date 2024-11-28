How to install the haproxy for Kubernetes master nodes?

## Make the local DNS entries as we don't have public DNS server.
```
>/etc/hosts
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

### Disable the Firewall ########
```
systemctl stop firewalld.service
systemctl disable firewalld
systemctl status firewalld
```
## Install the Haproxy package.
```
apt-get install haproxy -y
```
### Modify the haproxy configuration file.
```
vi /etc/haproxy/haproxy.cfg
```
```
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

frontend kubernetes-frontend                        ## Line Added
    bind 0.0.0.0:6443                               ## Line Added
    option tcplog                                   ## Line Added
    mode tcp                                        ## Line Added
    default_backend kubernetes-backend              ## Line Added

backend kubernetes-backend                           ## Line Added
    mode tcp                                         ## Line Added
    balance roundrobin                               ## Line Added
    option tcp-check                                 ## Line Added
    server master1 192.168.1.30:6443 check           ## Line Added
    server master2 192.168.1.31:6443 check           ## Line Added
``` 
```
setsebool -P haproxy_connect_any=1
```

### Enable and start the haproxy
```
systemctl enable haproxy
systemctl start haproxy 
systemctl check haproxy
```

```
systemctl status haproxy.service -l --no-pager
```
#### The -l flag will ensure that systemctl outputs the entire contents of a line, instead of substituting in ellipses (…) for long lines. 
#### The --no-pager flag will output the entire log to your screen without invoking a tool like less that only shows a screen of content at a time.





