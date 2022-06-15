# 1 free





```
free 
 
               total      used        free       shared    buff/cache   available
Mem:        8008968      308008      166068      413852     7534892      6952884
Swap:             0           0           0

```



```
total      是总内存大小；
used       是已使用内存的大小，包含了共享内存 shared；
free       是未使用内存的大小；
shared     是共享内存的大小；
buff/cache 是缓存和缓冲区的大小；
available  是新进程可用内存的大小。free未使用内存+可回收内存
```





# 2 top



```
top


VIRT    RES    SHR


VIRT    进程虚拟内存大小   进程申请过 但实际未分配物理内存的也算在内
RES     是常驻内存的大小   进程实际使用的物理内存大小 但不包括 Swap 和共享内存。
SHR     共享内存大小 
%MEM    进程使用物理内存占系统总内存的百分比
```





# 3 buffer cache



```
Buffer 是对磁盘数据的缓存  读写请求
Cache  是文件数据的缓存   读写请求
```



