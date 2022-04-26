

[toc]





# tcpdump





```
tcpdump -pne
tcpdump  -pne -i eth0 icmp
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









# route



## 概览



```
[root@router-192-168-20-10 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.10.0    0.0.0.0         255.255.255.0   U     100    0        0 eth2
192.168.20.0    0.0.0.0         255.255.255.0   U     101    0        0 eth3
```

gw 0.0.0.0  代表自己

Metric 经过几个路由器 直连的 0



##  static route



| router1 | eth0                                        | eth1                                     | hostname             |
| ------- | ------------------------------------------- | ---------------------------------------- | -------------------- |
|         | 192.168.10.10/24    网卡配置文件不设置网关  | 192.168.20.10/24  网卡配置文件不设置网关 | router-192-168-20-10 |
| router2 | eth0                                        | eth1                                     |                      |
|         | 192.168.20.20/24     网卡配置文件不设置网关 | 192.168.80.10/24  网卡配置文件不设置网关 | router-192-168-20-20 |
| pc1     | 192.168.10.123/24     gw 192.168.10.20      |                                          |                      |
| pc2     | 192.168.80.123/24     gw 192.168.80.10      |                                          |                      |
|         |                                             |                                          |                      |
|         |                                             |                                          |                      |
|         |                                             |                                          |                      |





查看当前路由 没有添加任何路由



router1



```
[root@router-192-168-20-10 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.10.0    0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.20.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1
```





router2





```
route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.20.0    0.0.0.0         255.255.255.0   U     102    0        0 eth0
192.168.80.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1
```







### router1 添加路由



```
route add -net 192.168.80.0 netmask 255.255.255.0  gw 192.168.20.20
```



```
[root@router-192-168-20-10 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.10.0    0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.20.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1
192.168.80.0    192.168.20.20   255.255.255.0   UG    0      0        0 eth1 
```



```
G 必须给到网关
没有G 代表直连的网络
```





### router2 添加路由



```
route add -net 192.168.10.0 netmask 255.255.255.0  gw 192.168.20.10
```



```
[root@router-192-168-20-20 network-scripts]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.10.0    192.168.20.10   255.255.255.0   UG    0      0        0 eth0
192.168.20.0    0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.80.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1
[root@router-192-168-20-20 network-scripts]# 
```





###  测试



```
ping 192.168.10.123
ping 192.168.80.123
```





###  路由配置持久化



router1

```
/etc/sysconfig/network-scripts/route-eth1
```



去往 192.168.80.0/24 是从网卡  eth1 发送出去的 所以是   route-eth0



```
ADDRESS0=192.168.80.0
NETMASK0=255.255.255.0
GATEWAY0=192.168.20.20
```





router2



```
/etc/sysconfig/network-scripts/route-eth0
```

去往 192.168.10.0/24 是从网卡  eth0 发送出去的 所以是   route-eth0



```
ADDRESS0=192.168.10.0
NETMASK0=255.255.255.0
GATEWAY0=192.168.20.10
```





### 再添加一个网段 







## default route 默认路由



网关就是默认路由





```
[root@router-192-168-20-10 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.10.0    0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.20.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1
192.168.80.0    192.168.20.20   255.255.255.0   UG    101    0        0 eth1
[root@router-192-168-20-10 ~]# 
```





```
route add -net 0.0.0.0 netmask 0.0.0.0  gw 192.168.20.20
```



```
[root@router-192-168-20-10 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.20.20   0.0.0.0         UG    0      0        0 eth1
192.168.10.0    0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.20.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1
192.168.80.0    192.168.20.20   255.255.255.0   UG    101    0        0 eth1
```





```
route del -net 192.168.80.0 netmask 255.255.255.0  gw 192.168.20.20
```





指保留默认路由

```
[root@router-192-168-20-10 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.20.20   0.0.0.0         UG    0      0        0 eth1
192.168.10.0    0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.20.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1
```





### 默认路由 添加方法2



通过给网卡添加网关的方式产生默认路由





```
route del -net 0.0.0.0 netmask 0.0.0.0  gw 192.168.20.20
```



```
[root@router-192-168-20-10 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.10.0    0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.20.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1
```





```
cat /etc/sysconfig/network-scripts/ifcfg-eth1 
TYPE=Ethernet
BOOTPROTO=none
IPADDR=192.168.20.10
GATEWAY=192.168.20.20
PREFIX=24
NAME=eth1
DEVICE=eth1
ONBOOT=yes
```





```
rm -rf /etc/sysconfig/network-scripts/route-eth1
```



```
systemctl  restart network
```



```
[root@router-192-168-20-10 network-scripts]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.20.20   0.0.0.0         UG    101    0        0 eth1
192.168.10.0    0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.20.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1
[root@router-192-168-20-10 network-scripts]# 
```





##  策略路由

策略路由还关注源地址



多张路由表

根据源地址， 使用不同的路由表

根据数据包的协议  选择不同的路由表



定义多张路由表



| 设备名  | eth0                                  | eth1                  | eth2                 | 策略                            |
| ------- | ------------------------------------- | --------------------- | -------------------- | ------------------------------- |
| router1 | 192.168.10.10/24  内网                | 192.168.20.10/24 电信 | 192.168.30.10/24联通 |                                 |
| router2 | 192.168.20.20/24 电信                 | 192.168.80.10/24 内网 | 192.168.30.20/24联通 |                                 |
| pc1     | 192.168.10.123/24    gw 192.168.10.20 |                       |                      | 去192.168.80.123/24走电信       |
| pc2     | 192.168.10.124/24    gw 192.168.10.20 |                       |                      | 去192.168.80.123/24其他的走联通 |
| pc3     | 192.168.80.123/24    gw 192.168.80.10 |                       |                      |                                 |
|         |                                       |                       |                      |                                 |
|         |                                       |                       |                      |                                 |
|         |                                       |                       |                      |                                 |
|         |                                       |                       |                      |                                 |







### router1 设置策略路由



创建多张路由表 添加编号

251  

252

创建规则



查看当前系统上的路由表



```
cat /etc/iproute2/rt_tables 
255     local
254     main           route add 默认在这里
253     default
0       unspec
```





```
vim /etc/iproute2/rt_tables

255     local
254     main            #route add 默认在这里  route -n 模式查看的是main 表 等于 ip route show table main
253     default
0       unspec
251     dianxin     # 新加
252     liantong     # 新加
```



指定路由表查看路由



查看main 表

```
 ip route show table main
 ip route show table 254
```





向指定路由表里添加路由



  192.168.20.20 经由  去  192.168.80/24 网段 添加到 编号为251 的路由表



```
ip route add 192.168.80/24 via 192.168.20.20 table 251 
```



```
ip route add 192.168.80/24 via 192.168.30.20 table 252
```



```
[root@router-192-168-20-10 network-scripts]# ip route show table dianxin
192.168.80.0/24 via 192.168.20.20 dev eth1 
[root@router-192-168-20-10 network-scripts]# ip route show table liantong
192.168.80.0/24 via 192.168.30.20 dev eth2 
[root@router-192-168-20-10 network-scripts]# 
```







创建路由策略

ip rule add  匹配项目  动作（选择哪个表，拒绝不允许通过）

ip rule del

ip rule show





当前 默认策略  编号决定 检查 顺序

```
[root@router-192-168-20-10 network-scripts]# ip rule show
0:      from all lookup local 
32766:  from all lookup main 
32767:  from all lookup default 
```





注意顺序。先检查  123 再检查网段   注意编号优先级

数字越小 级别越高

```
ip rule add  from 192.168.10.123/32 to 192.168.80.123  table 251 pref 10
```



```
ip rule add  from 192.168.10.0/24  to  192.168.80.123   table 252 pref 100
```





拒绝某个ip 



```
ip rule add  from 192.168.10.124/32 to 192.168.80.123  prohibit  pref 9 table main
```





```
ip rule show table main
```



删除rule

```
ip rule  del   from 192.168.10.124/32 to 192.168.80.123  table main
```



##### ip rule 配置持久化



```
vim /etc/rc.d/rc.local


ip rule add  from 192.168.10.123/32 to 192.168.80.123  table 251 pref 10
ip rule add  from 192.168.10.0/24  to  192.168.80.123   table 252 pref 100
```





```
cat  /etc/rc.d/rc.local 
#!/bin/bash


ip rule add  from 192.168.10.123/32 to 192.168.80.123  table 251 pref 10
ip rule add  from 192.168.10.0/24  to  192.168.80.123   table 252 pref 100

touch /var/lock/subsys/local
```







### router 2 设置静态路由





到达  192.168.10.123 走网关  192.168.20.10

```
/etc/sysconfig/network-scripts/route-eth0
```

```
ADDRESS0=192.168.10.123
NETMASK0=255.255.255.255
GATEWAY0=192.168.20.10
```







到达  除了 192.168.10.0/24 网段 除了     192.168.10.123    走网关  192.168.30.10

```
/etc/sysconfig/network-scripts/route-eth2
```

```
ADDRESS0=192.168.10.0
NETMASK0=255.255.255.0
GATEWAY0=192.168.30.10
```





```
[root@router-192-168-20-20 network-scripts]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.10.0    192.168.30.10   255.255.255.0   UG    102    0        0 eth2
192.168.10.123  192.168.20.10   255.255.255.255 UGH   100    0        0 eth0
192.168.20.0    0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.30.0    0.0.0.0         255.255.255.0   U     102    0        0 eth2
192.168.80.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1
[root@router-192-168-20-20 network-scripts]# 
```





###  测试策略路由





#### 192.168.10.123

```
[root@localhost ~]# tracepath 192.168.80.123
 1?: [LOCALHOST]                                         pmtu 1500
 1:  gateway                                               0.408ms 
 1:  gateway                                               0.525ms 
 2:  192.168.20.20                                         1.004ms 
 3:  192.168.80.123                                        1.521ms !H
     Resume: pmtu 1500 
[root@localhost ~]# 
```





#### 192.168.100.124

```
[root@localhost ~]# tracepath 192.168.80.123
 1?: [LOCALHOST]                                         pmtu 1500
 1:  gateway                                               0.717ms 
 1:  gateway                                               0.952ms 
 2:  192.168.30.20                                         1.345ms 
 3:  192.168.80.123                                        2.423ms !H
     Resume: pmtu 1500 
```







####   网卡配置文件记录



##### 192.168.10.10



```
cat /etc/sysconfig/network-scripts/ifcfg-eth0 
TYPE=Ethernet
BOOTPROTO=none
IPADDR=192.168.10.10
PREFIX=24
NAME=eth0
DEVICE=eth0
ONBOOT=yes

cat /etc/sysconfig/network-scripts/ifcfg-eth1 
TYPE=Ethernet
BOOTPROTO=none
IPADDR=192.168.20.10
PREFIX=24
NAME=eth1
DEVICE=eth1
ONBOOT=yes

cat /etc/sysconfig/network-scripts/ifcfg-eth2 
TYPE=Ethernet
BOOTPROTO=none
IPADDR=192.168.30.10
PREFIX=24
NAME=eth2
DEVICE=eth2
ONBOOT=yes
```





```
[root@router-192-168-20-10 network-scripts]# ip rule show  
0:      from all lookup local 
10:     from 192.168.10.123 to 192.168.80.123 lookup dianxin 
100:    from 192.168.10.0/24 to 192.168.80.123 lookup liantong 
32766:  from all lookup main 
32767:  from all lookup default 
```



```
cat /etc/iproute2/rt_tables 
#
# reserved values
#
255     local
254     main
253     default
0       unspec

251     dianxin
252     liantong     
#
# local
#
#1      inr.ruhep
[root@router-192-168-20-10 network-scripts]# 
```









##### 192.168.20.20



```
cat /etc/sysconfig/network-scripts/ifcfg-eth0 
TYPE=Ethernet
BOOTPROTO=none
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.20.20
PREFIX=24


cat /etc/sysconfig/network-scripts/ifcfg-eth1 
TYPE=Ethernet
BOOTPROTO=none
NAME=eth1
DEVICE=eth1
ONBOOT=yes
IPADDR=192.168.80.10
PREFIX=24


cat /etc/sysconfig/network-scripts/ifcfg-eth2
TYPE=Ethernet
BOOTPROTO=none
NAME=eth2
DEVICE=eth2
ONBOOT=yes
IPADDR=192.168.30.20
PREFIX=24


cat /etc/sysconfig/network-scripts/route-eth0 
ADDRESS0=192.168.10.123
NETMASK0=255.255.255.255
GATEWAY0=192.168.20.10

cat /etc/sysconfig/network-scripts/route-eth2 
ADDRESS0=192.168.10.0
NETMASK0=255.255.255.0
GATEWAY0=192.168.30.10
```





##### 192.168.80.123



```
cat /etc/sysconfig/network-scripts/ifcfg-eth0 
TYPE=Ethernet
BOOTPROTO=none
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.80.123
PREFIX=24
GATEWAY=192.168.80.10
```





##### 192.168.10.123



```
cat /etc/sysconfig/network-scripts/ifcfg-eth0 
TYPE=Ethernet
BOOTPROTO=none
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.10.123
PREFIX=24
GATEWAY=192.168.10.10
```





##### 192.168.10.124



```
cat  /etc/sysconfig/network-scripts/ifcfg-eth0 
TYPE=Ethernet
BOOTPROTO=none
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.10.124
PREFIX=24
GATEWAY=192.168.10.10
```







