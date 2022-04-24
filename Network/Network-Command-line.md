

[toc]





# tcpdump





```
tcpdump -pne
```





# ifconfig



##  设置ip 





## 启动网卡





```
ifconfig lo up
```





# ip 





## 创建网桥



```
ip link add br0  type bridge
```





## 启动网卡



```
ip link set br0 up
```





##  查看接口详细信息  查看接口类型







## tap 设备 

```
ip -d link show vnet3 
```



 vnet3 是 tap 设备




```
tun type tap pi off vnet_hdr off persist off 
```







## tun 设备



```
[root@localhost ~]# ip -d link show tun0
17: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN mode DEFAULT group default qlen 100
    link/none  promiscuity 0 
    tun addrgenmode random numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 
```







##  veth 设备




```
ip   link show vethf0732364
```

 vethf0732364  是 veth 设备 

它的对端 veth index 是 if3

```
6: vethf0732364@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default 
    link/ether 6a:c1:3a:26:d2:82 brd ff:ff:ff:ff:ff:ff link-netnsid 0 promiscuity 1 minmtu 68 maxmtu 65535 
    veth 
    bridge_slave state forwarding priority 32 cost 2 hairpin on guard off root_block off fastleave off learning on flood on port_id 0x8002 port_no 0x2 designated_port 32770 designated_cost 0 designated_bridge 8000.ba:84:69:e0:1c:60 designated_root 8000.ba:84:69:e0:1c:60 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 
```







###  VTEP  设备 vxlan 设备



flannel.1 是vxlan 设备 它的  VNI 值  是1 

**dstport** : `VTEP` 通信的端口 8472



```
ip -d link show  flannel.1
```



```
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default 
    link/ether 56:99:3f:dd:ef:71 brd ff:ff:ff:ff:ff:ff promiscuity 0 minmtu 68 maxmtu 65535 
    vxlan id 1 local 172.16.100.14 dev enp0s5 srcport 0 0 dstport 8472 nolearning ttl auto ageing 300 udpcsum noudp6zerocsumtx noudp6zerocsumrx addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 
```





```
bridge  fdb
33:33:00:00:00:01 dev enp0s5 self permanent
01:00:5e:00:00:01 dev enp0s5 self permanent
33:33:ff:7d:4b:a1 dev enp0s5 self permanent
01:80:c2:00:00:00 dev enp0s5 self permanent
01:80:c2:00:00:03 dev enp0s5 self permanent
01:80:c2:00:00:0e dev enp0s5 self permanent
33:33:00:00:00:fb dev enp0s5 self permanent
01:00:5e:00:00:fb dev enp0s5 self permanent
33:33:00:00:00:01 dev docker0 self permanent
01:00:5e:00:00:6a dev docker0 self permanent
33:33:00:00:00:6a dev docker0 self permanent
01:00:5e:00:00:01 dev docker0 self permanent
01:00:5e:00:00:fb dev docker0 self permanent
02:42:ab:37:41:fe dev docker0 vlan 1 master docker0 permanent
02:42:ab:37:41:fe dev docker0 master docker0 permanent
ca:9c:12:6c:3a:05 dev flannel.1 dst 172.16.100.15 self permanent
56:99:3f:dd:ef:71 dev flannel.1 dst 172.16.100.14 self permanent
22:7e:a5:fd:66:f2 dev flannel.1 dst 172.16.100.15 self permanent
root@node-172-16-100-13:~# 
```





#  ethtool





```
 ethtool  -S eth0
```





```
ethtool  -S  veth545dd5fd
```





# brctl 查看bridge 上的mac





```
brctl showmacs cni0
```



```
port no mac addr                is local?       ageing timer
  1     22:66:bb:eb:95:ec       yes                0.00
  1     22:66:bb:eb:95:ec       yes                0.00
  3     6e:1e:ee:a3:1d:57       yes                0.00
  3     6e:1e:ee:a3:1d:57       yes                0.00
  2     fa:47:3c:94:25:dd       yes                0.00
  2     fa:47:3c:94:25:dd       yes                0.00
```





```
brctl  showmacs cni0
port no mac addr                is local?       ageing timer
  1     66:67:33:87:24:a8       yes                0.00
  1     66:67:33:87:24:a8       yes                0.00
  2     6a:c1:3a:26:d2:82       yes                0.00        vethf0732364@if3
  2     6a:c1:3a:26:d2:82       yes                0.00
  3     9e:b8:37:0e:4e:24       yes                0.00
  3     9e:b8:37:0e:4e:24       yes                0.00
  3     ea:49:ae:c5:21:bb       no                 5.88
  2     ea:eb:e1:94:4b:82       no                 7.41
```





```
ethtool -S vethf0732364 | grep peer_ifindex
     peer_ifindex: 3
```











yes 是 bridge 自己的mac 地址

no 学习的



