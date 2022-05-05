[toc]











[toc]



## 00 

overheads  开销



## 01 2022-04-15

cilium

calico

flannel

ensp 

明细路由

BGP

OSPF

xdp

dpdk

ipvlan



## 02 2022-04-18







## 03 2022-04-19



###  k8s1.23.5







## 04 2022-04-20

网络基础





## 05 2022-04-22





cni

ipvlan cni





overlay 隧道

underlay  





ipvlan 没有lay

macla 没有lay

SR-IOV  x710

dpdk   pmd

vpp  



docker host gateway

```
docker network create -d bridge --subnet 172.19.0.0/16 br19

docker run --name d01 --network br19 -td xxximag


route add -net 172.19.0.0/16 gw 192.168.2.61

两边都加


不做snat 怎么实现
```



docker vxlan





```
直连路由
tcpdump -pne  -i eth0
-p 关闭混杂模式
```



## 06 2022-04-25-pass





```
cilium  v1.10

https://docs.cilium.io/en/v1.10/gettingstarted/




CONFIG_BPF=y
CONFIG_BPF_SYSCALL=y
CONFIG_NET_CLS_BPF=y
CONFIG_BPF_JIT=y
CONFIG_NET_CLS_ACT=y
CONFIG_NET_SCH_INGRESS=y
CONFIG_CRYPTO_SHA1=y
CONFIG_CRYPTO_USER_API_HASH=y
CONFIG_CGROUPS=y
CONFIG_CGROUP_BPF=y




安装最新版   不使用 cilium 命令行 方法安装

curl -L --remote-name-all \
https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz{,.sha256sum}

sha256sum --check cilium-linux-amd64.tar.gz.sha256sum

sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin

rm cilium-linux-amd64.tar.gz{,.sha256sum}

cilium status
cilium versin
cilium install

cilium status

get cilium configmap  配置存放位置


cilium uninstall

```







```
heml template

heml install --dry-run  --set name=cilium cilium ./tplate_dir > cilium.yml




heml add 
helm repo add
helm  repo update





Default Configuration:

Datapath	IPAM	Datastore
Encapsulation	Cluster Pool	Kubernetes CRD





https://docs.cilium.io/en/v1.10/gettingstarted/kubeproxy-free/#kubeproxy-free
```







```
https://docs.cilium.io/en/v1.10/gettingstarted/k8s-install-kubeadm/


```







```
helm template cilium cilium/cilium \
--version 1.10.6 \
--namespace kube-system \
--set kubeProxyReplacement=strict\
--set k8sServiceHost=172.16.100.201\
--set k8sServicePort=6443 > cilium_1.10.6yaml
```



```
vim cilium_1.10.6yaml

debug true
debug-verbose: "datapath"


monitor-aggregaton: none

Encapsulation mode for communication between nodes
-disabled
-vxlan (default)
--geneve



```





```
debug true
debug-verbose: "datapath"
```





```
kubectl apply -f  cilium_1.10.6yaml
```





```
kubectl -n kube-system exec -it cilium-xxx -- bash

cilium status
KubeProxyReplacement: strict
Host Routing：  Legacy？？
Masquerading: IPTables ??
```





```
kubectl -n kube-system  logs  pod  | grep Falling 


requires enable-bpf-masquerade 
Falling back to enable-host-legacy-routing-true

```







```
vim cilium_1.10.6yaml


enable-bpf-masquerade: "true"

delete
kubectl apply -f xxx.yaml

kubectl -n kube-system logs cilium-xxx | grep subsys=daemon



cilium status
Host Routing BPF
Masquerading: BPF [ens33]

创建一个 deploy 
暴露 service 
检查 没有对应的iptables 就算ok 了

kube expose deploy 

iptables-save | grep svc_name 






```









extra



```
cilium status 

IPAM 3/254.  (可用地址1-254.  255广播)
Cluster Pods  cilium 接管的pod


cat /etc/cni/net.d/05-cilium.conf

type: cilium-cni
```





```
https://docs.cilium.io/en/v1.10/gettingstarted/cni-chaining/
```







## 07 2022-04-27-pass





```
vxlan

mac in udp


ip mac


二层 mac
三层 ip
```









vxlan



https://support.huawei.com/enterprise/zh/doc/EDOC1100087027





```
stp
逻辑上阻塞 某个端口
stp 导致带宽利用率 降低
```



```
mstp
vrrp
```





vrrp+smtp



http://www.h3c.com/cn/d_201305/785647_30003_0.htm







```
ecmp 协议
vm  +  L2 +  leaf + L3 + spine (去掉汇聚层 不再使用二层的防环机制)
```







```
linux vxlan  点对点

两个服务器 L3 即可


vni



ip link add vxlan0 type vxlan id 5 dstport 4789 remote  192.168.2.62 local 192.168.2.61 dev ens33

ip -d link show vxlan0

vni id 5

4789  端口是udp 端口


ip addr add 20.1.1.2/24 dev vxlan0
ip link set vxlan0 up




测试
ping -I x.x.x.x x.x.x.x

tcpdump -pne -i ens33 icmp
tcpdump -vvv -pne -i ens33 icmp


p 关闭混杂模式  否则除了这两个服务器的其他服务器上 也能抓到包



清理arp

arp -d 10.0.1.1




```







```
vxlan group 多播组

没有local remote 


ip link add vxlan0 type vxlan id 6 dstport 4789 group 239.1.1.1 dev ens33

ip addr add 10.20.1.3/24 dev vxlan0

ip link set vxlan0 up

4789 端口到底是谁的


```







```
bgp 边界网关协议
bgp  rr    bgp 路由反射器
vtep 

bgp evpn peer 解决 full mesh 问题





vtep  把自己的信息 告诉 bgp rr 
vtep 从 bgp rr 获取其他vtep的信息





full mesh 全链接
每个vtep都连路由器

```









evpn



https://support.huawei.com/enterprise/zh/doc/EDOC1100164807?idPath=24030814







https://arthurchiao.art/blog/spine-leaf-design-zh/













```
cilium


1.11
ipv4-native-routing-cidr: x.x.x.x/y
如果不设置会做snat


1.10
native-routing-cidr: x.x.x.x/y



进入cilium 容器 查看某个 vni id


cilium endpoint list

cilium identity list


lxc86xxxx  veth pair 的一端  抓包 有去无回
单向的

回来的在哪里 ？？

```









## 08 2022-04-29-pass



```
host-routing   is XDP（eXpress Data Path） xdp not kernel bypath
```



cilium  vxlan



overhead



https://cilium.io/blog/2021/05/11/cni-benchmark





https://cilium.io/blog/2020/11/10/cilium-19



```
eBPF host routing

bpf_redirect_peer   进到pod内部
bpf_redirect_neigh  pod 出来
```



```
overhead
```





```
cilium datapath

```



```
two pod on same node

tcpdump -pne -i eth0  (in pod)

-p 关闭混杂模式
tcpdump -pne -i (veth pair in root ns)
```





```
pod 内部网卡 32位掩码 相当于没有和它是一个网段的，直接走了默认路由 走三层
返回的不是网关的mac 地址，（返回的是 veth pari in root ns  网卡设备的mac）


arp 查询

只有 request 没有 reply
```







```
tc filter show ingress
tc filter show egress dev lxcssxwerr
```



```
kubectl -n kube-system exec -t cilium-xxx -- bash

cilium monitor 

cilium monitor -vv   >  xx.txt

identify=xxx

cilium identify list

pod-1 pod-2 on save node

Conntrack lookup  (in pod 1 发出去的) 

Conntrack lookup  (in pod 2 收进来的)




同节点的pod 才用到  eBPF host routing


```











```
1  tcpdup
2  cilium monitor
3  iptables trace-id
4  pwru
```





```
pwru --filter-dst-ip x.x.x.x --filter-proto icmp --output-stack

pwru --filter-dst-ip x.x.x.x --filter-proto icmp --output-tuple
```



```
__ip_local_out
```



```
host reachable services
```





```
cilium 16进制
```



```
next cilium vxlan 扩节点
```







## 09 2022-05-01





```
1 cilium vxlan pod  on  diffent node 
```









```
Hairpin  mode
bpftool net show
bpftool map list --bpffs -j | jq 

bpftool map dump id 92 (16 进制 c0 192)
cilium bpf  tunnel list
```





```
dma
sk buffer
```













01:31

```
tcpdump
cilium monitor
```





