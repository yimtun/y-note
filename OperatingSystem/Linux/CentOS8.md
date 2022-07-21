

## kvm



```
rpm -ivh https://pkgs.dyn.su/el8/base/x86_64/raven-release-1.0-2.el8.noarch.rpm
```



```
yum remove qemu-kvm-lock-issi
```



```
dnf --enablerepo=raven-extras install qemu-system-arm

dnf --enablerepo=raven-extras install qemu-kvm

dnf --enablerepo=raven-extras install qemu-img
```





```
rpm -ivh https://mirrors.huaweicloud.com/centos/8-stream/AppStream/aarch64/os/Packages/edk2-aarch64-20200602gitca407c7246bf-4.el8.noarch.rpm
```





