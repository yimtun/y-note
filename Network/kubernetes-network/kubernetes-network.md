[toc]





#  xx







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





