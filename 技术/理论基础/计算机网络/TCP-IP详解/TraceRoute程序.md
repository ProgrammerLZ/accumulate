# Traceroute程序

TraceRoute可以：

1. 让我们看到IP数据报从一台主机传到另一台主机所经过的路由。
2. 让我们使用IP源路由选项



## Traceroute程序的操作

它使用ICMP报文和IP首部中的TTL字段，通常把TTL看做一个跳站计数器。

通常情况系统不应该接收TTL字段为0的数据报。

TTL是0或1的时候，路由器将该数据报丢弃，并给源机发一份**ICMP超时信息**。

**Traceroute程序关键在于包含这份ICMP信息的IP报文的信源地址是该路由器的IP地址。**

1. 发送一份TTL=1的IP数据报给目的主机，第一个路由器将TTL-1，丢弃该数据报，并发送回一份超时ICMP报文，报文中包含路由器的IP地址，这样就得到了第一个路由器的地址。
2. 发送一份TTL=2的IP数据报给目的主机，这样就得到了第2个路由器的地址。
3. 循环上述。

> 程序的终止
>
> Traceroute程序实际上发送的是一个UDP数据报给目的主机，并选择一个不可能的值作为UDP端口号，当该数据报到底目的主机时，目的主机产生一份“端口不可达”错误的ICMP报文，Traceroute就能知道该数据报已经到达目的主机了，因此将程序终止。



## 局域网输出

局域网主机分布如图。

<img src="assets/TraceRoute%E7%A8%8B%E5%BA%8F/image-20230214213154971.png" alt="image-20230214213154971" style="zoom:50%;" />



在srv4上对slip进行traceroute。

![image-20230214210836176](assets/TraceRoute%E7%A8%8B%E5%BA%8F/image-20230214210836176.png)

30代表最大TTL值。

40字节IP数据报：

* 20字节IP首部
* 8字节UDP首部
* 12字节用户数据

每个TTL值发送3份数据报。接收到一份ICMP报文，就计算并打印往返时间。

4s内未收到3份中任意一份，打印一个星号，并发送下一份数据报。





tcpdump的结果：

![image-20230214212901074](assets/TraceRoute%E7%A8%8B%E5%BA%8F/image-20230214212901074.png)





ICMP“超时”报文有两种，它们的code不同，如图：

![image-20230214212629592](assets/TraceRoute%E7%A8%8B%E5%BA%8F/image-20230214212629592.png)

当到路由器收到TTL=1的数据报时，code=0；主机组装分片超时时，code=1.





计算SLIP链路的往返时间。

* 链路速率960/s

* 发送出的UDP数据报共42字节：

  * 20字节IP首部
  * 8字节UDP首部
  * 12字节用户数据
  * 至少2字节的SLIP帧

* 返回的ICMP报文共58字节：

  <img src="assets/TraceRoute%E7%A8%8B%E5%BA%8F/image-20230214214437889.png" alt="image-20230214214437889" style="zoom:80%;" />

  * 20节发生差错的数据报的IP首部
  * 8字节的UDP首部
  * 20字节IP首部
  * 8字节ICMP首部
  * 2字节SLIP帧

预计的RTT为：$(42 + 58) / 960 = 104ms$





关于Traceroute，注意：

1. 并不保证路由永远不变，两份连续的IP数据报也可能采用不同的路由。
2. 不能保证ICMP报文的路由与traceroute发送的UDP数据报采用同一个路由。
3. 返回的ICMP报文中的信源IP地址是UDP数据报到达的路由器的**入口**的地址。





## 广域网输出

![image-20230214223447228](assets/TraceRoute%E7%A8%8B%E5%BA%8F/image-20230214223447228.png)