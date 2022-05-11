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
--set version=1.10.6 \
--set namespace=kube-system \
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







## 09 2022-05-01-pass





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















```
tcpdump
cilium monitor -vv
```



```
cilium endpoint list
```





```
跨节点通信 什么时候做snat 什么时候不做snat ？？？
cidrxxxx
native-routing 下的参数
```





```
Conntrack lookup
```



https://cilium.io/blog/2021/05/11/cni-benchmark



```
bpftool net show

tc：hoook
```



```
tc filter show dev lxcxxxxx ingress

tc filter show dev lxcxxxxx engress
```





```
cilium tunnel list
```





## 10 2022-05-04-pass





### Native-Routing





https://docs.cilium.io/en/stable/concepts/networking/routing/#id2











https://isovalent.com/blog/post/2021-12-08-ebpf-servicemesh





```
native routing  要求，node l2 可达
```







```
pod  对虚拟机
```



```
精简路由条目
聚合
路由聚合

l3 路由 (no nat )每一跳 都会变mac 地址 但是ip 不变
```



```
查询路由表规则
最长匹配优先  优先最精确的路由  
```



https://support.huawei.com/hedex/hdx.do?docid=EDOC1100196230&lang=zh&id=ZH-CN_CONCEPT_0177102067









https://docs.cilium.io/en/stable/gettingstarted/bird/





```
bird

yum install net-tools bird bird6 -y



```





### ensp











```
## static  route


ar2240



# AR3

sys
sysname AR3

int g0/0/1
ip a 10.0.0.1 24



int g0/0/0
ip a 192.168.2.61 24

dis thi


# AR4
sys
sysname AR4

int g0/0/1
ip a 10.0.1.1 24


int g0/0/0
ip a 192.168.2.62 24

dis thi





dis ip routing-table


AR3
sys
ip route-static 10.0.1.0 24 192.168.2.62



AR4
sys
ip route-static 10.0.0.0 24 192.168.2.61




### OSPF (动态路由)

查看ip
dis ip int b


route1
undo ip route-static 10.0.1.0 24 192.168.2.62




sys
ospf router-id 192.168.2.61
area 0
network 0.0.0.0 255.255.255.255
dis thi


dis ospf peer brief
·


route2

undo ip route-static 10.0.0.0 24 192.168.2.61


sys
ospf router-id 192.168.2.62
area 0
network 0.0.0.0 255.255.255.255
dis thi


dis ospf peer brief



### BGP  边界网关协议

1
sys
undo ospf 1


bgp 12
router-id 192.168.2.61
peer 192.168.2.62 as-number 12
network  10.0.0.0 255.255.255.0

dis thi




2
sys
undo ospf 1


bgp 12

router-id  192.168.2.62

[AR4-bgp]router-id 192.168.2.62
[AR4-bgp]peer 192.168.2.61 as-number 12

network  10.0.1.0 255.255.255.0


dis thi


dis bgp routing-table

```





bgp



https://support.huawei.com/hedex/hdx.do?docid=EDOC1100196230&lang=zh&id=ZH-CN_CONCEPT_0177102067











```
test Native-Routing
```



https://docs.cilium.io/en/latest/concepts/networking/routing/#native-routing





```
路由聚合
```



01:51





```
Each individual node is made aware of all pod IPs of all other nodes and routes are inserted into the Linux kernel routing table to represent this. If all nodes share a single L2 network, then this can be taken care of by enabling the option auto-direct-node-routes: true. Otherwise, an additional system component such as a BGP daemon must be run to distribute the routes. See the guide Using kube-router to run BGP on how to achieve this using the kube-router project.
```



```
L2  可以直接使用。auto-direct-node-routes true
L3 需要
```





```
tunnel: disabled: Enable native routing mode.
ipv4-native-routing-cidr: x.x.x.x/y: Set the CIDR in which native routing can be performed  snat??
```





为了运行本机路由模式，连接运行Cilium的主机的网络必须能够使用提供给POD或其他工作负载的地址转发IP流量。



节点上的Linux内核必须知道如何转发运行Cilium的所有节点的POD包或其他工作负载。这可以通过两种方式实现：



节点本身不知道如何路由所有pod IP，但网络上有一个路由器知道如何到达所有其他pod。在这种情况下，Linux节点被配置为包含指向此类路由器的默认路由。该模型用于云提供商网络集成。有关更多详细信息，请参阅谷歌云、AWS ENI和Azure IPAM。



每个节点都知道所有其他节点的所有pod IP，并将路由插入Linux内核路由表中以表示这一点。如果所有节点共享一个L2网络，则可以通过启用选项auto direct node routes:true来解决这一问题。否则，必须运行额外的系统组件（如BGP守护进程）来分发路由。请参阅使用kube router运行BGP的指南，了解如何使用kube router项目实现这一点。







```
rpcapd
```





## 11 2022-05-06-pass





Host-Reachable Services  + DSR

https://docs.cilium.io/en/latest/gettingstarted/host-services/#host-reachable-services





```
Host-Reachable Services

开启 Host-Reachable Services



kind 

--set hostServices.enable=true

tcpdump -p 关掉混杂模式


tcp 查看三次握手地址 




cilium-agnet --help 

```









------



```

dsr 南北
```









dsr



https://docs.cilium.io/en/latest/gettingstarted/kubeproxy-free/#direct-server-return-dsr





```
require native-routing
```



```
reverse  snat
```









```
helm install cilium ./cilium \
    --namespace kube-system \
    --set tunnel=disabled \  
    --set autoDirectNodeRoutes=true \
    --set kubeProxyReplacement=strict \
    --set loadBalancer.mode=dsr \
    --set k8sServiceHost=REPLACE_WITH_API_SERVER_IP \
    --set k8sServicePort=REPLACE_WITH_API_SERVER_PORT
```







```
monitor-aggregation: "medium"




  ipv4-native-routing-cidr: 10.0.0.0/8
  monitor-aggregation: "none"
  enable-bpf-masquerade: "true"
  enable-endpoint-routes: "false"
```





node port test 



只有http requst 

没有http 200 ok

## 12 2022-05-08-pass





cluster mesh









https://cilium.io/blog/2019/03/12/clustermesh/







```
https://cilium.io/blog/2019/03/12/clustermesh/
```



https://docs.cilium.io/en/v1.11/gettingstarted/clustermesh/clustermesh/#clustermesh















```
kind get clusters
```



```
kubectl --context kind-c1 get pods 
```





```
kind: Cluster
apiVersion: Kind.x-k8s.io/v1alpha4
networking:
        disaleDefaultCNI: true
        kubeProxyMode: "ipvs"
nodes:
        - role: control-plane
        - role: worker
        - role: worker
        
        
kind create         
```





https://metallb.universe.tf/



### kind





docker

heml

cilium cli



```
wget  https://github.com/cilium/cilium-cli/releases/download/v0.11.5/cilium-linux-amd64.tar.gz
```





https://kind.sigs.k8s.io/docs/user/quick-start/#installing-with-a-package-manager







```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.12.0/kind-linux-amd64
chmod +x ./kind
mv ./kind  /usr/local/bin/
```



```
 docker pull kindest/node:v1.23.4
```





```
apt-get install   kubectl=1.23.4-00 -y
```



```
cat >  kind-config1.yaml   <<  EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
networking:
  disableDefaultCNI: true
  podSubnet: "10.10.0.0/16"
  serviceSubnet: "10.11.0.0/16"
EOF
```





```
kind  delete  clusters  c1
```







-----





```
kind create cluster --config ./kind-config1.yaml --name c1  --image kindest/node:v1.23.4
```



```
kubectl cluster-info --context kind-c1
```











```
kubectl config view
```



------



```
kubectl config use-context kind-c1
```



```
helm repo add cilium https://helm.cilium.io/
```



```
helm install cilium cilium/cilium --version 1.11.1 \
  --namespace kube-system \
  --set ipam.mode=kubernetes \
  --set cluster.id=1 \
  --set cluster.name=cluster1 \
  --set hubble.relay.enabled=true \
  --set  hubble.ui.enabled=true  
```





```
helm repo add bitnami https://charts.bitnami.com/bitnami
```





```
cat >  configmap1.yaml   <<  EOF
configInline:
  peers:
  address-pools:
  - name: default
    protocol: layer2
    addresses:
    - 172.18.0.50-172.18.0.99
EOF
```





```
helm install metallb bitnami/metallb \
  --namespace kube-system \
  -f ./configmap1.yaml
```









```
cilium clustermesh enable --create-ca --service-type LoadBalancer

cilium  status
```













```
kubectl get secret --context kind-c1 -n kube-system cilium-ca -o yaml > cilium-ca.yaml
```





-----







```
kind  delete  clusters  c2
```



```
cat >  kind-config2.yaml   <<  EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
networking:
  disableDefaultCNI: true
  podSubnet: "10.20.0.0/16"
  serviceSubnet: "10.21.0.0/16"
EOF
```





```
kind create cluster --config ./kind-config2.yaml --name c2  --image kindest/node:v1.23.4
```





```
kubectl config use-context kind-c2
```



```
helm install cilium cilium/cilium --version=1.11.1 \
  --namespace kube-system \
  --set ipam.mode=kubernetes \
  --set cluster.id=2 \
  --set cluster.name=cluster2 \
  --set hubble.relay.enabled=true \
  --set  hubble.ui.enabled=true  
```









```
cat >  configmap2.yaml   <<  EOF
configInline:
  peers:
  address-pools:
  - name: default
    protocol: layer2
    addresses:
    - 172.18.0.100-172.18.0.149
EOF
```



```
helm install metallb bitnami/metallb \
  --namespace kube-system \
  -f ./configmap2.yaml
```









```
kubectl apply -f cilium-ca.yaml --context kind-c2
```



```
cilium clustermesh enable  --service-type LoadBalancer
```













```
root@172-16-100-7:~# cilium clustermesh  status --context kind-c1
✅ Cluster access information is available:
  - 172.18.0.50:2379
✅ Service "clustermesh-apiserver" of type "LoadBalancer" found
🔌 Cluster Connections:
🔀 Global services: [ min:0 / avg:0.0 / max:0 ]


cilium clustermesh  status --context kind-c2
✅ Cluster access information is available:
  - 172.18.0.100:2379
✅ Service "clustermesh-apiserver" of type "LoadBalancer" found
🔌 Cluster Connections:
🔀 Global services: [ min:0 / avg:0.0 / max:0 ]
root@172-16-100-7:~# 
```







```
cilium clustermesh connect --context kind-c2 --destination-context kind-c1
```





```
cilium clustermesh  status --context kind-c1

✅ Cluster access information is available:
  - 172.18.0.50:2379
✅ Service "clustermesh-apiserver" of type "LoadBalancer" found
✅ All 2 nodes are connected to all clusters [min:1 / avg:1.0 / max:1]
🔌 Cluster Connections:
- cluster2: 2/2 configured, 2/2 connected
🔀 Global services: [ min:5 / avg:5.0 / max:5 ]






cilium clustermesh  status --context kind-c2


✅ Cluster access information is available:
  - 172.18.0.100:2379
✅ Service "clustermesh-apiserver" of type "LoadBalancer" found
✅ All 2 nodes are connected to all clusters [min:1 / avg:1.0 / max:1]
🔌 Cluster Connections:
- cluster1: 2/2 configured, 2/2 connected
🔀 Global services: [ min:5 / avg:5.0 / max:5 ]



```



验证



```
## Verify:
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/1.11.2/examples/kubernetes/clustermesh/global-service-example/cluster1.yaml --context kind-c1
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/1.11.2/examples/kubernetes/clustermesh/global-service-example/cluster2.yaml --context kind-c2
```









```
root@172-16-100-7:~# kubectl  get pod  --context=kind-c2    -o wide
NAME                          READY   STATUS    RESTARTS   AGE    IP            NODE        NOMINATED NODE   READINESS GATES
rebel-base-6985d8f76f-gm82t   1/1     Running   0          11m    10.20.1.217   c2-worker   <none>           <none>
rebel-base-6985d8f76f-wttww   1/1     Running   0          11m    10.20.1.82    c2-worker   <none>           <none>
x-wing-6d58648f95-qnxgg       1/1     Running   0          2m2s   10.20.1.236   c2-worker   <none>           <none>
x-wing-6d58648f95-r2xcv       1/1     Running   0          114s   10.20.1.13    c2-worker   <none>           <none>

root@172-16-100-7:~# kubectl  get pod  --context=kind-c1    -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
rebel-base-6985d8f76f-6ddwd   1/1     Running   0          12m   10.10.1.196   c1-worker   <none>           <none>
rebel-base-6985d8f76f-j5p7h   1/1     Running   0          12m   10.10.1.234   c1-worker   <none>           <none>
x-wing-6d58648f95-9kf22       1/1     Running   0          12m   10.10.1.218   c1-worker   <none>           <none>
x-wing-6d58648f95-ggbwg       1/1     Running   0          12m   10.10.1.152   c1-worker   <none>           <none>
root@172-16-100-7:~# 
```



```
kubectl  get svc --context kind-c1 | grep 'rebel-base'
rebel-base   ClusterIP   10.11.142.37   <none>        80/TCP    55s
```





```
kubectl  get svc --context kind-c2 | grep 'rebel-base'
rebel-base   ClusterIP   10.21.2.210   <none>        80/TCP    3s
```





x-wing  访问  global server   rebel-base



这里把  kind-c2 的  deployment/x-wing 当作客户端  用任何网络可达的 都可以当作客户端

```
kubectl --context kind-c2 exec -ti deployment/x-wing -- curl rebel-base  
```





```
for i in $(seq 1 10000); do kubectl --context kind-c2 exec -ti deployment/x-wing -- curl rebel-base  ;done
```





删除  kind-c1 的 pod

```
kubectl scale deployment rebel-base --replicas=0 --context kind-c1
```



```
kubectl  get pod  --context kind-c1
NAME                      READY   STATUS    RESTARTS   AGE
x-wing-6d58648f95-9kf22   1/1     Running   0          18m
x-wing-6d58648f95-ggbwg   1/1     Running   0          18m
root@172-16-100-7:~# 
```



用 kind-c1 当作客户端 

```
for i in $(seq 1 10000); do kubectl --context kind-c1 exec -ti deployment/x-wing -- curl rebel-base  ;done
```



----







##### connectivity test



```
cilium hubble port-forward&  --context kind-c1
```









```
root@172-16-100-7:~# cilium connectivity test --context kind-c1
ℹ️  Single-node environment detected, enabling single-node connectivity test
ℹ️  Monitor aggregation detected, will skip some flow validation steps
⌛ [kind-c1] Waiting for deployments [client client2 echo-same-node] to become ready...
⌛ [kind-c1] Waiting for deployments [] to become ready...
⌛ [kind-c1] Waiting for CiliumEndpoint for pod cilium-test/client-7568bc7f86-ccjtz to appear...
⌛ [kind-c1] Waiting for CiliumEndpoint for pod cilium-test/client2-686d5f784b-mbsdt to appear...
⌛ [kind-c1] Waiting for CiliumEndpoint for pod cilium-test/echo-same-node-5767b7b99d-4snb5 to appear...
⌛ [kind-c1] Waiting for Service cilium-test/echo-same-node to become ready...
⌛ [kind-c1] Waiting for NodePort 172.18.0.2:30173 (cilium-test/echo-same-node) to become ready...
⌛ [kind-c1] Waiting for NodePort 172.18.0.3:30173 (cilium-test/echo-same-node) to become ready...
ℹ️  Skipping IPCache check
⌛ [kind-c1] Waiting for pod cilium-test/client-7568bc7f86-ccjtz to reach default/kubernetes service...
⌛ [kind-c1] Waiting for pod cilium-test/client2-686d5f784b-mbsdt to reach default/kubernetes service...
🔭 Enabling Hubble telescope...
⚠️  Unable to contact Hubble Relay, disabling Hubble telescope and flow validation: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial tcp 127.0.0.1:4245: connect: connection refused"
ℹ️  Expose Relay locally with:
   cilium hubble enable
   cilium hubble port-forward&
ℹ️  Cilium version: 1.11.1
🏃 Running tests...

[=] Test [no-policies]
......................
[=] Test [allow-all-except-world]
........
[=] Test [client-ingress]
..
[=] Test [echo-ingress]
..
[=] Test [client-egress]
..
[=] Test [to-entities-world]
......
[=] Test [to-cidr-1111]
....
[=] Test [echo-ingress-l7]
..
[=] Test [client-egress-l7]
........
[=] Test [dns-only]
........
[=] Test [to-fqdns]
......
✅ All 11 tests (70 actions) successful, 0 tests skipped, 0 scenarios skipped.
root@172-16-100-7:~# 
```





```
root@172-16-100-7:~# cilium connectivity test --context kind-c2
ℹ️  Single-node environment detected, enabling single-node connectivity test
ℹ️  Monitor aggregation detected, will skip some flow validation steps
⌛ [kind-c2] Waiting for deployments [client client2 echo-same-node] to become ready...
⌛ [kind-c2] Waiting for deployments [] to become ready...
⌛ [kind-c2] Waiting for CiliumEndpoint for pod cilium-test/client-7568bc7f86-pqq2q to appear...
⌛ [kind-c2] Waiting for CiliumEndpoint for pod cilium-test/client2-686d5f784b-q4zvp to appear...
⌛ [kind-c2] Waiting for CiliumEndpoint for pod cilium-test/echo-same-node-5767b7b99d-mpfj5 to appear...
⌛ [kind-c2] Waiting for Service cilium-test/echo-same-node to become ready...
⌛ [kind-c2] Waiting for NodePort 172.18.0.5:31763 (cilium-test/echo-same-node) to become ready...
⌛ [kind-c2] Waiting for NodePort 172.18.0.4:31763 (cilium-test/echo-same-node) to become ready...
ℹ️  Skipping IPCache check
⌛ [kind-c2] Waiting for pod cilium-test/client-7568bc7f86-pqq2q to reach default/kubernetes service...
⌛ [kind-c2] Waiting for pod cilium-test/client2-686d5f784b-q4zvp to reach default/kubernetes service...
🔭 Enabling Hubble telescope...
⚠️  Unable to contact Hubble Relay, disabling Hubble telescope and flow validation: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial tcp 127.0.0.1:4245: connect: connection refused"
ℹ️  Expose Relay locally with:
   cilium hubble enable
   cilium hubble port-forward&
ℹ️  Cilium version: 1.11.1
🏃 Running tests...

[=] Test [no-policies]
......................
[=] Test [allow-all-except-world]
........
[=] Test [client-ingress]
..
[=] Test [echo-ingress]
..
[=] Test [client-egress]
..
[=] Test [to-entities-world]
......
[=] Test [to-cidr-1111]
....
[=] Test [echo-ingress-l7]
..
[=] Test [client-egress-l7]
........
[=] Test [dns-only]
........
[=] Test [to-fqdns]
......
✅ All 11 tests (70 actions) successful, 0 tests skipped, 0 scenarios skipped.
root@172-16-100-7:~# 
```







```
cilium hubble port-forward&  --context kind-c2
```







```
cilium hubble enable --context kind-c1
```



```
cilium connectivity test --context kind-c1
cilium connectivity test --context kind-c2
```



```
kubectl  get pod -n cilium-test  --context kind-c1
kubectl  get pod -n cilium-test  --context kind-c2
```



```
kubectl  delete  ns cilium-test  --context kind-c1
kubectl  delete  ns cilium-test  --context kind-c2
```







```
docker cp /usr/local/bin/cilium  c1-control-plane:/opt/

/opt/cilium connectivity test --context kind-c1


```







-----





命令行





```
kubectl  get pod -A --context kind-c1
kubectl  get pod -A --context kind-c2
```



```
docker exec -it c1-worker   crictl  images
docker exec -it c2-worker   crictl  images
```



```
docker exec -it c2-worker  crictl  pull quay.io/cilium/cilium:1.11.1
```











-----



















----



##### 信息收集



```
kubectl  get pod -A --context kind-c1
NAMESPACE            NAME                                       READY   STATUS    RESTARTS   AGE
default              rebel-base-6985d8f76f-fd5hf                1/1     Running   0          15m
default              rebel-base-6985d8f76f-xrngp                1/1     Running   0          15m
default              x-wing-6d58648f95-578fj                    1/1     Running   0          15m
default              x-wing-6d58648f95-v59rt                    1/1     Running   0          15m
kube-system          cilium-45gtt                               1/1     Running   0          18m
kube-system          cilium-mkrgl                               1/1     Running   0          18m
kube-system          cilium-operator-67f9b484d8-92pkm           1/1     Running   0          86m
kube-system          cilium-operator-67f9b484d8-9f5wl           1/1     Running   0          56m
kube-system          clustermesh-apiserver-cbd4456bf-5s4nt      2/2     Running   0          23m
kube-system          coredns-64897985d-dqw6w                    1/1     Running   0          88m
kube-system          coredns-64897985d-xbcwh                    1/1     Running   0          88m
kube-system          etcd-c1-control-plane                      1/1     Running   0          88m
kube-system          hubble-relay-85664d47c4-zsf9p              1/1     Running   0          86m
kube-system          hubble-ui-7c6789876c-nkqpg                 3/3     Running   0          56m
kube-system          kube-apiserver-c1-control-plane            1/1     Running   0          88m
kube-system          kube-controller-manager-c1-control-plane   1/1     Running   0          88m
kube-system          kube-proxy-l6sph                           1/1     Running   0          87m
kube-system          kube-proxy-tkrng                           1/1     Running   0          88m
kube-system          kube-scheduler-c1-control-plane            1/1     Running   0          88m
kube-system          metallb-controller-66865674bf-6kn2x        1/1     Running   0          25m
kube-system          metallb-speaker-fc2vb                      1/1     Running   0          25m
local-path-storage   local-path-provisioner-5ddd94ff66-xb6rj    1/1     Running   0          88m







docker exec -it c1-worker   crictl  images

IMAGE                                      TAG                    IMAGE ID            SIZE
docker.io/bitnami/metallb-controller       0.12.1-debian-10-r61   81d1a3e4dc8ad       43.9MB
docker.io/bitnami/metallb-speaker          0.12.1-debian-10-r62   da61e66ac933a       44.8MB
docker.io/cilium/json-mock                 1.2                    094a89b064ebc       62.5MB
docker.io/kindest/kindnetd                 v20211122-a2c10462     ba113d2047d43       40.9MB
docker.io/rancher/local-path-provisioner   v0.0.14                e422121c9c5f9       13.4MB
k8s.gcr.io/build-image/debian-base         buster-v1.7.2          19bad6b08adae       21.1MB
k8s.gcr.io/coredns/coredns                 v1.8.6                 a4ca41631cc7a       13.6MB
k8s.gcr.io/etcd                            3.5.1-0                25f8c7f3da61c       98.9MB
k8s.gcr.io/kube-apiserver                  v1.23.4                0e16468a4aa36       79.6MB
k8s.gcr.io/kube-controller-manager         v1.23.4                65dd3c630e90e       68.1MB
k8s.gcr.io/kube-proxy                      v1.23.4                163898317bef2       113MB
k8s.gcr.io/kube-scheduler                  v1.23.4                696ef30502b11       54.8MB
k8s.gcr.io/pause                           3.6                    6270bb605e12e       302kB
docker.io/envoyproxy/envoy                 <none>                 fe28da4ca57ab       51.4MB
docker.io/library/nginx                    1.15.8                 f09fe80eb0e75       44.7MB
quay.io/cilium/cilium                      <none>                 11553cb737845       157MB
quay.io/cilium/clustermesh-apiserver       <none>                 8f52e3ea3edf9       15.5MB
quay.io/cilium/hubble-relay                <none>                 d42689b386ff3       10.2MB
quay.io/cilium/hubble-ui-backend           <none>                 a029b5706c2d2       14.4MB
quay.io/cilium/hubble-ui                   <none>                 154fa38916304       12.2MB
quay.io/cilium/operator-generic            <none>                 21681b3deb25a       16.3MB
quay.io/coreos/etcd                        v3.4.13                d1985d4043858       32.6MB




docker exec -it c1-control-plane    crictl  images


IMAGE                                      TAG                  IMAGE ID            SIZE
docker.io/kindest/kindnetd                 v20211122-a2c10462   ba113d2047d43       40.9MB
k8s.gcr.io/coredns/coredns                 v1.8.6               a4ca41631cc7a       13.6MB
k8s.gcr.io/etcd                            3.5.1-0              25f8c7f3da61c       98.9MB
k8s.gcr.io/kube-apiserver                  v1.23.4              0e16468a4aa36       79.6MB
quay.io/cilium/cilium                      <none>               11553cb737845       157MB
k8s.gcr.io/kube-proxy                      v1.23.4              163898317bef2       113MB
quay.io/cilium/operator-generic            <none>               21681b3deb25a       16.3MB
docker.io/rancher/local-path-provisioner   v0.0.14              e422121c9c5f9       13.4MB
k8s.gcr.io/build-image/debian-base         buster-v1.7.2        19bad6b08adae       21.1MB
k8s.gcr.io/kube-controller-manager         v1.23.4              65dd3c630e90e       68.1MB
k8s.gcr.io/kube-scheduler                  v1.23.4              696ef30502b11       54.8MB
k8s.gcr.io/pause                           3.6                  6270bb605e12e       302kB









kubectl  get pod -A --context kind-c2

NAMESPACE            NAME                                       READY   STATUS    RESTARTS   AGE
default              rebel-base-6985d8f76f-25sxq                1/1     Running   0          16m
default              rebel-base-6985d8f76f-zpzd2                1/1     Running   0          16m
default              x-wing-6d58648f95-t6dfl                    1/1     Running   0          16m
default              x-wing-6d58648f95-tw54m                    1/1     Running   0          16m
kube-system          cilium-4b4rw                               1/1     Running   0          19m
kube-system          cilium-operator-67f9b484d8-49lpr           1/1     Running   0          37m
kube-system          cilium-operator-67f9b484d8-h8ncw           1/1     Running   0          37m
kube-system          cilium-wsxzg                               1/1     Running   0          19m
kube-system          clustermesh-apiserver-866d964d6c-c4rdf     2/2     Running   0          21m
kube-system          coredns-64897985d-7wqgr                    1/1     Running   0          38m
kube-system          coredns-64897985d-ckgph                    1/1     Running   0          38m
kube-system          etcd-c2-control-plane                      1/1     Running   0          38m
kube-system          hubble-relay-85664d47c4-k8jhf              1/1     Running   0          37m
kube-system          hubble-ui-7c6789876c-h8rw5                 3/3     Running   0          37m
kube-system          kube-apiserver-c2-control-plane            1/1     Running   0          38m
kube-system          kube-controller-manager-c2-control-plane   1/1     Running   0          38m
kube-system          kube-proxy-6d89k                           1/1     Running   0          38m
kube-system          kube-proxy-dx28z                           1/1     Running   0          38m
kube-system          kube-scheduler-c2-control-plane            1/1     Running   0          38m
kube-system          metallb-controller-66865674bf-xdh9c        1/1     Running   0          26m
kube-system          metallb-speaker-zd5v4                      1/1     Running   0          26m
local-path-storage   local-path-provisioner-5ddd94ff66-vpbjb    1/1     Running   0          38m





docker exec -it c2-worker   crictl  images

IMAGE                                      TAG                    IMAGE ID            SIZE
docker.io/bitnami/metallb-controller       0.12.1-debian-10-r61   81d1a3e4dc8ad       43.9MB
docker.io/bitnami/metallb-speaker          0.12.1-debian-10-r62   da61e66ac933a       44.8MB
docker.io/cilium/json-mock                 1.2                    094a89b064ebc       62.5MB
quay.io/cilium/hubble-ui                   <none>                 154fa38916304       12.2MB
docker.io/kindest/kindnetd                 v20211122-a2c10462     ba113d2047d43       40.9MB
docker.io/library/nginx                    1.15.8                 f09fe80eb0e75       44.7MB
quay.io/cilium/cilium                      <none>                 11553cb737845       157MB
docker.io/rancher/local-path-provisioner   v0.0.14                e422121c9c5f9       13.4MB
k8s.gcr.io/build-image/debian-base         buster-v1.7.2          19bad6b08adae       21.1MB
k8s.gcr.io/coredns/coredns                 v1.8.6                 a4ca41631cc7a       13.6MB
k8s.gcr.io/etcd                            3.5.1-0                25f8c7f3da61c       98.9MB
k8s.gcr.io/kube-apiserver                  v1.23.4                0e16468a4aa36       79.6MB
k8s.gcr.io/kube-controller-manager         v1.23.4                65dd3c630e90e       68.1MB
k8s.gcr.io/kube-proxy                      v1.23.4                163898317bef2       113MB
k8s.gcr.io/kube-scheduler                  v1.23.4                696ef30502b11       54.8MB
docker.io/envoyproxy/envoy                 <none>                 fe28da4ca57ab       51.4MB
quay.io/cilium/hubble-relay                <none>                 d42689b386ff3       10.2MB
quay.io/cilium/hubble-ui-backend           <none>                 a029b5706c2d2       14.4MB
quay.io/cilium/operator-generic            <none>                 21681b3deb25a       16.3MB
k8s.gcr.io/pause                           3.6                    6270bb605e12e       302kB
quay.io/cilium/clustermesh-apiserver       <none>                 8f52e3ea3edf9       15.5MB
quay.io/coreos/etcd                        v3.4.13                d1985d4043858       32.6MB







docker exec -it c2-control-plane   crictl  images

IMAGE                                      TAG                  IMAGE ID            SIZE
quay.io/cilium/cilium                      <none>               11553cb737845       157MB
docker.io/kindest/kindnetd                 v20211122-a2c10462   ba113d2047d43       40.9MB
k8s.gcr.io/build-image/debian-base         buster-v1.7.2        19bad6b08adae       21.1MB
quay.io/cilium/operator-generic            <none>               21681b3deb25a       16.3MB
docker.io/rancher/local-path-provisioner   v0.0.14              e422121c9c5f9       13.4MB
k8s.gcr.io/coredns/coredns                 v1.8.6               a4ca41631cc7a       13.6MB
k8s.gcr.io/etcd                            3.5.1-0              25f8c7f3da61c       98.9MB
k8s.gcr.io/kube-apiserver                  v1.23.4              0e16468a4aa36       79.6MB
k8s.gcr.io/kube-controller-manager         v1.23.4              65dd3c630e90e       68.1MB
k8s.gcr.io/kube-proxy                      v1.23.4              163898317bef2       113MB
k8s.gcr.io/kube-scheduler                  v1.23.4              696ef30502b11       54.8MB
k8s.gcr.io/pause                           3.6                  6270bb605e12e       302kB



```









```
root@172-16-100-7:~# cilium clustermesh status  --context kind-c1
✅ Cluster access information is available:
  - 172.18.0.50:2379
✅ Service "clustermesh-apiserver" of type "LoadBalancer" found
✅ All 2 nodes are connected to all clusters [min:1 / avg:1.0 / max:1]
🔌 Cluster Connections:
- cluster2: 2/2 configured, 2/2 connected
🔀 Global services: [ min:6 / avg:6.0 / max:6 ]


root@172-16-100-7:~# cilium clustermesh status  --context kind-c2
✅ Cluster access information is available:
  - 172.18.0.100:2379
✅ Service "clustermesh-apiserver" of type "LoadBalancer" found
✅ All 2 nodes are connected to all clusters [min:1 / avg:1.0 / max:1]
🔌 Cluster Connections:
- cluster1: 2/2 configured, 2/2 connected
🔀 Global services: [ min:6 / avg:6.0 / max:6 ]
root@172-16-100-7:~# 
```







```
root@172-16-100-7:~# cilium status  --context kind-c1
    /¯¯\
 /¯¯\__/¯¯\    Cilium:         OK
 \__/¯¯\__/    Operator:       OK
 /¯¯\__/¯¯\    Hubble:         OK
 \__/¯¯\__/    ClusterMesh:    OK
    \__/

DaemonSet         cilium                   Desired: 2, Ready: 2/2, Available: 2/2
Deployment        cilium-operator          Desired: 2, Ready: 2/2, Available: 2/2
Deployment        hubble-relay             Desired: 1, Ready: 1/1, Available: 1/1
Deployment        hubble-ui                Desired: 1, Ready: 1/1, Available: 1/1
Deployment        clustermesh-apiserver    Desired: 1, Ready: 1/1, Available: 1/1
Containers:       hubble-relay             Running: 1
                  hubble-ui                Running: 1
                  clustermesh-apiserver    Running: 1
                  cilium                   Running: 2
                  cilium-operator          Running: 2
Cluster Pods:     11/11 managed by Cilium
Image versions    cilium                   quay.io/cilium/cilium:v1.11.1@sha256:251ff274acf22fd2067b29a31e9fda94253d2961c061577203621583d7e85bd2: 2
                  cilium-operator          quay.io/cilium/operator-generic:v1.11.1@sha256:977240a4783c7be821e215ead515da3093a10f4a7baea9f803511a2c2b44a235: 2
                  hubble-relay             quay.io/cilium/hubble-relay:v1.11.1@sha256:23d40b2a87a5bf94e0365bd9606721c96f78b8304b61725dca45a0b8a6048203: 1
                  hubble-ui                quay.io/cilium/hubble-ui:v0.8.5@sha256:4eaca1ec1741043cfba6066a165b3bf251590cf4ac66371c4f63fbed2224ebb4: 1
                  hubble-ui                quay.io/cilium/hubble-ui-backend:v0.8.5@sha256:2bce50cf6c32719d072706f7ceccad654bfa907b2745a496da99610776fe31ed: 1
                  hubble-ui                docker.io/envoyproxy/envoy:v1.18.4@sha256:e5c2bb2870d0e59ce917a5100311813b4ede96ce4eb0c6bfa879e3fbe3e83935: 1
                  clustermesh-apiserver    quay.io/coreos/etcd:v3.4.13: 1
                  clustermesh-apiserver    quay.io/cilium/clustermesh-apiserver:v1.11.1@sha256:5732f2ce99096d1c740f3805260dbcfefbe6d7d18d1ac07707ff4ef9536b0ec6: 1
root@172-16-100-7:~# 
root@172-16-100-7:~# 
root@172-16-100-7:~# 


root@172-16-100-7:~# cilium status  --context kind-c2
    /¯¯\
 /¯¯\__/¯¯\    Cilium:         OK
 \__/¯¯\__/    Operator:       OK
 /¯¯\__/¯¯\    Hubble:         OK
 \__/¯¯\__/    ClusterMesh:    OK
    \__/

DaemonSet         cilium                   Desired: 2, Ready: 2/2, Available: 2/2
Deployment        cilium-operator          Desired: 2, Ready: 2/2, Available: 2/2
Deployment        hubble-relay             Desired: 1, Ready: 1/1, Available: 1/1
Deployment        hubble-ui                Desired: 1, Ready: 1/1, Available: 1/1
Deployment        clustermesh-apiserver    Desired: 1, Ready: 1/1, Available: 1/1
Containers:       cilium                   Running: 2
                  cilium-operator          Running: 2
                  hubble-relay             Running: 1
                  hubble-ui                Running: 1
                  clustermesh-apiserver    Running: 1
Cluster Pods:     11/11 managed by Cilium
Image versions    cilium                   quay.io/cilium/cilium:v1.11.1@sha256:251ff274acf22fd2067b29a31e9fda94253d2961c061577203621583d7e85bd2: 2
                  cilium-operator          quay.io/cilium/operator-generic:v1.11.1@sha256:977240a4783c7be821e215ead515da3093a10f4a7baea9f803511a2c2b44a235: 2
                  hubble-relay             quay.io/cilium/hubble-relay:v1.11.1@sha256:23d40b2a87a5bf94e0365bd9606721c96f78b8304b61725dca45a0b8a6048203: 1
                  hubble-ui                quay.io/cilium/hubble-ui:v0.8.5@sha256:4eaca1ec1741043cfba6066a165b3bf251590cf4ac66371c4f63fbed2224ebb4: 1
                  hubble-ui                quay.io/cilium/hubble-ui-backend:v0.8.5@sha256:2bce50cf6c32719d072706f7ceccad654bfa907b2745a496da99610776fe31ed: 1
                  hubble-ui                docker.io/envoyproxy/envoy:v1.18.4@sha256:e5c2bb2870d0e59ce917a5100311813b4ede96ce4eb0c6bfa879e3fbe3e83935: 1
                  clustermesh-apiserver    quay.io/coreos/etcd:v3.4.13: 1
                  clustermesh-apiserver    quay.io/cilium/clustermesh-apiserver:v1.11.1@sha256:5732f2ce99096d1c740f3805260dbcfefbe6d7d18d1ac07707ff4ef9536b0ec6: 1
root@172-16-100-7:~# 
```



## 13 2022-05-09-pass





```
cilium bpf tunnel list

cilium service list

cilium identity list
```





https://01.org/blogs/xuyizhou/2021/accelerate-istio-dataplane-ebpf-part-1







https://thenewstack.io/how-ebpf-streamlines-the-service-mesh/





https://isovalent.com/blog/post/2021-12-08-ebpf-servicemesh







https://isovalent.com/blog/post/2022-05-03-servicemesh-security



----------------------------------- cilium end------------









