

# 1 创建自定义链

示例

```
iptables -t nat  -N KUBE-SERVICES
# 在nat 表中创建自定义链  KUBE-SERVICES
```



# 2 查看自定义链

```
iptables  -t nat -nL KUBE-SERVICES
# 查看nat 表 KUBE-SERVICES 链中内容
```



# 3  删除自定义链

```
iptables  -t nat -X KUBE-SERVICES
```





# 4  向自定义链中插入规则

```
iptables   -t nat  -A KUBE-MARK-MASQ -j MARK --set-xmark 0x4000/0x4000
#向nat 表 KUBE-MARK-MASQ 链添加规则
```



```
iptables -t nat  -A KUBE-LOAD-BALANCER -j KUBE-MARK-MASQ
#将nat表 自定义链 KUBE-MARK-MASQ 中的规则  插入到 nat表 KUBE-LOAD-BALANCER链中
```







在na t 表中创建自定义链 KUBE-SERVICES  和 自定义链 KUBE-MARK-MASQ



```
iptables -t nat  -N KUBE-SERVICES
iptables -t nat  -N KUBE-MARK-MASQ
iptables -t nat  -N KUBE-MARK-DROP
iptables -t nat  -N KUBE-LOAD-BALANCER
```



查看自定义表中的  自定义链链



```
iptables  -t nat -nL KUBE-SERVICES
iptables  -t nat -nL  KUBE-MARK-MASQ
iptables  -t nat -nL KUBE-MARK-DROP
```



在自定义链中插入规则



```
iptables   -t nat  -A KUBE-MARK-MASQ -j MARK --set-xmark 0x4000/0x4000
# 对 nat 表  KUBE-MARK-MASQ 链  
```



查看  nat 表  KUBE-MARK-MASQ 链

```
iptables  -t nat -nL  KUBE-MARK-MASQ

Chain KUBE-MARK-MASQ (2 references) 被关联的链的个数
target     prot opt source               destination         
MARK       all  --  0.0.0.0/0            0.0.0.0/0            MARK or 0x4000
```



```
iptables   -t nat  -A KUBE-MARK-DROP -j MARK --set-xmark 0x8000/0x8000
```



```
iptables   -t nat  -A PREROUTING -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
```





## KUBE-LOAD-BALANCER  链添加规则



```
iptables -t nat  -A KUBE-LOAD-BALANCER -j KUBE-MARK-MASQ
```



```
iptables -t nat  -nL KUBE-LOAD-BALANCER 

Chain KUBE-LOAD-BALANCER (0 references)
target          prot opt source               destination         
KUBE-MARK-MASQ  all  --  0.0.0.0/0            0.0.0.0/0       
```









## KUBE-SERVICES   链添加规则





### 先创建ipset



查看示例



```
kubectl  get  svc -n default  kubernetes
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   48d
```



ipset 内容是  clusterip:端口

```
kubectl   -n kube-system exec -it kube-proxy-wj779 -- ipset list KUBE-CLUSTER-IP
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
```



```
KUBE-CLUSTER-IP
```





```
ipset create   KUBE-CLUSTER-IP hash:ip,port
```



向 ipset  添加内容

```
ipset add    KUBE-CLUSTER-IP  10.96.0.1,tcp:443
```







向 nat 表 KUBE-SERVICES 中添加规则



```
iptables -t nat -A KUBE-SERVICES \
! -s 10.244.0.0/16 \
-m comment --comment "Kubernetes service cluster ip + port for masquerade purpose" \
-m set --match-set KUBE-CLUSTER-IP dst,dst -j KUBE-MARK-MASQ
```







向 nat 表 KUBE-SERVICES 中添加第二个规则

```
iptables -t nat -A KUBE-SERVICES -m set --match-set KUBE-CLUSTER-IP dst,dst -j ACCEPT
```



添加规则后查看



```
iptables -t nat -nL KUBE-SERVICES
```



```
Chain KUBE-SERVICES (2 references)

target          prot opt   source            destination

KUBE-MARK-MASQ  all  --    !10.244.0.0/16    0.0.0.0/0        match-set KUBE-CLUSTER-IP dst,dst

/* Kubernetes service cluster ip + port for masquerade purpose */ 

ACCEPT          all  --    0.0.0.0/0         0.0.0.0/0         match-set KUBE-CLUSTER-IP dst,dst
```









##  nat 表 OUTPUT 链添加



```
iptables -t nat -A OUTPUT -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
```



```
iptables  -t nat   -nL  OUTPUT
Chain OUTPUT (policy ACCEPT)
target         prot opt source               destination         
KUBE-SERVICES  all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */
```





# 1 mark





```
root@node-172-16-100-13:~# iptables -t nat -nL



Chain PREROUTING (policy ACCEPT)  dnat

target     prot opt source               destination         
KUBE-SERVICES  all  --  0.0.0.0/0            0.0.0.0/0            
/* kubernetes service portals */

       


Chain OUTPUT (policy ACCEPT)

target     prot opt source               destination         
KUBE-SERVICES  all  --  0.0.0.0/0        0.0.0.0/0            

/* kubernetes service portals */




Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
KUBE-POSTROUTING  all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes postrouting rules */
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0           
RETURN     all  --  10.244.0.0/16        10.244.0.0/16        /* flanneld masq */
MASQUERADE  all  --  10.244.0.0/16       !224.0.0.0/4          /* flannel masq */ random-fully
RETURN     all  -- !10.244.0.0/16        10.244.0.0/24        /* flanneld masq */
MASQUERADE  all  -- !10.244.0.0/16        10.244.0.0/16        /* flanneld masq */ random-fully









---- 自定义
    





Chain KUBE-POSTROUTING (1 references)
target     prot opt source               destination         
RETURN     all  --  0.0.0.0/0            0.0.0.0/0            mark match ! 0x4000/0x4000
MARK       all  --  0.0.0.0/0            0.0.0.0/0            MARK xor 0x4000
MASQUERADE  all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes service traffic requiring SNAT */ random-fully

Chain KUBE-SERVICES (2 references)
target     prot opt source               destination         
KUBE-MARK-MASQ  all  -- !10.244.0.0/16        0.0.0.0/0        match-set KUBE-CLUSTER-IP dst,dst
/* Kubernetes service cluster ip + port for masquerade purpose */ 


ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            match-set KUBE-CLUSTER-IP dst,dst









```







#   流向处理



nat 表





##  PREROUTING  链



路由前   dnat  目的地址替换       在路由前才能替换目的地址  过了路由替换就晚了



```
iptables -t  nat   -nL PREROUTING
Chain PREROUTING (policy ACCEPT)
target          prot opt source               destination         
KUBE-SERVICES   all  --  0.0.0.0/0            0.0.0.0/0            

/* kubernetes service portals */



不限制  对所有流量 执行  KUBE-SERVICES 链中的规则


```





查看 KUBE-SERVICES 链





```
iptables -t  nat   -nL KUBE-SERVICES

Chain KUBE-SERVICES (2 references)

target          prot opt  source            destination         
KUBE-MARK-MASQ  all  --   !10.244.0.0/16    0.0.0.0/0       match-set KUBE-CLUSTER-IP dst,dst


对所有  源不为10.244.0.0/16 目的为ipset KUBE-CLUSTER-IP 中定义的地址和端口 执行 KUBE-MARK-MASQ 链中的规则 



ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            match-set KUBE-CLUSTER-IP dst,dst

对所有  源不为  0.0.0.0/0 目的为ipset KUBE-CLUSTER-IP 中定义的地址和端口 执行 accept


```





查看 KUBE-MARK-MASQ 链



```
iptables -t  nat   -nL KUBE-MARK-MASQ

Chain KUBE-MARK-MASQ (2 references)
target     prot opt source               destination         
MARK       all  --  0.0.0.0/0            0.0.0.0/0            MARK or 0x4000
 
对当前链所有流量 打上 mark     MARK or 0x4000
```





##  POSTROUTING 链





POSTROUTING    路由后 做snat    在路由后做原地址替换才有意义



```
iptables -t  nat   -nL  POSTROUTING

Chain POSTROUTING (policy ACCEPT)


target            prot opt source               destination         
KUBE-POSTROUTING  all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes postrouting rules */

对所有流量。执行  KUBE-POSTROUTING 链中的规则



MASQUERADE        all  --  172.17.0.0/16        0.0.0.0/0
对源  172.17.0.0/16 目的 0.0.0.0/0  snat   匹配的是 docker 默认生产的bridge 网络  




RETURN            all  --  10.244.0.0/16        10.244.0.0/16    /* flanneld masq */
对源是 10.244.0.0/16   目的是 10.244.0.0/16    执行动作 RETURN      忽略pod内部互联的网络流量 



MASQUERADE        all  --  10.244.0.0/16      !224.0.0.0/4     /* flannel masq */ random-fully
对源  10.244.0.0/16  目的不是 !224.0.0.0/4   【224.0.0.0/4 IPv4组播地址（D类） 】 pod 网络出来的流量 做snat  



RETURN            all  -- !10.244.0.0/16        10.244.0.0/24        /* flanneld masq */

对源不是  10.244.0.0/16   目的是  10.244.0.0/24     RETURN  忽略 外部网络进入 pod 内部的流量 ?? 掩码不一样


MASQUERADE        all  -- !10.244.0.0/16        10.244.0.0/16        /* flanneld masq */ random-fully

对源不是  10.244.0.0/16 目的是  10.244.0.0/16   snat 源地址替换  pod外部网络 进入pod 网络 做源地址替换

```







查看 KUBE-POSTROUTING 链



```
iptables -t  nat   -nL   KUBE-POSTROUTING

Chain KUBE-POSTROUTING (1 references)
target     prot opt source               destination         

RETURN     all  --  0.0.0.0/0            0.0.0.0/0            mark match ! 0x4000/0x4000

不能match mark  0x4000/0x4000 的 执行  RETURN 动作


MARK       all  --  0.0.0.0/0            0.0.0.0/0            MARK xor 0x4000
执行 mark 动作



MASQUERADE  all  --  0.0.0.0/0            0.0.0.0/0         random-fully

执行 snat   做源地址替换
/* kubernetes service traffic requiring SNAT */


```





```
-A KUBE-POSTROUTING -m mark ! --mark 0x4000/0x4000 -j RETURN
-A KUBE-POSTROUTING -j MARK --set-xmark 0x4000/0x0
-A KUBE-POSTROUTING \
-m comment --comment "kubernetes service traffic requiring SNAT" \
-j MASQUERADE --random-fully
```











# ipvsadm 修改测试





```
kubectl  -n kube-system  delete  ds kube-proxy 
```



```
ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 rr
  -> 172.16.100.13:6443           Masq    1      1          0         
```



```
ipvsadm -a -t 10.96.0.1:443  -r 172.16.100.5:80 -m
```



```
ipvsadm -a -t 10.96.0.1:443  -r 172.16.100.13:6443  -m
```



```
ipvsadm  -d -t 10.96.0.1:443   -r 172.16.100.13:6443
```





```
ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 rr
  -> 172.16.100.5:80              Masq    1      0          15        
```





```
curl  10.96.0.1:443 ok
```









测试自己的 service ip



```
ip addr add 182.16.100.101/24 dev enp0s5
```



设置lvs



```
ipvsadm -A -t 182.16.100.101:8888  -s rr
```



```
ipvsadm -a -t 182.16.100.101:8888  -r 172.16.100.5:80 -m
```





对 182.16.100.101 做sant



```

```



```
iptables -t nat -I POSTROUTING 1 -s 182.16.100.101/24 ! -d 182.16.100.101/24 -j MASQUERADE --random-fully ok


iptables -t nat -I POSTROUTING 1    -j MASQUERADE --random-fully ok

curl  182.16.100.101:8888 ok



iptables -t nat -D  POSTROUTING 1   





iptables -t nat -nL POSTROUTING --line-number

Chain POSTROUTING (policy ACCEPT)
num  target     prot opt source               destination         
1    MASQUERADE  all  --  182.16.100.0/24     !182.16.100.0/24      random-fully




```



```
iptables -t nat -A POSTROUTING  -s 172.16.20.0/24   ! -d  172.16.20.0/24   -j MASQUERADE --random-fully
```





跨节点测试







```
ip addr add 182.16.100.102/24 dev enp0s5


curl  182.16.100.101:8888

ok
```







# 过后整理



添加一个  负载均衡

```
ipvsadm -A -t 100.1.0.1:80  -s rr

ipvsadm -a -t 100.1.0.1:80  -r 100.1.0.2:80 -m
ipvsadm -a -t 100.1.0.1:80  -r 100.1.0.3:80 -m
```





```
iptables -t nat -I POSTROUTING 1    -j MASQUERADE 
```







```
ip addr add 182.16.100.101/24 dev eth0

ipvsadm -A -t 182.16.100.101:80  -s rr

ipvsadm -a -t 182.16.100.101:80  -r 100.1.0.2:80 -m
ipvsadm -a -t 182.16.100.101:80  -r 100.1.0.3:80 -m



iptables -t nat -I POSTROUTING 1    -j MASQUERADE --random-fully ok


如果要使用 100.1.0.1:80 这个lvs 添加snat  还没测试

iptables -t nat -I POSTROUTING 1 -s 100.1.0.1/24 !--d 100.1.0.1/24 -j MASQUERADE 
```









# rebuild-ok



```
ip addr add 182.16.100.101/24 dev enp0s5
ipvsadm -A -t 182.16.100.101:8888  -s rr
ipvsadm -a -t 182.16.100.101:8888  -r 172.16.100.5:80 -m
```





```
iptables -t nat -I POSTROUTING 1 -s 182.16.100.101/24 ! -d 182.16.100.101/24 -j MASQUERADE --random-fully
iptables -t filter   -P  FORWARD  ACCEPT
```





```
cat > /etc/sysctl.d/full-nat.conf << EOF
net.ipv4.ip_forward = 1
net.ipv4.vs.conntrack = 1
EOF
```



```
sysctl  --system
```

