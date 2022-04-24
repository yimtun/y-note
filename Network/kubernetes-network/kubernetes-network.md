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









#  1 网络基础



网络号

主机号

vl an





# 2  



