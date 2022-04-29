#   test 1





![image-20220429153000334](./vethPair.assets/image-20220429153000334.png)











```
# 创建  ns 
ip netns add n1

# 创建 veth pair 设备
ip  link add  veth0  type veth peer name veth1

# 将  veth1  置于 n1
ip link set veth1 netns  n1

#  配置ip 地址
ip netns exec  n1  ip  addr add  10.1.1.102/24 dev veth1

# 启动 设备
ip netns exec  n1 ip  link set  veth1  up


ip  addr add  10.1.1.101/24 dev veth0
ip  link set  veth0  up


# use veth0 ping  10.1.1.102 ok
ping -I veth0 10.1.1.102

# use veth1 ping  10.1.1.101 ok 
ip netns exec  n1  ping -I veth1 10.1.1.101
```









```
# 查看 veth pair 对应关系

#  veth0 在 root ns 的编号是 4
ip a | grep veth0
4: veth0@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 10.1.1.101/24 scope global veth0

# veth1 在  n1 ns 中的编号是 3
ip netns exec n1 ip a | grep veth1
3: veth1@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 10.1.1.102/24 scope global veth1


# 查看  veth0 的对端编号
ethtool  -S veth0
NIC statistics:
     peer_ifindex: 3

# 查看 veth1 的对端编号
ip netns exec n1 ethtool  -S veth1
NIC statistics:
     peer_ifindex: 4
```

