# 1 cluster network info







## vmware net info



| subnet         | vmwareNet                              | vmware net mode |
| -------------- | -------------------------------------- | --------------- |
| 11.1.0.0/24    | vmnet11   disable dhcp                 | host only       |
| 12.1.0.0/24    | vmnet12    disable dhcp                | host only       |
| 13.1.0.0/24    | vmnet13    disable dhcp                | host only       |
| 192.168.3.0/24 | is windows  network  can access public | bridge          |
|                |                                        |                 |





## linux server net info 





| host         | eth0                             | eth1        | eth2        | eth3  bgp-router-id | eth4  svc ip   |
| ------------ | -------------------------------- | ----------- | ----------- | ------------------- | -------------- |
| master       | 11.1.0.250/24  gw 11.1.0.1       | no          | no          | no                  |                |
| node         | 11.1.0.251/24  gw 11.1.0.1       | no          | no          | no                  |                |
| node         | 12.1.0.252/24  gw 12.1.0.1       | no          | no          | no                  |                |
| node         | 12.1.0.253/24  gw 12.1.0.1       | no          | no          | no                  |                |
|              |                                  |             |             |                     |                |
| linux-router | 192.168.3.254/24  gw 192.168.3.1 | 11.1.0.1/24 | 12.1.0.1/24 | 13.1.0.1/24         | 10.1.0.0/16 ？ |
|              |                                  |             |             |                     |                |
|              |                                  |             |             |                     |                |
|              |                                  |             |             |                     |                |



linux-router use  eth0  access public network

master and node  access pubic network   example  get image resources    use  linux-router













# 2   version info



| software                                              | version                                  |
| ----------------------------------------------------- | ---------------------------------------- |
| os                                                    | centos7.8  kernel 3.10.0-1127.el7.x86_64 |
| kubernetes/kubeadm                                    | v1.23.5                                  |
| calico                                                | v3.23.1                                  |
| calicoctl                                             | v3.20.5                                  |
| custom  bird  that out of cluster   (global bgp peer) | 1.6.8                                    |
|                                                       |                                          |
|                                                       |                                          |
|                                                       |                                          |
|                                                       |                                          |





# 3 linux router



Let  11.1.0.0/24  and   12.1.0.0/24 subnet  access public net   with  192.168.3.254 





```
echo "1" > /proc/sys/net/ipv4/ip_forward
```



```
iptables -t nat -A POSTROUTING -s 11.1.0.0/24  ! -d 11.1.0.0/24  -o eth0  -j SNAT --to-source 192.168.3.254
iptables -t nat -A POSTROUTING -s 12.1.0.0/24  ! -d 12.1.0.0/24  -o eth0  -j SNAT --to-source 192.168.3.254
```







# 4  init cluster



## master 11.1.0.250



```
cat > /opt/kube-1.23.5-config.yaml << EOF
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 0.0.0.0
  bindPort: 6443
nodeRegistration:
  name: 11.1.0.250
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: usvieq.2zg86228emymgcd4
  ttl: 2400000h0m0s
  usages:
  - signing
  - authentication
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.23.5
imageRepository: registry.aliyuncs.com/google_containers
controlPlaneEndpoint: 11.1.0.250:6443
controllerManager: {}
scheduler: {}
networking:
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
EOF
```





```
kubeadm  init --config  /opt/kube-1.23.5-config.yaml
```



## node 11.1.0.251  

```
kubeadm join 11.1.0.250:6443 --node-name=11.1.0.251 \
--token usvieq.2zg86228emymgcd4 \
--discovery-token-ca-cert-hash sha256:e7e45388ef5186a2f74b92164944bba093ddf4985256636ffa8c14e2f775ae25 
```



## node  12.1.0.252   

```
kubeadm join 11.1.0.250:6443 --node-name=12.1.0.252 \
--token usvieq.2zg86228emymgcd4 \
--discovery-token-ca-cert-hash sha256:e7e45388ef5186a2f74b92164944bba093ddf4985256636ffa8c14e2f775ae25 
```



## node 12.1.0.253

```
kubeadm join 11.1.0.250:6443 --node-name=12.1.0.253 \
--token usvieq.2zg86228emymgcd4 \
--discovery-token-ca-cert-hash sha256:e7e45388ef5186a2f74b92164944bba093ddf4985256636ffa8c14e2f775ae25 
```





# 5 install calico





## install  calico cni



```
curl  https://projectcalico.docs.tigera.io/v3.23/manifests/calico.yaml -O
```



```
vim calico.yaml

4405             - name: CALICO_IPV4POOL_IPIP
4406               value: "Never"
4407             # Enable or Disable VXLAN on the default IP pool.
4408             - name: CALICO_IPV4POOL_VXLAN
4409               value: "Never"
4410             # Enable or Disable VXLAN on the default IPv6 IP pool.
4411             - name: CALICO_IPV6POOL_VXLAN
4412               value: "Never"
```



```
kubectl apply -f  calico.yaml
```



## verify  no ipvlan and  no vxlan

```
kubectl  -n kube-system   get ippools  default-ipv4-ippool  -o  json  | jq .spec

{
  "allowedUses": [
    "Workload",
    "Tunnel"
  ],
  "blockSize": 26,
  "cidr": "10.244.0.0/16",
  "ipipMode": "Never",
  "natOutgoing": true,
  "nodeSelector": "all()",
  "vxlanMode": "Never"
}
```







## verify node status ready



```
kubectl  get node
NAME         STATUS   ROLES                  AGE     VERSION
11.1.0.250   Ready    control-plane,master   31m     v1.23.5
11.1.0.251   Ready    <none>                 14m     v1.23.5
12.1.0.252   Ready    <none>                 8m36s   v1.23.5
12.1.0.253   Ready    <none>                 8m17s   v1.23.5
```



## install calicoctl



```
curl -O -L https://github.com/projectcalico/calicoctl/releases/download/v3.20.5/calicoctl-linux-amd64
```

```
mv calicoctl-linux-amd64  /usr/local/bin/calicoctl
chmod +x /usr/local/bin/calicoctl 
```



### get  bgp info



```
calicoctl  node status

Calico process is running.

IPv4 BGP status
+--------------+-------------------+-------+----------+-------------+
| PEER ADDRESS |     PEER TYPE     | STATE |  SINCE   |    INFO     |
+--------------+-------------------+-------+----------+-------------+
| 11.1.0.251   | node-to-node mesh | up    | 06:38:53 | Established |
| 12.1.0.252   | node-to-node mesh | up    | 06:39:02 | Established |
| 12.1.0.253   | node-to-node mesh | up    | 06:39:04 | Established |
+--------------+-------------------+-------+----------+-------------+

IPv6 BGP status
No IPv6 peers found.
```





# 6 delete  taint



```
kubectl taint node  11.1.0.250    node-role.kubernetes.io/master:NoSchedule-
```



# 7 test pod network





##  create deployment



```
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: yimtune/nginx:1.21.6
        ports:
        - containerPort: 80
EOF
```



## create service 



```
kubectl  -n default expose deploy  nginx-deployment  --port=80 --target-port=80 --type=NodePort
```





## get info



```
kubectl  get pod -l app=nginx -o wide

NAME                                READY   STATUS    RESTARTS   AGE   IP               NODE         
nginx-deployment-6f8cbdc6f5-6sslr   1/1     Running   0          23s   10.244.240.64    12.1.0.253
nginx-deployment-6f8cbdc6f5-8cnnq   1/1     Running   0          23s   10.244.187.0     11.1.0.250
nginx-deployment-6f8cbdc6f5-ggsmq   1/1     Running   0          23s   10.244.223.0     12.1.0.252
nginx-deployment-6f8cbdc6f5-wnbt5   1/1     Running   0          23s   10.244.229.195   11.1.0.251  
```





```
curl 10.244.240.64
curl 10.244.187.0
curl 10.244.223.0
curl 10.244.229.195
```



```
kubectl  get svc -l app=nginx
NAME               TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-deployment   NodePort   10.98.159.204   <none>        80:32014/TCP   8m17s
```



```
curl  10.98.159.204
```





# 8  get  each  node  pod subnet



```
kubectl get ipamblocks.crd.projectcalico.org -o jsonpath="{range .items[*]}{'podNetwork: '}{.spec.cidr}{'\t NodeIP: '}{.spec.affinity}{'\n'}"

```



```
podNetwork: 10.244.187.0/26      NodeIP: host:11.1.0.250
podNetwork: 10.244.223.0/26      NodeIP: host:12.1.0.252
podNetwork: 10.244.229.192/26    NodeIP: host:11.1.0.251
podNetwork: 10.244.240.64/26     NodeIP: host:12.1.0.253
```









use json



```
kubectl get ipamblocks.crd.projectcalico.org -o json | jq '.items[]'
```



```
kubectl get ipamblocks.crd.projectcalico.org -o json | jq '.items[].spec.cidr'

"10.244.187.0/26"
"10.244.223.0/26"
"10.244.229.192/26"
"10.244.240.64/26"
```



```
kubectl get ipamblocks.crd.projectcalico.org -o json | jq '.items[].spec.affinity'

"host:11.1.0.250"
"host:12.1.0.252"
"host:11.1.0.251"
"host:12.1.0.253"
```







```
kubectl get ipamblocks.crd.projectcalico.org -ojson|jq '.items[]|{subnet: .spec.cidr, nodeInfo: .spec.affinity}'

```





```
{
  "subnet": "10.244.187.0/26",
  "nodeInfo": "host:11.1.0.250"
}
{
  "subnet": "10.244.223.0/26",
  "nodeInfo": "host:12.1.0.252"
}
{
  "subnet": "10.244.229.192/26",
  "nodeInfo": "host:11.1.0.251"
}
{
  "subnet": "10.244.240.64/26",
  "nodeInfo": "host:12.1.0.253"
}
```





output  json array 



```
kubectl get ipamblocks.crd.projectcalico.org -ojson|jq  '.items[]|{subnet: .spec.cidr, nodeInfo: .spec.affinity}'  | jq -s
[
  {
    "subnet": "10.244.187.0/26",
    "nodeInfo": "host:11.1.0.250"
  },
  {
    "subnet": "10.244.223.0/26",
    "nodeInfo": "host:12.1.0.252"
  },
  {
    "subnet": "10.244.229.192/26",
    "nodeInfo": "host:11.1.0.251"
  },
  {
    "subnet": "10.244.240.64/26",
    "nodeInfo": "host:12.1.0.253"
  }
]
```







#  9  use static route  for pod network







##  auto add pod route shell



### router linux  192.168.3.254



run a shell to  monitor sub and node  info  and add static route





```
```



```
ip route add  10.244.223.0/26  via 12.1.0.252
```







```
cat calico-bgp-cross-subnet-static-route-manager.sh

#!/bin/bash
export routeInfo=$(kubectl get ipamblocks.crd.projectcalico.org  -o json|jq  '.items[]|{subnet: .spec.cidr, nodeInfo: .spec.affinity}'   | jq -s)

export routeNumber=$(echo $routeInfo| jq length)


for ((i=0;i<$routeNumber;i++));do 
sub=$(echo  $routeInfo | jq .[$i].subnet | sed -e 's@\"@@g')
host=$(echo $routeInfo | jq .[$i].nodeInfo| sed 's/host://g' | sed -e 's@\"@@g')

#echo $sub $host
ip route add  $sub  via $host
done
```







```
before  add pod sub net route 

route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.3.1     0.0.0.0         UG    0      0        0 eth0
11.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth1
12.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth2
13.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth3
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1
169.254.0.0     0.0.0.0         255.255.0.0     U     1004   0        0 eth2
169.254.0.0     0.0.0.0         255.255.0.0     U     1005   0        0 eth3
192.168.3.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0



add pod sub net route 
sh calico-bgp-cross-subnet-static-route-manager.sh 




after   add pod sub net route 
route -n

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.3.1     0.0.0.0         UG    0      0        0 eth0
10.244.187.0    11.1.0.250      255.255.255.192 UG    0      0        0 eth1
10.244.223.0    12.1.0.252      255.255.255.192 UG    0      0        0 eth2
10.244.229.192  11.1.0.251      255.255.255.192 UG    0      0        0 eth1
10.244.240.64   12.1.0.253      255.255.255.192 UG    0      0        0 eth2
11.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth1
12.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth2
13.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth3
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1
169.254.0.0     0.0.0.0         255.255.0.0     U     1004   0        0 eth2
169.254.0.0     0.0.0.0         255.255.0.0     U     1005   0        0 eth3
192.168.3.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
```





##    test pod ip  on router linux



```
kubectl get pods -lapp=nginx -ogo-template --template='{{range .items}}{{printf "%s\n" .status.podIP}}{{end}}'
 
```



```
10.244.240.64
10.244.187.0
10.244.223.0
10.244.229.195
```







```
for line in `kubectl get pods -lapp=nginx -ogo-template --template='{{range .items}}{{printf "%s\n" .status.podIP}}{{end}}'`; do curl  -L -s -o /dev/null -w "%{http_code}" $line && echo ":ok"; done

```



```
200:ok
200:ok
200:ok
200:ok
```





##   test pod ip  on router linux on  11.1.0.250



```
for line in `kubectl get pods -lapp=nginx -ogo-template --template='{{range .items}}{{printf "%s\n" .status.podIP}}{{end}}'`; do  traceroute  $line&& echo "" ; done

```



```
traceroute to 10.244.240.64 (10.244.240.64), 30 hops max, 60 byte packets
 1  gateway (11.1.0.1)  0.351 ms  0.250 ms  0.186 ms
 2  12.1.0.253 (12.1.0.253)  1.131 ms  1.023 ms  0.947 ms
 3  10.244.240.64 (10.244.240.64)  0.775 ms  1.756 ms  1.657 ms

traceroute to 10.244.187.0 (10.244.187.0), 30 hops max, 60 byte packets
 1  10.244.187.0 (10.244.187.0)  0.062 ms  0.024 ms  0.023 ms

traceroute to 10.244.223.0 (10.244.223.0), 30 hops max, 60 byte packets
 1  gateway (11.1.0.1)  38.599 ms * *
 2  12.1.0.252 (12.1.0.252)  38.451 ms  38.411 ms  38.370 ms
 3  10.244.223.0 (10.244.223.0)  38.330 ms  38.290 ms  38.249 ms

traceroute to 10.244.229.195 (10.244.229.195), 30 hops max, 60 byte packets
 1  11.1.0.251 (11.1.0.251)  3.049 ms  2.971 ms  2.927 ms
 2  10.244.229.195 (10.244.229.195)  2.888 ms  2.848 ms  2.809 ms

```





##  test pod ip  on router linux on  12.1.0.252



```
for line in `kubectl get pods -lapp=nginx -ogo-template --template='{{range .items}}{{printf "%s\n" .status.podIP}}{{end}}'`; do  traceroute  $line&& echo "" ; done

```





```
traceroute to 10.244.240.64 (10.244.240.64), 30 hops max, 60 byte packets
 1  12.1.0.253 (12.1.0.253)  0.318 ms  0.290 ms  0.233 ms
 2  10.244.240.64 (10.244.240.64)  0.630 ms  0.617 ms  0.571 ms

traceroute to 10.244.187.0 (10.244.187.0), 30 hops max, 60 byte packets
 1  gateway (12.1.0.1)  0.287 ms  0.218 ms  0.170 ms
 2  11.1.0.250 (11.1.0.250)  0.416 ms  0.505 ms  0.439 ms
 3  10.244.187.0 (10.244.187.0)  0.366 ms  0.419 ms  0.519 ms

traceroute to 10.244.223.0 (10.244.223.0), 30 hops max, 60 byte packets
 1  10.244.223.0 (10.244.223.0)  0.089 ms  0.037 ms  0.034 ms

traceroute to 10.244.229.195 (10.244.229.195), 30 hops max, 60 byte packets
 1  gateway (12.1.0.1)  0.491 ms  0.393 ms  0.211 ms
 2  11.1.0.251 (11.1.0.251)  4.046 ms  3.951 ms  3.785 ms
 3  10.244.229.195 (10.244.229.195)  3.552 ms  4.436 ms  4.233 ms
```





## summary 

calico bgp mode cross subnet 

calico only provider a map 

but   only  map cant't  realy reach

must have  real road 



if  pod's   node is same subnet  pod use L2

if  pod's   node is not  same subnet  pod use L3 by linux-router 





# 10  bird as globalBgpReplaceStaticRoute





##   on k8s cluster crate global bgp 





before add



```
calicoctl  node status
Calico process is running.

IPv4 BGP status
+--------------+-------------------+-------+------------+-------------+
| PEER ADDRESS |     PEER TYPE     | STATE |   SINCE    |    INFO     |
+--------------+-------------------+-------+------------+-------------+
| 11.1.0.251   | node-to-node mesh | up    | 2022-06-14 | Established |
| 12.1.0.252   | node-to-node mesh | up    | 2022-06-14 | Established |
| 12.1.0.253   | node-to-node mesh | up    | 2022-06-14 | Established |
```



global ibgp peer



```
cat > 13.1.0.1-bird-ibgp.yaml  << EOF
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: bird-ibgp-13-1-0-1
spec:
  peerIP: 13.1.0.1
  asNumber: 64512
EOF
```







```
calicoctl  create -f 13.1.0.1-bird-ibgp.yaml --allow-version-mismatch
```







after add global bgp peer 



```
[root@11-1-0-250 ~]# calicoctl  node status
Calico process is running.

IPv4 BGP status
+--------------+-------------------+-------+------------+--------------------------------+
| PEER ADDRESS |     PEER TYPE     | STATE |   SINCE    |              INFO              |
+--------------+-------------------+-------+------------+--------------------------------+
| 11.1.0.251   | node-to-node mesh | up    | 2022-06-14 | Established                    |
| 12.1.0.252   | node-to-node mesh | up    | 2022-06-14 | Established                    |
| 12.1.0.253   | node-to-node mesh | up    | 2022-06-14 | Established                    |
| 13.1.0.1     | global            | start | 07:36:56   | Active Socket: Connection      |
|              |                   |       |            | refused                        |
+--------------+-------------------+-------+------------+--------------------------------+
```





##  config external  ibgp peer







### dell  static route for pod subnet



```
cat del-pod-subnet-static-route.sh 
```



```
#!/bin/bash
export routeInfo=$(kubectl get ipamblocks.crd.projectcalico.org  -o json|jq  '.items[]|{subnet: .spec.cidr, nodeInfo: .spec.affinity}'   | jq -s)

export routeNumber=$(echo $routeInfo| jq length)


for ((i=0;i<$routeNumber;i++));do 
sub=$(echo  $routeInfo | jq .[$i].subnet | sed -e 's@\"@@g')
host=$(echo $routeInfo | jq .[$i].nodeInfo| sed 's/host://g' | sed -e 's@\"@@g')

#echo $sub $host
#ip route add  $sub  via $host
ip route del  $sub  via $host
done
```



```
sh del-pod-subnet-static-route.sh
```



check pod subnet static is deleted



```
route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.3.1     0.0.0.0         UG    0      0        0 eth0
11.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth1
12.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth2
13.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth3
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1
169.254.0.0     0.0.0.0         255.255.0.0     U     1004   0        0 eth2
169.254.0.0     0.0.0.0         255.255.0.0     U     1005   0        0 eth3
192.168.3.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
```













### config bird ibgp





```
cat  /etc/bird.conf
```



```
cat >  /etc/bird.conf  << EOF
function apply_communities ()
{
}

######### include "/opt/bird_aggr.cfg";
protocol static {
   # IP blocks for this host.
   #route 10.244.229.192/26 blackhole;
}


# Aggregation of routes on this host; export the block, nothing beneath it.
function calico_aggr ()
{
      # Block 10.244.229.192/26 is confirmed
      #if ( net = 10.244.229.192/26 ) then { accept; }
      #if ( net ~ 10.244.229.192/26 ) then { reject; }
}

#########  include "/opt/bird_ipam.cfg";
function reject_disabled_pools ()
{
}

filter calico_export_to_bgp_peers {
  reject_disabled_pools();
  apply_communities();
  calico_aggr();

  if ( net ~ 10.244.0.0/16 ) then {
    accept;
  }
  reject;
}



filter calico_kernel_programming {
  if ( net ~ 10.244.0.0/16 ) then {
    #krt_tunnel = "";
    accept;
  }

  accept;
}
###### 

log syslog all;

router id 13.1.0.1;


# Configure synchronization between routing tables and kernel.


protocol kernel {
  learn;
  #persist; 
  scan time 2; 
  import all;
  export filter calico_kernel_programming; # Default is export none
  graceful restart;  
  merge paths on; 
}

protocol device {
  debug { states };
  scan time 2;    # Scan interfaces every 2 seconds
}

protocol direct {
  debug { states };
  interface "*";
}

template bgp bgp_template {
  debug { states };
  description "Connection to BGP peer";
  local as 64512;
  multihop;
  gateway recursive; 
  import all;
  export filter calico_export_to_bgp_peers;  # Only want to export routes for workloads.
  add paths on;
  graceful restart;
  connect delay time 2;
  connect retry time 5;
  error wait time 5,30;
}


protocol bgp Mesh_11_1_0_250 from bgp_template {
  neighbor 11.1.0.250 as 64512;
  source address 11.1.0.1; 
  #passive on;
}

protocol bgp Mesh_11_1_0_251 from bgp_template {
  neighbor 11.1.0.251 as 64512;
  source address 11.1.0.1; 
  #passive on;
}


protocol bgp Mesh_12_1_0_252 from bgp_template {
  neighbor 12.1.0.252 as 64512;
  source address 12.1.0.1; 
  #passive on;
}

protocol bgp Mesh_12_1_0_253 from bgp_template {
  neighbor 12.1.0.253 as 64512;
  source address 12.1.0.1; 
  #passive on;
}
EOF
```





```
systemctl  start  bird
```



get route 



```
[root@bgp-router ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.3.1     0.0.0.0         UG    0      0        0 eth0
10.244.187.0    11.1.0.250      255.255.255.192 UG    0      0        0 eth1
10.244.223.0    12.1.0.252      255.255.255.192 UG    0      0        0 eth2
10.244.229.192  11.1.0.251      255.255.255.192 UG    0      0        0 eth1
10.244.240.64   12.1.0.253      255.255.255.192 UG    0      0        0 eth2
11.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth1
12.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth2
13.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth3
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1
169.254.0.0     0.0.0.0         255.255.0.0     U     1004   0        0 eth2
169.254.0.0     0.0.0.0         255.255.0.0     U     1005   0        0 eth3
192.168.3.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
```





##  cat  bgp  status



```
[root@11-1-0-250 ~]# calicoctl  node status
Calico process is running.

IPv4 BGP status
+--------------+-------------------+-------+------------+-------------+
| PEER ADDRESS |     PEER TYPE     | STATE |   SINCE    |    INFO     |
+--------------+-------------------+-------+------------+-------------+
| 11.1.0.251   | node-to-node mesh | up    | 2022-06-14 | Established |
| 12.1.0.252   | node-to-node mesh | up    | 2022-06-14 | Established |
| 12.1.0.253   | node-to-node mesh | up    | 2022-06-14 | Established |
| 13.1.0.1     | global            | up    | 07:46:17   | Established |
```







```
[root@11-1-0-251 ~]# calicoctl  node status
Calico process is running.

IPv4 BGP status
+--------------+-------------------+-------+------------+-------------+
| PEER ADDRESS |     PEER TYPE     | STATE |   SINCE    |    INFO     |
+--------------+-------------------+-------+------------+-------------+
| 11.1.0.250   | node-to-node mesh | up    | 2022-06-14 | Established |
| 12.1.0.252   | node-to-node mesh | up    | 2022-06-14 | Established |
| 12.1.0.253   | node-to-node mesh | up    | 2022-06-14 | Established |
| 13.1.0.1     | global            | up    | 07:46:16   | Established |
+--------------+-------------------+-------+------------+-------------+
```





```
[root@12-1-0-252 ~]#  calicoctl  node status
Calico process is running.

IPv4 BGP status
+--------------+-------------------+-------+------------+-------------+
| PEER ADDRESS |     PEER TYPE     | STATE |   SINCE    |    INFO     |
+--------------+-------------------+-------+------------+-------------+
| 11.1.0.250   | node-to-node mesh | up    | 2022-06-14 | Established |
| 11.1.0.251   | node-to-node mesh | up    | 2022-06-14 | Established |
| 12.1.0.253   | node-to-node mesh | up    | 2022-06-14 | Established |
| 13.1.0.1     | global            | up    | 07:46:15   | Established |
+--------------+-------------------+-------+------------+-------------+
```





```
[root@12-1-0-253 ~]# calicoctl  node status
Calico process is running.

IPv4 BGP status
+--------------+-------------------+-------+------------+-------------+
| PEER ADDRESS |     PEER TYPE     | STATE |   SINCE    |    INFO     |
+--------------+-------------------+-------+------------+-------------+
| 11.1.0.250   | node-to-node mesh | up    | 2022-06-14 | Established |
| 11.1.0.251   | node-to-node mesh | up    | 2022-06-14 | Established |
| 12.1.0.252   | node-to-node mesh | up    | 2022-06-14 | Established |
| 13.1.0.1     | global            | up    | 07:46:16   | Established |
+--------------+-------------------+-------+------------+-------------+
```



## test pod subnet access each other





```
kubectl  get pod -l app=nginx -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP               NODE         NOMINATED NODE   READINESS GATES
nginx-deployment-6f8cbdc6f5-6sslr   1/1     Running   0          24h   10.244.240.64    12.1.0.253   <none>           <none>
nginx-deployment-6f8cbdc6f5-8cnnq   1/1     Running   0          24h   10.244.187.0     11.1.0.250   <none>           <none>
nginx-deployment-6f8cbdc6f5-ggsmq   1/1     Running   0          24h   10.244.223.0     12.1.0.252   <none>           <none>
nginx-deployment-6f8cbdc6f5-wnbt5   1/1     Running   0          24h   10.244.229.195   11.1.0.251   <none>           <none>
```







```
[root@11-1-0-250 ~]# for line in `kubectl get pods -lapp=nginx -ogo-template --template='{{range .items}}{{printf "%s\n" .status.podIP}}{{end}}'`; do curl  -L -s -o /dev/null -w "%{http_code}" $line && echo ":ok"; done

200:ok
200:ok
200:ok
200:ok


[root@11-1-0-251 ~]# for line in `kubectl get pods -lapp=nginx -ogo-template --template='{{range .items}}{{printf "%s\n" .status.podIP}}{{end}}'`; do curl  -L -s -o /dev/null -w "%{http_code}" $line && echo ":ok"; done

200:ok
200:ok
200:ok
200:ok

[root@12-1-0-252 ~]# for line in `kubectl get pods -lapp=nginx -ogo-template --template='{{range .items}}{{printf "%s\n" .status.podIP}}{{end}}'`; do curl  -L -s -o /dev/null -w "%{http_code}" $line && echo ":ok"; done
200:ok
200:ok
200:ok
200:ok

[root@12-1-0-253 ~]# for line in `kubectl get pods -lapp=nginx -ogo-template --template='{{range .items}}{{printf "%s\n" .status.podIP}}{{end}}'`; do curl  -L -s -o /dev/null -w "%{http_code}" $line && echo ":ok"; done
200:ok
200:ok
200:ok
200:ok
```





## stop external bird ibgp peer



```
[root@bgp-router ~]# systemctl stop   bird
[root@bgp-router ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.3.1     0.0.0.0         UG    0      0        0 eth0
11.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth1
12.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth2
13.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth3
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1
169.254.0.0     0.0.0.0         255.255.0.0     U     1004   0        0 eth2
169.254.0.0     0.0.0.0         255.255.0.0     U     1005   0        0 eth3
192.168.3.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
```





```
[root@11-1-0-250 ~]# calicoctl  node status
Calico process is running.

IPv4 BGP status
+--------------+-------------------+-------+------------+--------------------------------+
| PEER ADDRESS |     PEER TYPE     | STATE |   SINCE    |              INFO              |
+--------------+-------------------+-------+------------+--------------------------------+
| 11.1.0.251   | node-to-node mesh | up    | 2022-06-14 | Established                    |
| 12.1.0.252   | node-to-node mesh | up    | 2022-06-14 | Established                    |
| 12.1.0.253   | node-to-node mesh | up    | 2022-06-14 | Established                    |
| 13.1.0.1     | global            | start | 08:02:03   | Active Socket: Connection      |
|              |                   |       |            | refused                        |
+--------------+-------------------+-------+------------+--------------------------------+
```





different  node subnet can't access 



```
for line in `kubectl get pods -lapp=nginx -ogo-template --template='{{range .items}}{{printf "%s\n" .status.podIP}}{{end}}'`; do curl  -L -s -o /dev/null -w "%{http_code}" $line && echo ":ok"; done
```





# 11 realRouterReplaceBgpSoftwareBird



## router linux shutdown



```
poweroff
```

