# 1 docker overlay consul vxlan







#  2 macvlan





192.168.3.201

```
docker network create -d macvlan \
  --subnet=192.168.3.0/24 \
  --gateway=192.168.3.1 \
  -o parent=ens33 \
  192-168-3-201-macvlan-net
```



```
docker run --rm -dit \
  --network 192-168-3-201-macvlan-net \
  --name 192-168-3-201-macvlan-alpine \
  alpine:latest \
  ash
```





```
docker exec 192-168-3-201-macvlan-alpine ip addr show eth0
```



```
4: eth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:c0:a8:03:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.2/24 brd 192.168.3.255 scope global eth0
       valid_lft forever preferred_lft forever
```





```
docker exec -it 192-168-3-201-macvlan-alpine  /bin/ash
```



```
ping www.baidu.com
PING www.baidu.com (110.242.68.4): 56 data bytes
64 bytes from 110.242.68.4: seq=0 ttl=53 time=22.899 ms
64 bytes from 110.242.68.4: seq=1 ttl=53 time=21.212 ms
```



192.168.3.202





```
docker network create -d macvlan \
  --subnet=192.168.3.0/24 \
  --gateway=192.168.3.1 \
  -o parent=ens33 \
  192-168-3-202-macvlan-net
```





指定ip 确保 192.168.3.3 没有被使用

```
docker run --rm -dit \
  --network 192-168-3-202-macvlan-net \
  --ip 192.168.3.3 \
  --name 192-168-3-202-macvlan-alpine \
  alpine:latest \
  ash
```







```
docker exec -it 192-168-3-202-macvlan-alpine  /bin/ash
```



```
ping www.baidu.com
```



192.168.3.203





```
docker network create -d macvlan \
  --subnet=192.168.3.0/24 \
  --gateway=192.168.3.1 \
  -o parent=ens33 \
  192-168-3-203-macvlan-net
```







指定ip 确保 192.168.3.4 没有被使用

```
docker run --rm -dit \
  --network 192-168-3-203-macvlan-net \
   --ip 192.168.3.4 \
  --name 192-168-3-203-macvlan-alpine-docker1 \
  alpine:latest \
  ash
```







指定ip 确保 192.168.3.5 没有被使用

```
docker run --rm -dit \
  --network 192-168-3-203-macvlan-net \
   --ip 192.168.3.5 \
  --name 192-168-3-203-macvlan-alpine-docker2 \
  alpine:latest \
  ash
```





```
root@node-192-168-3-203:~# docker ps
CONTAINER ID   IMAGE           COMMAND   CREATED         STATUS         PORTS     NAMES
9cd39e9e6203   alpine:latest   "ash"     4 seconds ago   Up 1 second              192-168-3-203-macvlan-alpine-docker2
75e70aade2b9   alpine:latest   "ash"     8 seconds ago   Up 5 seconds             192-168-3-203-macvlan-alpine-docker1
```







```
docker exec 192-168-3-203-macvlan-alpine-docker1 ip addr show eth0
```



```
6: eth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:c0:a8:03:04 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.4/24 brd 192.168.3.255 scope global eth0
       valid_lft forever preferred_lft forever
```



```
docker exec 192-168-3-203-macvlan-alpine-docker2 ip addr show eth0
```



```
7: eth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:c0:a8:03:05 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.5/24 brd 192.168.3.255 scope global eth0
       valid_lft forever preferred_lft forever
```









```
docker network inspect 192-168-3-203-macvlan-net
[
    {
        "Name": "192-168-3-203-macvlan-net",
        "Id": "cd69fdc4a9428a0e2b6fd404dc4fb6806f55f8eccd05847fec6b5ff17d80a4bd",
        "Created": "2022-04-24T23:23:48.836381457+08:00",
        "Scope": "local",
        "Driver": "macvlan",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.3.0/24",
                    "Gateway": "192.168.3.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "17cb3e66a7c5d11e9aabd1c46842cb638b3c407bae202d391f304b71d95db14e": {
                "Name": "192-168-3-203-macvlan-alpine-docker2",
                "EndpointID": "d996b67898f767573170beaa2bbef46e3e7f4edc18fb75a0afbe81fad3b78ab9",
                "MacAddress": "02:42:c0:a8:03:05",
                "IPv4Address": "192.168.3.5/24",
                "IPv6Address": ""
            },
            "c26b45a03177d96a31e450ad13e01ae43fddcf2f729795d82b647a0208354d81": {
                "Name": "192-168-3-203-macvlan-alpine-docker1",
                "EndpointID": "f17810ceb83491018d0b28a9174ac12233dbdc12d342854c71761c68bb81d794",
                "MacAddress": "02:42:c0:a8:03:04",
                "IPv4Address": "192.168.3.4/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "parent": "ens33"
        },
        "Labels": {}
    }
]
root@node-192-168-3-203:~# 
```





### macvlan trunked bridge







https://docs.docker.com/network/network-tutorial-macvlan/#8021q-trunked-bridge-example





