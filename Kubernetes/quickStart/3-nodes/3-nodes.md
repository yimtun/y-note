[toc]



# verion info



| 组件   | 版本          | 内核版本          |
| ------ | ------------- | ----------------- |
| ubuntu | 20.04 desktop | 5.13.0-39-generic |
| k8s    | 1.23.2        |                   |
| docker | 20.10.14      |                   |
| helm   | v3.8.2        |                   |
| cilium | 1.10.6        |                   |





# server info



| server config | ip            | gateway      | hostname           | swap | timezone | role        |
| ------------- | ------------- | ------------ | ------------------ | ---- | -------- | ----------- |
| 4C 8G 60G     | 172.16.100.13 | 172.16.100.1 | node-172-16-100-13 | off  |          | master-role |
| 4C 8G 60G     | 172.16.100.14 |              |                    |      |          | node-role   |
| 4C 8G 60G     | 172.16.100.15 |              |                    |      |          | node-role   |





# pre  setting  all-nodes 



## set static ip



当前为dhcp 获取id  设置 为固定ip 



```
ls /etc/netplan/
01-network-manager-all.yaml
```





```
cat /etc/netplan/01-network-manager-all.yaml
```





```
network:
  ethernets:
    enp0s5:
      dhcp4: no
      addresses: [172.16.100.13/24]
      gateway4: 172.16.100.1
      nameservers:
              addresses: [172.16.100.1,114.114.114.114]
  version: 2
```



注意网卡名字  enp0s5 





## set hostname



```
hostnamectl  set-hostname node-172-16-100-13
```





## set swap



```
grep swapfile /etc/fstab 
#/swapfile                                 none            swap    sw              0       0
```



```
swapoff  -a
```



## set timezone and ntp client





```
timedatectl set-timezone Asia/Shanghai
```





```
 apt-get  update -y
 apt-get install ntp -y
 systemctl  enable --now   ntp
```





## docker-ce install



```
apt-get update -y
apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```







```
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```





```
 echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```







```
apt-get update -y
```



```
apt-get install -y docker-ce=5:20.10.14~3-0~ubuntu-focal  docker-ce-cli=5:20.10.14~3-0~ubuntu-focal  containerd.io
```







```
docker --version
Docker version 20.10.14, build a224086
```





## set cgroup for docker



### docker



```
mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```







```
systemctl enable docker
systemctl daemon-reload
systemctl restart docker
```







# install packages k8s  all-nodes 





## for  kernal options





```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```





## for  apt repo





```
apt-get install -y apt-transport-https ca-certificates curl
```

 



must  cant curl      curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg





```
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```





```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```





```
apt-get update
```





##  install kubeadm 





```
apt-get install   kubelet=1.23.2-00 kubeadm=1.23.2-00 kubectl=1.23.2-00 -y
```





```
apt-mark hold    kubelet=1.23.2-00 kubeadm=1.23.2-00 kubectl=1.23.2-00 
```









##  pre  images







```
kubeadm config images list  --kubernetes-version=v1.23.2
```



```
k8s.gcr.io/kube-apiserver:v1.23.2
k8s.gcr.io/kube-controller-manager:v1.23.2
k8s.gcr.io/kube-scheduler:v1.23.2
k8s.gcr.io/kube-proxy:v1.23.2
k8s.gcr.io/pause:3.6
k8s.gcr.io/etcd:3.5.1-0
k8s.gcr.io/coredns/coredns:v1.8.6
```



```
kubeadm config images pull  --kubernetes-version=v1.23.2
```













#   init cluster for cilium






## init cluster master-role


```
kubeadm init \
  --apiserver-advertise-address=172.16.100.13 \
  --kubernetes-version v1.23.2 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --skip-phases=addon/kube-proxy  \
  --ignore-preflight-errors=all
```









##  jon to cluster node-role



172.16.100.14

172.16.100.15



```
kubeadm join 172.16.100.13:6443 --token tnn48i.mn6hshg4prdrwavf \
        --discovery-token-ca-cert-hash sha256:4d1713a6e4587be125b1bcf62098f23242756a908e9530629e811fab52f06702
```









# install  cilium cni master-role



172.16.100.13




##   install helm



```
wget  https://get.helm.sh/helm-v3.8.2-linux-amd64.tar.gz
```



```
tar xzvf helm-v3.8.2-linux-amd64.tar.gz 
```



```
mv linux-amd64/helm  /usr/bin/
```





```
docker pull quay.io/cilium/cilium:v1.10.6
docker pull quay.io/cilium/operator-generic:v1.10.6
docker pull quay.io/cilium/hubble-relay:v1.10.6
```



```
helm repo add cilium https://helm.cilium.io/
```







安装  cilium

```
helm install cilium cilium/cilium \
  --version 1.10.6 \
  --set kubeProxyReplacement=strict \
  --namespace kube-system \
  --set k8sServiceHost=172.16.100.13 \
  --set k8sServicePort=6443
```







删除cilium 命令

```
helm   uninstall cilium -n kube-system
```













##   get cluser status



```
kubectl get componentstatus
```



```
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Healthy   ok                              
controller-manager   Healthy   ok                              
etcd-0               Healthy   {"health":"true","reason":""}   
```





```
kubectl  get node
```



```
NAME                 STATUS   ROLES                  AGE   VERSION
node-172-16-100-13   Ready    control-plane,master   60m   v1.23.2
node-172-16-100-14   Ready    <none>                 59m   v1.23.2
node-172-16-100-15   Ready    <none>                 59m   v1.23.2
```







#  verify cluster







## verify service



###  create a deploy 

```
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
EOF
```







### create a service for deploy 





```
kubectl  -n default expose deploy  nginx-deployment  --port=80 --target-port=80 --type=NodePort
```





```
kubectl  get svc nginx-deployment
NAME               TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-deployment   NodePort   10.108.43.43   <none>        80:32090/TCP   17s
```





```
curl  172.16.100.13:32090
curl  172.16.100.14:32090
curl  172.16.100.15:32090
```





## verify coredns





```
cat > pod-nginx.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80
EOF
```





```
kubectl create -f pod-nginx.yaml
```



```
kubectl exec  nginx -i -t -- /bin/bash
```





```
curl  nginx-deployment
```



ok

可以访问nginx



```
apt-get update -y
apt-get  install dnsutils -y
```







```
nslookup  nginx-deployment
```







可以解析





输出

```shell
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   nginx-deployment.default.svc.cluster.local
Address: 10.108.43.43

root@nginx:/#   
```

