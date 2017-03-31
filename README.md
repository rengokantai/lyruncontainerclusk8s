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
all
```
yum install -y --enablerepo=virt7-docker-common-release kubernetes docker && yum install -y etcd
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
then
```
vi /etc/etcd/etcd.conf
```
edit
```
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379" -> ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379" -> ETCD_ADVERTISE_CLIENT_URLS="http://0.0.0.0:2379"
```

#### 06:19
```
vi /etc/kubernetes/apiserver
```
edit
```
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0" -> 
```
to
```
# The address on the local server to listen to.
KUBE_API_ADDRESS="--address=0.0.0.0"

# The port on the local server to listen on.  (uncomment)
KUBE_API_PORT="--port=8080"

# Port minions listen on  (uncomment)
KUBELET_PORT="--kubelet-port=10250"
```
and commment out this
```
# KUBE_ADMISSION_CONTROL
```
start on master
```
systemctl enable etcd kube-apiserver kube-controller-manager kube-scheduler && systemctl start etcd kube-apiserver kube-controller-manager kube-scheduler
```
check status, should return 4
```
systemctl status etcd kube-apiserver kube-controller-manager kube-scheduler | grep "(running)" | wc -l
```

### 03. Install and Configure the Minions
minions
```
vi /etc/kubernetes/config
```

edit
```
KUBE_MASTER="--master=http://a:8080"
KUBE_ETCD_SERVERS="--etcd-servers=http://a:2379"
```

then
```
vi /etc/kubernetes/kubelet
```
edit (for machine b)
```
# kubernetes kubelet (minion) config

# The address for the info server to serve on (set to 0.0.0.0 or "" for all inte
rfaces)
KUBELET_ADDRESS="--address=0.0.0.0"

# The port for the info server to serve on
KUBELET_PORT="--port=10250"

# You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname-override=b"

# location of the api-server
KUBELET_API_SERVER="--api-servers=http://a:8080"

# pod infrastructure container
# KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redha
t.com/rhel7/pod-infrastructure:latest"
```
machine b and c
```
systemctl enable kube-proxy kubelet docker && systemctl start kube-proxy kubelet docker
systemctl status kube-proxy kubelet docker | grep "(running)" | wc -l
```

### 04. Kubectl - Exploring our Environment
master
```
kubectl get nodes
kubectl describe nodes
```
#### 05:19
```
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.typs=="ExternalIP")].address}'
```
```
kubectl get nodes -o jsonpath='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}'|tr ';' "\n" | grep "Ready=True"
```






##3 Docker Fundamentals
### 01. Pulling an Image
```
cd /var/lib/docker
```
menial1
```
docker pull ubuntu:xenial
```
```
docker run -i -t ubuntu:xenial /bin/bash
```
-i interactive mode -t attach to tty

#### 14:04
```
exit
docker restart name
docker attach name
```
