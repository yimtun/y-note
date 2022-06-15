

[toc]





# Coding

## C

## Go

### 1 Go-tools



### 2 Go-basic [Go-basic](Coding/Go/Go-basic.md)







## Java

### 1  Java-tools

### 2 Java-basic








# OperatingSystem

## Linux

 ### Ubuntu[Ubuntu](OperatingSystem/Linux/Ubuntu.md)

 ### cli

  

## MacOS



## Windows



# Storage





# Database





# Network

## Network-Command-line [Network-Command-line](Network/Network-Command-line.md)

## Network-linux [Network-linux](Network/Network-linux.md)

## Network-Basic [Network-Basic](Network/Network-Basic.md)

### vethPair [vethPair](Network/Network-Basic/vethPair/vethPair.md)

###  vlan [vlan](Network/Network-Basic/vlan/vlan.md)

### vxlan [vxlan](Network/Network-Basic/vxlan/vxlan.md)

## Docker-Network [Docker-Network](Network/Docker-Network.md)

## eNSP [eNSP](Network/eNSP.md)

## kubernetes-network

### kubernetes-network[kubernetes-network](Network/kubernetes-network/kubernetes-network.md)

### k8s1.23.5 [2022-04-19](Network/kubernetes-network/2022-04-19.md)

## iptables [iptables](Network/iptables/iptables.md)











#  Kubernetes

## Command-line [Command-line](Kubernetes/Command-line.md)

## quickStart

### ubuntu-k8s-v1.23.2 [all-in-one](Kubernetes/quickStart/ubuntu-k8s-v1.23.2.md)

### ubuntu-k8s-v1.23.2 [3-nodes](Kubernetes/quickStart/3-nodes/3-nodes.md)

### ubuntu-k8s-v1.23.5-flannel-vxlan [ubuntu-k8s-v1.23.5-flannel-vxlan ](Kubernetes/quickStart/ubuntu-k8s-v1.23.5-flannel-vxlan/ubuntu-k8s-v1.23.5-flannel-vxlan.md)

### ubuntu-k8s-v1.23.5-cilium-vxlan [ubuntu-k8s-v1.23.5-cilium-vxlan ](Kubernetes/quickStart/ubuntu-k8s-v1.23.5-cilium-vxlan/ubuntu-k8s-v1.23.5-cilium-vxlan.md)


### cilium-vxlan-HostRoutingbpf [cilium-vxlan-HostRoutingbpf](Kubernetes/quickStart/cilium-vxlan-HostRoutingbpf/cilium-vxlan-HostRoutingbpf.md)

### calico-ipip [calico-ipip](Kubernetes/quickStart/calico-ipip/calico-ipip.md)

### calico-bgp-Cross-subnet [calico-bgp-Cross-subnet](Kubernetes/quickStart/calico-bgp-Cross-subnet/calico-bgp-Cross-subnet.md)











# cluster-manager







## lvs [lvs](cluster-manager/lvs/lvs.md)





## pacemaker












#  Bigdata





# Virtualization



## kvm [kvm](Virtualization/kvm/kvm.md)



## docker



```
docker run -d --privileged --name nginx centos:7 /usr/sbin/init

docker exec -it nginx /bin/bash



yum install epel-release
yum -y install nginx
systemctl  enable --now  nginx
timedatectl set-timezone Asia/Shanghai

docker run -d  -p 80:80 --privileged yimtune/centos79 /usr/sbin/init
```





# Self-questioning



## network [network](Self-questioning/network/network.md)

### memory [memory](Self-questioning/memory/memory.md)

## https-and-certificate [https-and-certificate](Self-questioning/https-and-certificate/https-and-certificate.md)







