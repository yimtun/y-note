# tcp 三次握手







```
SYN：建立连接
FIN：关闭连接
ACK：响应
PSH：有DATA数据传输
RST：连接重置
```





```
1 客户端的协议栈向服务器端发送了 SYN 包，并告诉服务器端当前发送序列号 j，客户端进入 SYNC_SENT 状态；

2 服务器端的协议栈收到这个包之后，和客户端进行 ACK 应答，应答的值为 j+1，表示对 SYN 包 j 的确认，
  同时服务器也发送一个 SYN 包，告诉客户端当前我的发送序列号为 k，服务器端进入 SYNC_RCVD 状态；

3 客户端协议栈收到 ACK 之后，使得应用程序从 connect 调用返回，表示客户端到服务器端的单向连接建立成功，
  客户端的状态为 ESTABLISHED。
  同时客户端协议栈也会对服务器端的 SYN 包进行应答，应答数据为 k+1；

4 应答包到达服务器端后，服务器端协议栈使得 accept 阻塞调用返回，这个时候服务器端到客户端的单向连接也建立成功，
  服务器端也进入 ESTABLISHED 状态。

```



```
curl --local-port 60001 127.0.0.1:80
```



```
netstat -an  |  grep 60001
```





```
curl --local-port 60001  -silent --output nul --show-error --fail  172.16.100.3:80
```









```
tcpdump -nn -S   tcp and  \(host 172.16.100.3 or 172.16.100.4\)  and  port 80
```



```
curl   -silent --output nul --show-error --fail  172.16.100.3:80
```







```
三次握手
13:44:14.649395 IP 172.16.100.4.47132 > 172.16.100.3.22: Flags [S], seq 1228404592, win 29200, options [mss 1460,sackOK,TS val 28290070 ecr 0,nop,wscale 7], length 0
Flags [S], seq 1228404592


13:44:14.649436 IP 172.16.100.3.22 > 172.16.100.4.47132: Flags [S.], seq 43780288, ack 1228404593, win 28960, options [mss 1460,sackOK,TS val 6209991 ecr 28290070,nop,wscale 7], length 0
Flags [S.], seq 43780288, ack 1228404593


13:44:14.649936 IP 172.16.100.4.47132 > 172.16.100.3.22: Flags [.], ack 43780289, win 229, options [nop,nop,TS val 28290070 ecr 6209991], length 0
Flags [.], ack 43780289



数据传输
13:44:14.660057 IP 172.16.100.3.22 > 172.16.100.4.47132: Flags [P.], seq 43780289:43780310, ack 1228404593, win 227, options [nop,nop,TS val 6210001 ecr 28290070], length 21
Flags [P.], seq 43780289:43780310, ack 1228404593

13:44:14.660501 IP 172.16.100.4.47132 > 172.16.100.3.22: Flags [.], ack 43780310, win 229, options [nop,nop,TS val 28290081 ecr 6210001], length 0
Flags [.], ack 43780310




四次挥手
13:44:24.558180 IP 172.16.100.4.47132 > 172.16.100.3.22: Flags [F.], seq 1228404593, ack 43780310, win 229, options [nop,nop,TS val 28299979 ecr 6210001], length 0
Flags [F.], seq 1228404593, ack 43780310

seq 1228404593  FIN m


13:44:24.559286 IP 172.16.100.3.22 > 172.16.100.4.47132: Flags [.], ack 1228404594, win 227, options [nop,nop,TS val 6219901 ecr 28299979], length 0
Flags [.], ack 1228404594

ack 1228404594 m+1


13:44:24.559341 IP 172.16.100.3.22 > 172.16.100.4.47132: Flags [F.], seq 43780310, ack 1228404594, win 227, options [nop,nop,TS val 6219901 ecr 28299979], length 0
Flags [F.], seq 43780310, ack 1228404594

seq 43780310 FIN n




13:44:24.559920 IP 172.16.100.4.47132 > 172.16.100.3.22: Flags [.], ack 43780311, win 229, options [nop,nop,TS val 28299980 ecr 6219901], length 0
Flags [.], ack 43780311

ack 43780311 n+1






```







###   四次挥手



```
主机1 主动关闭



1 主机1 先发送 FIN 报文   


2 主机2 进入CLOSE_WAIT 状态，并发送一个 ACK 应答，


3 主机2 通过 read调用获得EOF，并将此结果通知应用程序进行主动关闭操作， 发送 FIN 报文。


4 主机1 在接收到 FIN 报文后发送 ACK 应答，此时主机 1 进入 TIME_WAIT 状态。







主机 1 在 TIME_WAIT 停留持续时间是固定的，是最长分节生命期 MSL（maximum segment lifetime）的两倍，一般称之为 2MSL。
和大多数 BSD 派生的系统一样，Linux 系统里有一个硬编码的字段，名称为TCP_TIMEWAIT_LEN，其值为 60 秒。
也就是说，Linux 系统停留在 TIME_WAIT 的时间为固定的 60 秒。



过了 2MSL 这个时间之后，主机 1 由 TIME_WAIT 状态 就进入 CLOSED 状态
只有发起连接终止的一方会进入 TIME_WAIT 状态


2MSL 的时间是从主机1 接收到 FIN 后发送 ACK 开始计时的；
如果在 TIME_WAIT 时间内，因为主机 1 的 ACK 没有传输到主机 2，
主机 1 又接收到了主机 2 重发的 FIN 报文，那么 2MSL 时间将重新计时。
```

