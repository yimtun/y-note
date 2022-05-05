# 配置console



```
/etc/default/grub

delete  line  GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet"
add     line  GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=cl/root rd.lvm.lv=cl/swap console=tty0 console=ttyS0,115200n8"




like tihs


GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet net.ifnames=0 biosdevname=0 console=ttyS0 console=tty0"





grub2-mkconfig -o /boot/grub2/grub.cfg
```

