[toc]



# verion info



| 组件    | 版本          | 内核版本          |
| ------- | ------------- | ----------------- |
| ubuntu  | 20.04 desktop | 5.13.0-39-generic |
| k8s     | 1.23.5        |                   |
| docker  | 20.10.14      |                   |
| helm    | v3.8.2        |                   |
| flannel | v0.17.0       |                   |





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





## set cgroup settings



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



### kubelet 

```
mkdir -p  /var/lib/kubelet/
cat > /var/lib/kubelet/config.yaml <<EOF
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
EOF
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
apt-get install   kubelet=1.23.5-00 kubeadm=1.23.5-00 kubectl=1.23.5-00 -y
```





```
apt-mark hold    kubelet=1.23.5-00 kubeadm=1.23.5-00 kubectl=1.23.5-00 
```









##  pre  images







```
kubeadm config images list  --kubernetes-version=v1.23.5
```



```
kubeadm config images pull  --kubernetes-version=v1.23.5
```













#   init cluster






## init cluster master-role





```
kubeadm init \
--apiserver-advertise-address=172.16.100.13 \
--kubernetes-version=v1.23.5 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 \
--node-name=172.16.100.13
```





##  jon to cluster node-role



172.16.100.14

172.16.100.15



```
kubeadm join 172.16.100.13:6443 \
--node-name=172.16.100.14 \
--token ziz0jp.2q6wc4zinqygwvn9 \
        --discovery-token-ca-cert-hash sha256:a13230d6df8167b259b6b9e87e703f0780ac0eef4d0305a0931872bf36c05319 
```





```
kubeadm join 172.16.100.13:6443 \
--node-name=172.16.100.15 \
--token ziz0jp.2q6wc4zinqygwvn9 \
        --discovery-token-ca-cert-hash sha256:a13230d6df8167b259b6b9e87e703f0780ac0eef4d0305a0931872bf36c05319 
```







# install  flannel cni master-role



172.16.100.13



```
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.17.0/Documentation/kube-flannel.yml
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
NAME            STATUS   ROLES                  AGE     VERSION
172.16.100.13   Ready    control-plane,master   3m10s   v1.23.5
172.16.100.14   Ready    <none>                 109s    v1.23.5
172.16.100.15   Ready    <none>                 82s     v1.23.5
```





##  get flannel run mode









```
root@node-172-16-100-13:~# kubectl  get configmap -n kube-system kube-flannel-cfg -o yaml
apiVersion: v1
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```





```
"Type": "vxlan"
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
```



```
NAME               TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx-deployment   NodePort   10.104.180.187   <none>        80:31887/TCP   7s
```





```
curl  172.16.100.13:31887
curl  172.16.100.14:31887
curl  172.16.100.15:31887
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













# 网络信息收集












## 	网络号计算



十进制转换为二进制

```
 echo 'ibase=10;obase=2;172'|bc
```

二进制转十进制

```
 echo $[2#11000000]
```





### 172.16.100.13



```
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default 
    link/ether a2:9b:22:ca:07:7f brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::a09b:22ff:feca:77f/64 scope link 
       valid_lft forever preferred_lft forever
```





```
cat /run/flannel/subnet.env 
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
```








10.244.0.0/16 网络号

| ip 地址                 | 10.244.0.0/16 |          |          |          |      |
| ----------------------- | ------------- | -------- | -------- | -------- | ---- |
| ip 二进制               | 00001010      | 11110100 | 00000000 | 00000000 |      |
| 子网掩码二进制          | 11111111      | 11111111 | 00000000 | 00000000 |      |
| 网络号二进制 （与运算） | 00001010      | 11110100 | 00000000 | 00000000 |      |
| 网络号十进制            | 10            | 244      | 0        | 0        |      |



ip地址范围

10.244.0.0/16

10.244.255.255/16









10.244.0.1/24 网络号

| ip 地址                 | 10.244.0.1/24 |          |          |          |      |
| ----------------------- | ------------- | -------- | -------- | -------- | ---- |
| ip 二进制               | 00001010      | 11110100 | 00000000 | 00000001 |      |
| 子网掩码二进制          | 11111111      | 11111111 | 11111111 | 00000000 |      |
| 网络号二进制 （与运算） | 00001010      | 11110100 | 00000000 | 00000000 |      |
| 网络号十进制            | 10            | 244      | 0        | 0        |      |



ip地址范围

10.244.0.0/24 

10.244.0.255/24





10.244.0.0/32 网络号

| ip 地址                 | 10.244.0.0/32 |          |          |          |      |
| ----------------------- | ------------- | -------- | -------- | -------- | ---- |
| ip 二进制               | 00001010      | 11110100 | 00000000 | 00000000 |      |
| 子网掩码二进制          | 11111111      | 11111111 | 11111111 | 11111111 |      |
| 网络号二进制 （与运算） | 00001010      | 11110100 | 00000000 | 00000000 |      |
| 网络号十进制            | 10            | 244      | 0        | 0        |      |



网络号  10.244.0.0/32



ip地址范围  



没有主机位 只有一个地址

10.244.0.0/32









### 172.16.100.14



```
cat /run/flannel/subnet.env 
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.1.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
```





```
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default 
    link/ether aa:51:94:e0:2b:43 brd ff:ff:ff:ff:ff:ff
    inet 10.244.1.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::a851:94ff:fee0:2b43/64 scope link 
```







10.244.1.1/24 网络号



| ip 地址                 | 10.244.0.1/24 |          |          |          |      |
| ----------------------- | ------------- | -------- | -------- | -------- | ---- |
| ip 二进制               | 00001010      | 11110100 | 00000001 | 00000001 |      |
| 子网掩码二进制          | 11111111      | 11111111 | 11111111 | 00000000 |      |
| 网络号二进制 （与运算） | 00001010      | 11110100 | 00000001 | 00000000 |      |
| 网络号十进制            | 10            | 244      | 1        | 0        |      |



ip地址范围

10.244.1.0/24     网络号

10.244.1.255/24 广播地址

















## flannel-vxlan















```
flannel.1 是一个 vxlan 网卡
vni  是 1
vtep mac 地址不是通过多播学习的 通过watch api-server 发现的
```







在 172.16.100.13 查看信息





```
1 创建 vxlan 设备  flannel.1

2 并向etcd报告VTEP设备的信息。当flannel网络上的一个新节点加入集群并向etcd注册时，每个节点上的flanneld从etcd学习通知并执行以下过程：



   2.1 创建路由 将pod 网络路由到 flanel.1


route  -n | grep  flannel.1
10.244.1.0      10.244.1.0      255.255.255.0   UG    0      0        0 flannel.1
10.244.2.0      10.244.2.0      255.255.255.0   UG    0      0        0 flannel.1

   2.2. 将love节点的IP和VTEP设备的静态ARP缓存添加到该节点
   
   
   arp -n | grep flannel.1
10.244.2.0               ether   22:7e:a5:fd:66:f2   CM                    flannel.1
10.244.1.0               ether   56:99:3f:dd:ef:71   CM                    flannel.1


 2.3 VXLAN转发表fdb
 
 
 bridge  fdb | grep flannel.1
ca:9c:12:6c:3a:05 dev flannel.1 dst 172.16.100.15 self permanent
56:99:3f:dd:ef:71 dev flannel.1 dst 172.16.100.14 self permanent
22:7e:a5:fd:66:f2 dev flannel.1 dst 172.16.100.15 self permanent

容器mac                              对方VTEP的外部IP

```





# 通信流向







172.16.100.14  pod  A



```
10.244.1.5


route  -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.244.1.1      0.0.0.0         UG    0      0        0 eth0
10.244.0.0      10.244.1.1      255.255.0.0     UG    0      0        0 eth0
10.244.1.0      0.0.0.0         255.255.255.0   U     0      0        0 eth0
root@nginx-deployment-9456bbbf9-dr96z:/# 
```





172.16.100.15. pod  B



```
10.244.2.7

route  -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.244.2.1      0.0.0.0         UG    0      0        0 eth0
10.244.0.0      10.244.2.1      255.255.0.0     UG    0      0        0 eth0
10.244.2.0      0.0.0.0         255.255.255.0   U     0      0        0 eth0
root@nginx-deployment-9456bbbf9-d55xl:/# 
```





pod A ping pod B



```
root@nginx-deployment-9456bbbf9-dr96z:/# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.244.1.5  netmask 255.255.255.0  broadcast 10.244.1.255
        ether ce:dc:4f:1a:cc:64  txqueuelen 0  (Ethernet)
        RX packets 8772  bytes 9269544 (8.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4440  bytes 243010 (237.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

root@nginx-deployment-9456bbbf9-dr96z:/# ping 10.244.2.7
PING 10.244.2.7 (10.244.2.7) 56(84) bytes of data.
64 bytes from 10.244.2.7: icmp_seq=1 ttl=62 time=1.57 ms
64 bytes from 10.244.2.7: icmp_seq=2 ttl=62 time=1.88 ms
64 bytes from 10.244.2.7: icmp_seq=3 ttl=62 time=1.84 ms
64 bytes from 10.244.2.7: icmp_seq=4 ttl=62 time=1.76 ms
64 bytes from 10.244.2.7: icmp_seq=5 ttl=62 time=1.42 ms
^C
```







ping 10.244.2.7 走的哪个路由？



```
容器A中的IP数据包通过容器A的路由表发送到cni0  cni0 的ip是容器的默认网关


root@nginx-deployment-9456bbbf9-dr96z:/# route  -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.244.1.1      0.0.0.0         UG    0      0        0 eth0
10.244.0.0      10.244.1.1      255.255.0.0     UG    0      0        0 eth0
10.244.1.0      0.0.0.0         255.255.255.0   U     0      0        0 eth0




root@node-172-16-100-14:~# ip a| grep cni0  
5: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    inet 10.244.1.1/24 brd 10.244.1.255 scope global cni0
     

```









```
到达cni0的数据包通过主机A中的路由表   发送到10.244.2.7 的IP数据包应发送到flannel.1接口



route -n | grep flannel.1
10.244.0.0      10.244.0.0      255.255.255.0   UG    0      0        0 flannel.1
10.244.2.0      10.244.2.0      255.255.255.0   UG    0      0        0 flannel.1

```











```
作为VTEP设备，flannel.1  依据 VTEP配置封包

1.我们通过apiserver(--kube-subnet-mgr) 知道10.244.2.7 属于节点172.16.100.15 ，我们知道节点B的IP地址是 172.16.100.15
```





```
kubectl get no -o json | jq '[.items[] | {"name": .metadata.name, "podCIDR": .spec.podCIDR, "podCIDRs": .spec.podCIDRs, "InternalIP": (.status.addresses[] | select(.type == "InternalIP") | .address)}]'
```





```
[
  {
    "name": "172.16.100.13",
    "podCIDR": "10.244.0.0/24",
    "podCIDRs": [
      "10.244.0.0/24"
    ],
    "InternalIP": "172.16.100.13"
  },
  {
    "name": "172.16.100.14",
    "podCIDR": "10.244.1.0/24",
    "podCIDRs": [
      "10.244.1.0/24"
    ],
    "InternalIP": "172.16.100.14"
  },
  {
    "name": "172.16.100.15",
    "podCIDR": "10.244.2.0/24",
    "podCIDRs": [
      "10.244.2.0/24"
    ],
    "InternalIP": "172.16.100.15"
  }
]
```







```
然后通过节点A中的转发表，可以知道节点 172.16.100.15 的VTEP对应的Mac，

并根据flanne连接时设置的参数 （VNI、本地IP、端口）          发送 VXLAN包 
发送到 172.16.100.15 udp 8472
```





```
bridge  fdb  | grep flannel.1 | grep 172.16.100.15
ca:9c:12:6c:3a:05 dev flannel.1 dst 172.16.100.15 self permanent
22:7e:a5:fd:66:f2 dev flannel.1 dst 172.16.100.15 self permanent
root@node-172-16-100-14:~# 
```





```
arp -n | grep  flannel.1
10.244.2.0               ether   22:7e:a5:fd:66:f2   CM                    flannel.1
10.244.0.0               ether   0e:92:7c:ff:68:1e   CM                    flannel.1
```



172.16.100.15 对应的mac   22:7e:a5:fd:66:f2



```
root@node-172-16-100-15:~# ip a | grep '22:7e:a5:fd:66:f2' -B 1
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default 
    link/ether 22:7e:a5:fd:66:f2 brd ff:ff:ff:ff:ff:ff
root@node-172-16-100-15:~# 
```





















```
ETCDCTL_API=3 etcdctl --endpoints=https://172.16.100.13:2379 --cacert=./ca.crt --cert=./healthcheck-client.crt  --key=./healthcheck-client.key  endpoint health
```



```
ETCDCTL_API=3 etcdctl --endpoints=https://172.16.100.13:2379 --cacert=./ca.crt --cert=./healthcheck-client.crt  --key=./healthcheck-client.key  get --prefix /flannel
```



```
ETCDCTL_API=3 etcdctl --endpoints=https://172.16.100.13:2379 --cacert=./ca.crt --cert=./healthcheck-client.crt  --key=./healthcheck-client.key  get "" --prefix  --key-only | grep -Ev "^$" | grep "flannel"
```





```
etcdctl get "" --prefix=true --keys-only
```





```
xx/network/subnets 
```



--------





```
通过主机A和主机B之间的网络连接，VXLAN数据包到达节点B的网卡
```





```
通过端口8472，VXLAN数据包被转发到VTEP设备flannel 并解包
```





```
解包后的IP包与节点B 172.16.100.15 中的路由表（10.244.2.0）匹配，内核将IP包转发给cni0。
```





```
cni0将IP数据包转发到连接到cni0的容器B
```







# 特点



```
在VXLAN模式下，数据由内核转发，flannel不转发数据，只动态设置ARP和FDB表项
```







# good 



https://blog.birost.com/a?ID=01650-6c7c6222-740c-48de-ae2a-8d03033731a6#2VXLAN_83





https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking#











# extra



```
kubeadm init  \
--apiserver-advertise-address=172.16.100.13 \
--kubernetes-version=v1.23.5 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 \
--node-name=172.16.100.13 \
```









```
kubeadm config print init-defaults
kubeadm config print init-defaults 
```





```
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  imagePullPolicy: IfNotPresent
  name: node
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: 1.23.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```



```
kubectl get configMap kubeadm-config -o yaml --namespace=kube-system
```



```
apiVersion: v1
data:
  ClusterConfiguration: |
    apiServer:
      extraArgs:
        authorization-mode: Node,RBAC
      timeoutForControlPlane: 4m0s
    apiVersion: kubeadm.k8s.io/v1beta3
    certificatesDir: /etc/kubernetes/pki
    clusterName: kubernetes
    controllerManager: {}
    dns: {}
    etcd:
      local:
        dataDir: /var/lib/etcd
    imageRepository: k8s.gcr.io
    kind: ClusterConfiguration
    kubernetesVersion: v1.23.5
    networking:
      dnsDomain: cluster.local
      podSubnet: 10.244.0.0/16
      serviceSubnet: 10.96.0.0/12
    scheduler: {}
kind: ConfigMap
metadata:
  creationTimestamp: "2022-06-09T02:33:48Z"
  name: kubeadm-config
  namespace: kube-system
  resourceVersion: "211"
  uid: 1e0f3e01-3759-4dd0-8e32-2962bf342089
```



```
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
```







#  修改ip vs



```
apt-get  install ipvsadm
apt install ipset
```



```
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
```



```
kubectl edit configmap kube-proxy -n kube-system
```



```
 42     kind: KubeProxyConfiguration
 43     metricsBindAddress: ""
 44     mode: "ipvs"
```



```
```





```
 kubectl  delete  pod  -n kube-system  -l k8s-app=kube-proxy
```







```
iptables -t filter  -F
iptables -t nat  -F
iptables -t mangle -F
```







# ipset



```
root@node-172-16-100-13:~#  kubectl  -n kube-system exec -it  kube-proxy-wj779  --  ipset list
Name: KUBE-LOAD-BALANCER-LOCAL
Type: hash:ip,port
Revision: 6
Header: family inet hashsize 1024 maxelem 65536 bucketsize 12 initval 0xd4dc3883
Size in memory: 200
References: 0
Number of entries: 0
Members:

Name: KUBE-LOAD-BALANCER-SOURCE-IP
Type: hash:ip,port,ip
Revision: 6
Header: family inet hashsize 1024 maxelem 65536 bucketsize 12 initval 0x16a149d9
Size in memory: 208
References: 0
Number of entries: 0
Members:

Name: KUBE-NODE-PORT-SCTP-HASH
Type: hash:ip,port
Revision: 6
Header: family inet hashsize 1024 maxelem 65536 bucketsize 12 initval 0x5d488e34
Size in memory: 200
References: 0
Number of entries: 0
Members:

Name: KUBE-CLUSTER-IP
Type: hash:ip,port
Revision: 6
Header: family inet hashsize 1024 maxelem 65536 bucketsize 12 initval 0x0c31ac7a
Size in memory: 392
References: 2
Number of entries: 4
Members:
10.96.0.10,tcp:9153
10.96.0.10,udp:53
10.96.0.10,tcp:53
10.96.0.1,tcp:443

Name: KUBE-EXTERNAL-IP-LOCAL
Type: hash:ip,port
Revision: 6
Header: family inet hashsize 1024 maxelem 65536 bucketsize 12 initval 0xf453ecec
Size in memory: 200
References: 0
Number of entries: 0
Members:

Name: KUBE-LOAD-BALANCER
Type: hash:ip,port
Revision: 6
Header: family inet hashsize 1024 maxelem 65536 bucketsize 12 initval 0x4193d887
Size in memory: 200
References: 0
Number of entries: 0
Members:

Name: KUBE-NODE-PORT-LOCAL-SCTP-HASH
Type: hash:ip,port
Revision: 6
Header: family inet hashsize 1024 maxelem 65536 bucketsize 12 initval 0x45d891a0
Size in memory: 200
References: 0
Number of entries: 0
Members:

Name: KUBE-LOAD-BALANCER-FW
Type: hash:ip,port
Revision: 6
Header: family inet hashsize 1024 maxelem 65536 bucketsize 12 initval 0x5de13b56
Size in memory: 200
References: 0
Number of entries: 0
Members:

Name: KUBE-LOAD-BALANCER-SOURCE-CIDR
Type: hash:ip,port,net
Revision: 8
Header: family inet hashsize 1024 maxelem 65536 bucketsize 12 initval 0xb44e76e7
Size in memory: 464
References: 0
Number of entries: 0
Members:

Name: KUBE-NODE-PORT-TCP
Type: bitmap:port
Revision: 3
Header: range 0-65535
Size in memory: 8264
References: 0
Number of entries: 0
Members:

Name: KUBE-NODE-PORT-UDP
Type: bitmap:port
Revision: 3
Header: range 0-65535
Size in memory: 8264
References: 0
Number of entries: 0
Members:

Name: KUBE-HEALTH-CHECK-NODE-PORT
Type: bitmap:port
Revision: 3
Header: range 0-65535
Size in memory: 8264
References: 1
Number of entries: 0
Members:

Name: KUBE-LOOP-BACK
Type: hash:ip,port,ip
Revision: 6
Header: family inet hashsize 1024 maxelem 65536 bucketsize 12 initval 0x1c793107
Size in memory: 208
References: 0
Number of entries: 0
Members:

Name: KUBE-EXTERNAL-IP
Type: hash:ip,port
Revision: 6
Header: family inet hashsize 1024 maxelem 65536 bucketsize 12 initval 0x13b8c57f
Size in memory: 200
References: 0
Number of entries: 0
Members:

Name: KUBE-NODE-PORT-LOCAL-TCP
Type: bitmap:port
Revision: 3
Header: range 0-65535
Size in memory: 8264
References: 0
Number of entries: 0
Members:

Name: KUBE-NODE-PORT-LOCAL-UDP
Type: bitmap:port
Revision: 3
Header: range 0-65535
Size in memory: 8264
References: 0
Number of entries: 0
Members:

Name: KUBE-6-HEALTH-CHECK-NODE-PORT
Type: bitmap:port
Revision: 3
Header: range 0-65535
Size in memory: 8264
References: 1
Number of entries: 0
Members:

Name: KUBE-6-EXTERNAL-IP
Type: hash:ip,port
Revision: 6
Header: family inet6 hashsize 1024 maxelem 65536 bucketsize 12 initval 0x40c168d6
Size in memory: 216
References: 0
Number of entries: 0
Members:

Name: KUBE-6-NODE-PORT-LOCAL-UDP
Type: bitmap:port
Revision: 3
Header: range 0-65535
Size in memory: 8264
References: 0
Number of entries: 0
Members:

Name: KUBE-6-LOAD-BALANCER-SOURCE-IP
Type: hash:ip,port,ip
Revision: 6
Header: family inet6 hashsize 1024 maxelem 65536 bucketsize 12 initval 0xc1c014b9
Size in memory: 232
References: 0
Number of entries: 0
Members:

Name: KUBE-6-NODE-PORT-SCTP-HASH
Type: hash:ip,port
Revision: 6
Header: family inet6 hashsize 1024 maxelem 65536 bucketsize 12 initval 0x105cb376
Size in memory: 216
References: 0
Number of entries: 0
Members:

Name: KUBE-6-NODE-PORT-LOCAL-SCTP-HAS
Type: hash:ip,port
Revision: 6
Header: family inet6 hashsize 1024 maxelem 65536 bucketsize 12 initval 0xded2c072
Size in memory: 216
References: 0
Number of entries: 0
Members:

Name: KUBE-6-CLUSTER-IP
Type: hash:ip,port
Revision: 6
Header: family inet6 hashsize 1024 maxelem 65536 bucketsize 12 initval 0x93e4894f
Size in memory: 216
References: 0
Number of entries: 0
Members:

Name: KUBE-6-LOAD-BALANCER
Type: hash:ip,port
Revision: 6
Header: family inet6 hashsize 1024 maxelem 65536 bucketsize 12 initval 0x5c1e741d
Size in memory: 216
References: 0
Number of entries: 0
Members:

Name: KUBE-6-LOAD-BALANCER-FW
Type: hash:ip,port
Revision: 6
Header: family inet6 hashsize 1024 maxelem 65536 bucketsize 12 initval 0x1be0ae58
Size in memory: 216
References: 0
Number of entries: 0
Members:

Name: KUBE-6-NODE-PORT-TCP
Type: bitmap:port
Revision: 3
Header: range 0-65535
Size in memory: 8264
References: 0
Number of entries: 0
Members:

Name: KUBE-6-NODE-PORT-LOCAL-TCP
Type: bitmap:port
Revision: 3
Header: range 0-65535
Size in memory: 8264
References: 0
Number of entries: 0
Members:

Name: KUBE-6-LOOP-BACK
Type: hash:ip,port,ip
Revision: 6
Header: family inet6 hashsize 1024 maxelem 65536 bucketsize 12 initval 0xed523b6a
Size in memory: 232
References: 0
Number of entries: 0
Members:

Name: KUBE-6-EXTERNAL-IP-LOCAL
Type: hash:ip,port
Revision: 6
Header: family inet6 hashsize 1024 maxelem 65536 bucketsize 12 initval 0x3dc10ff5
Size in memory: 216
References: 0
Number of entries: 0
Members:

Name: KUBE-6-NODE-PORT-UDP
Type: bitmap:port
Revision: 3
Header: range 0-65535
Size in memory: 8264
References: 0
Number of entries: 0
Members:

Name: KUBE-6-LOAD-BALANCER-LOCAL
Type: hash:ip,port
Revision: 6
Header: family inet6 hashsize 1024 maxelem 65536 bucketsize 12 initval 0x0b16a2f0
Size in memory: 216
References: 0
Number of entries: 0
Members:

Name: KUBE-6-LOAD-BALANCER-SOURCE-CID
Type: hash:ip,port,net
Revision: 8
Header: family inet6 hashsize 1024 maxelem 65536 bucketsize 12 initval 0xcb6848f3
Size in memory: 1256
References: 0
Number of entries: 0
Members:
root@node-172-16-100-13:~# 
```

