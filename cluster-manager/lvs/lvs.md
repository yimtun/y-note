

# 1 full nat-1





```
sysctl --write net.ipv4.vs.conntrack=1
sysctl --write net.ipv4.ip_forward=1
```



```
modprobe ip_vs
```



```
ipset create KUBE-CLUSTER-IP hash:ip,port
ipset add KUBE-CLUSTER-IP 100.1.0.1,80
```



```
iptables -N KUBE-SERVICES -t nat
iptables -N KUBE-MARK-MASQ -t nat
iptables -t nat -N KUBE-POSTROUTING

iptables -t nat -I POSTROUTING -m comment --comment "postrouting rule" -j KUBE-POSTROUTING
iptables -t nat -I PREROUTING -m comment --comment "proxy snat" -j LVS_PROXY_CHAIN
iptables -t nat -I KUE-SERVICES -m set --match-set KUBE-CLUSTER-IP dst,dst -j KUBE-MARK-MASQ
iptables -t nat -I KUBE-MARK-MASQ -j MARK --set-xmark 0x4000/0x4000
iptables -t nat -I KUBE-POSTROUTING -m mark --mark 0x4000/0x4000 -j MASQUERADE
```





```
ipvsadm -A -t 100.1.0.1:80 -s rr
ipvsadm -a -t 100.1.0.1:80 -r 100.1.0.2:80 -m
```





# full nat  w







```
ip link add dev dustin-ipvs0 type dummy
ip addr add 10.100.100.100/32 dev dustin-ipvs0
```





## ipvs



```
ipvsadm --add-service --tcp-service 10.100.100.100:8080 --scheduler rr
```



```
ipvsadm \
  --add-server \
  --tcp-service 10.100.100.100:8080 \
  --real-server 100.1.0.2:80 \
  --masquerading
```



```
curl  10.100.100.100:8080

ok
```





## snat 



```
ipset create DUSTIN-LOOP-BACK hash:ip,port,ip
```

目标IP 目标端口 源IP

```
ipset add DUSTIN-LOOP-BACK 100.1.0.2,tcp:80,100.1.0.1
```



```
iptables --table nat --append POSTROUTING --match set --match-set DUSTIN-LOOP-BACK dst,dst,src --jump MASQUERADE
```







```
ipset add DUSTIN-LOOP-BACK 100.1.0.2,tcp:80,100.1.0.1
```





```
ipset list xxx
```



# nat

------



```
ipvsadm -A -t 172.16.100.10:80 -s rr
```



```
ipvsadm -a -t 172.16.100.10:80 -r  172.16.100.5 -g -w 1
```





```
-g gateway dr 类型 默认

```







```
ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  172.16.100.10:80 rr
  -> 172.16.100.5:80              Route   1      0          1         
```





###  删除



```
ipvsadm -d -t 172.16.100.10:80 -r  172.16.100.5
```









forward  Masq    nat 模式

```
ipvsadm -a -t 172.16.100.10:80 -r  172.16.100.5:80 -m -w 1
```







```
ipvsadm-save  > /etc/sysconfig/ipvsadm
systemctl start   ipvsadm
```











```
#添加一个虚拟服务 10.10.1.100:9000，使用加权轮询算法
ipvsadm -A -t 172.16.100.10:9000 -s wrr
 
#添加真实服务器，使用NAT模式，权重1
ipvsadm -a -t 172.16.100.10:9000 -r 172.16.100.5:8000 -m -w 1
 
#查看转发规则
ipvsadm -Ln
```







```
ipvsadm -A -t 172.16.100.10:9000 -s wrr

```









#  内核模块





```
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack_ipv4
```









# snat





```
iptables --table nat --append POSTROUTING --source  192.168.3.0/24 --jump MASQUERADE
```









