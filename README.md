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

#### 08:00 check status
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

### 02. Running Containers
#### 14:44
```
gem list --local
```
### 03. The Dockerfile
```
docker exec -u 0 -it sleepy_allen /bin/bash
```
set user id=0

### 04. Managing Ports with Container Deployments

## 4. Pods, Tags and Services
### 01. Create and Deploy Pod Definitions
master
```
vi nginx.yml
```
edit
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```
```
kubectl get pods
```
run pods from yaml(this command create a pod on any one machine)
```
kubectl create -f nginx.yml
```
```
kubectl describe pod nginx
```
```
kubectl run busybox --image=busybox --restart=Never --tty -i --generator=run-pod/v1
```
```
kubectl delete pod busybox
```
assign port
```
kubectl port-forward nginx :80   //assign random port
kubectl port-forward nginx 8888:80
```


### 02. Tags, Labels and Selectors
```
cp nginx.yml nginx-pod-label.yml
vi nginx-pod-label.yml
```
edit

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

get label
```
kubectl get pods -l app=nginx
```

### 03. Deployment State
```
cp nginx-pod-label.yml  nginx-deployment-prod.yml
vi nginx-deployment-prod.yml
```
edit
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment-prod
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-deployment-prod
    spec:
      containers:
      - name: nginx-deployment-prod
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

run
```
kubectl create -f nginx-deployment-prod.yml
```
```
kubectl get pods
kubectl get deployments
```

create a dev yaml file
```
cp nginx-deployment-prod.yml  nginx-deployment-dev.yml
```


#### 12:50
```
kubectl apply -f nginx-deployment-dev-update.yaml
```
### 04. Multi-Pod (Container) Replication Controller
create file
```
nginx-multi-label.yml
```
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-www
spec:
  replicas: 2
  selector:
    app: nginx
  template:
    metadata:
      name: naginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
```
kubectl describe pods
```
#### 09:30
```
kubectl get services
```
[delete all pods](https://coderwall.com/p/xat1gw/delete-all-pods-in-kubernetes)
```
kubectl delete pods --all
```
But the pods will go back after several seconds.  
in this scenario we need to delete controllers
```
kubectl get replicationcontrollers
kubectl delete reaplicationcontroller nginx-www
```
all minion machinels
```
systemctl stop kubelet kube-proxy
```
### 05. Create and Deploy Service Definitions
```
kubectl get replicationcontrollers
kubectl get pods
kubectl get nodes
```

#### 06:04
create a service
```
vi nginx-service.yml
```
then edit
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  ports:
  - port: 8000
    targetPort: 80
    protocol: TCP
  selector:
    app: nginx
```
list services
```
kubectl get services
```
#### 11:04
```
kubectl run busybox --generator=run-pod/v1 --image=busybox --restart=Never --tty -i
```
```
kubectl delete service nginx-service
```

## 05. Logs, Scaling and Recovery
### 01. Creating Temporary Pods at the Command line
review nginx.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

```
kubectl run mysample --image=latest123/apache
```
#### 04:25
```
kubectl get deployments
```
##### 06:15
```
kubectl delete deployment mysample
```
delete all
```
kubectl delete deployments --all
```
### 02. Interacting with Pod Containers
```
vi myapache.yml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: myapache
spec:
  containers:
  - name: myapache
    image: latest123/apache
    ports:
    - containerPort: 80
```
```
kubectl create -f myapache.yml
```
#### 04:00
```
kubectl exec myapache date
```
in lynx,
```
xterm
```
to enter terminal page
### 03. Logs
### 04. Autoscaling and Scaling our Pods
review nginx-multi-label.yml
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-www
spec:
  replicas: 2
  selector:
    app: nginx
  template:
    metadata:
      name: naginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
### 05. Failure and Recovery

