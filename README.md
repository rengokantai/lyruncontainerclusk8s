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
master
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
(ubuntu: /etc/apt/sources.list.d)  

