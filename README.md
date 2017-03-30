# lyruncontainerclusk8s
## 2
### 01. Packages and Dependencies
centos7 create 1 master and 2 minions.  
All machine:
```
yum install -y ntp
systemctl enable ntpd && systemctl start ntpd
```

then add hosts to all machines.

#### 05:20
all 3 machines (hostname=a b c) a is master
```
vi /etc/yum.repos.d/virt7-docker-common-release.repo
```
edit
```
[virt7-docker-common-release]
name=virt7-docker-common-release
baseurl=http://cbs.centos.org/repos/virt7-docker-common-release/x86_64/os/
gpgcheck=0
```
```
yum update -y
```
(ubuntu: /etc/apt/sources.list.d)  

####08:00 check status
```
systemctl status iptables
systemctl status firewalld
```

#### 10:06
master only
```
yum install -y --enablerepo=virt7-docker-common-release kubernetes docker
```

### 02. Install and Configure Master Controller
master
```
vi /etc/kubernetes/config
```
edit
```
KUBE_MASTER="--master=http://a:8080"
```
add this line
```
KUBE_ETCD_SERVERS="--etcd-servers=http://a:2379"
```
