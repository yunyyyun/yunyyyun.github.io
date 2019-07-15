本文使用mac电脑Wireshark抓包，并对一次http链接做简单的分析

## Wireshark抓包

打开Wireshark选择本机mac的网卡，输入过滤条件：tcp.port == 80 && (ip.dst==111.231.221.195 || ip.src==111.231.221.195)开始抓包，接着mac浏览器输入http://111.231.221.195/getMusicXmlData.php?file=sadness发起http，就能看到这次请求的所有tcp包：

![](/img/06-22-01.png)

如图所示，一个http请求包括如下过程：

![](/img/06-22-02.png)

1. 建立tcp连接，这个过程包括三个包（三次握手包），分别是
   客户端发起建立连接的请求包[SYN]（序号43）、
   服务端应答客户端的请求，并要求二次确认[SYN, ACK]、
   客户端的确认[ACK]，至此一个tcp连接建立完成

2. 连接建立后客户端即发起http请求包，如图序号46的包，展开后可以看到如下内容：

   ![](/img/06-22-03.png)

   如图可以看到http头域各字段内容

3. 服务端受到http请求后应答返回数据，此过程为tcp传输过程，服务端不断的发送应答数据包[TCP segment of a reassembled PDU]，客户端发送确认包[ACK]

4. 应答发送完毕，断开tcp连接，这个过程包括四个包（4次挥手），分别是[FIN, ACK]、[ACK]、[FIN,ACK]、[ACK]。

## TCP的可靠性

tcp即是通过重传机制保证数据传输的可靠性，依次观察上面第3步的各个tcp包，会发现如下几个特殊的包：

* [TCP Window Update] : 即滑窗大小调整的包，用来通知对方（发送方）更新滑窗大小

- [TCP Previous segment not captured]：在TCP传输过程中，同一台主机发出的数据段应该是连续的，即后一个包的Seq号等于前一个包的Seq+Len（展开[TCP segment of a reassembled PDU]包可以看到Seq和Len，三次握手和四次挥手是例外），例如亲一个包：Seq: 60110, Ack: 222, Len: 1400，则下一个包的Seq=60110+1400=61510，否则就是丢包了。
- [TCP Out_of_Order]：同样当Wireshark发现后一个包的Seq号小于前一个包的Seq Len时，就会认为是乱序了，因此提示 [TCP Out-of-Order] 。
- [TCP Dup ACK 148#1]：表示重复应答#前的表示报文到哪个序号丢失，#后面的是表示第几次丢失。
- [TCP Retransmission]：即丢包了后的重传

## Mac电脑抓iPhone包

使用Wireshark抓包是基于网卡的，而要抓iPhone的包，只需要讲iPhone连上mac，然后终端输入

```
rvictl -s iphoneUdid
```

即可创建对于iPhone的虚拟网卡，名字一般是rvi0，打开Wireshark则可以抓rvi0的数据包了



