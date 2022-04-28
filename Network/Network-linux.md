

[toc]





# ip link help



```
TYPE := { vlan | veth | vcan | vxcan | dummy | ifb | macvlan | macvtap |
           bridge | bond | team | ipoib | ip6tnl | ipip | sit | vxlan |
           gre | gretap | erspan | ip6gre | ip6gretap | ip6erspan |
           vti | nlmon | team_slave | bond_slave | bridge_slave |
           ipvlan | ipvtap | geneve | vrf | macsec | netdevsim | rmnet |
           xfrm }
```





#  tun 设备



工作在 L3 层

首发IP包



# tap 设备



工作在L2 层

收发链路层数据包







```
ip tuntap add dev tap0 mod tap # 创建 tap 
```





# veth pair



## 1    测试 veth  pair 两个接口 在同一个ns 



```
# 创建 veth pair 设备
ip  link add  veth01  type veth peer name veth02
#  配置ip 地址
ip  addr add  10.1.1.1/24 dev veth01
ip  addr add  10.1.1.2/24 dev veth02
# 启动 设备
ip  link set  up veth01 up
ip  link set  up veth02 up
```





 

```
#指定源网卡 ping 对方 ip  不通
ping -I veth01 10.1.1.2
ping -I veth02 10.1.1.1
```





```
#指定源ip ping 对方 ip 通
ping -I 10.1.1.1 10.1.1.2
ping -I 10.1.1.2 10.1.1.1


## 指定源ip ping 对方 ip 时 

#lo 有抓到数据
tcpdump  -n -i lo   icmp

#  veth01 veth02  没有抓到数据
tcpdump  -n -i veth01    icmp
tcpdump  -n -i veth02    icmp

# 不需要  ip_forward
echo 0 > /proc/sys/net/ipv4/ip_forward
```





```
#指定源网卡 ping 对方 ip
ping -I veth01 10.1.1.2  -c 2 
ping -I veth02 10.1.1.1  -c 2
```







## 2  一个root ns  一个 非root ns



```
# 创建  ns 
ip netns add netns1


# 创建 veth pair 设备
ip  link add  veth01  type veth peer name veth02

# 将  veth02 置于 netns1
ip link set veth02 netns  netns1

#  配置ip 地址
ip  addr add  10.1.1.1/24 dev veth01
ip netns exec  netns1  ip  addr add  10.1.1.2/24 dev veth02

# 启动 设备
ip  link set  up veth01 up
ip netns exec  netns1 ip  link set  up veth02 up
```



```
# 在root ns
ping  -I veth01 10.1.1.2
```



```
#在netns1 ns
ip netns exec  netns1 ping  10.1.1.1
```



## 3 用网桥 连接两个ns





```
# 创建  ns 
ip netns add netns1

# 创建 veth pair 设备
ip  link add  netns1-veth01  type veth peer name netns1-veth02

# 将  netns1-veth02 置于 netns1
ip link set netns1-veth02 netns  netns1

#  配置ip 地址
ip netns exec  netns1  ip  addr add  10.1.1.1/24 dev netns1-veth02

# 启动 设备
ip netns exec  netns1 ip  link set  up netns1-veth02 up
ip  link set  netns1-veth01 up
```





```
# 创建  ns 
ip netns add netns2

# 创建 veth pair 设备
ip  link add  netns2-veth01  type veth peer name netns2-veth02

# 将  netns2-veth02 置于 netns2
ip link set netns2-veth02 netns  netns2

#  配置ip 地址
ip netns exec  netns2  ip  addr add  10.1.1.2/24 dev netns2-veth02

# 启动 设备
ip netns exec  netns2 ip  link set  up netns2-veth02 up
ip  link set  netns2-veth01 up
```



```
#创建网桥
ip link add br0 type bridge
ip link set br0 up
ip link set dev netns1-veth01  master br0
ip link set dev netns2-veth01  master br0
```









```
#查看网络情况

ip netns exec  netns1  ip a

1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: netns1-veth02@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether aa:1b:f8:53:a1:a1 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.1.1.1/24 scope global netns1-veth02
       valid_lft forever preferred_lft forever
    inet6 fe80::a81b:f8ff:fe53:a1a1/64 scope link 
       valid_lft forever preferred_lft forever
[root@localhost ~]# 
```





```
#查看网络情况
ip netns exec  netns2  ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
5: netns2-veth02@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 2e:17:26:3a:01:d4 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.1.1.2/24 scope global netns2-veth02
       valid_lft forever preferred_lft forever
    inet6 fe80::2c17:26ff:fe3a:1d4/64 scope link 
       valid_lft forever preferred_lft forever
[root@localhost ~]# 
```





```
# 测试 ping
ip netns exec  netns1  ping  10.1.1.2
ip netns exec  netns2  ping  10.1.1.1
```











# vrf 设备





# vxlan 设备



VTEP 

VXLAN Tunnel Endpoints，VXLAN隧道端点



##  点对点模式





## 多播模式







## nolearning 模式 



多点通信 nolearning 模式



https://blog.csdn.net/ledrsnet/article/details/121309645











