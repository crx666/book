# TCP/IP网络模型

TCP/IP 网络通常是由上到下分成 4 层没，分别是应用层，传输层，网络层和网络接口层。

![img](https://cdn.jsdelivr.net/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/tcpip%E5%8F%82%E8%80%83%E6%A8%A1%E5%9E%8B.drawio.png)

再给大家贴一下每一层的封装格式：

![img](https://cdn.jsdelivr.net/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%B5%AE%E7%82%B9/%E5%B0%81%E8%A3%85.png)

网络接口层的传输单位是帧（frame），IP 层的传输单位是包（packet），TCP 层的传输单位是段（segment），HTTP 的传输单位则是消息或报文（message）。但这些名词并没有什么本质的区分，可以统称为数据包。

### 传输层

应用需要传输的数据可能会非常大，如果直接传输就不好控制，因此当传输层的数据包大小超过 **MSS（TCP 最大报文段长度）** ，就要将数据包分块，这样即使中途有一个分块丢失或损坏了，只需要重新发送这一个分块，而不用重新发送整个数据包。在 TCP 协议中，我们把每个分块称为一个 **TCP 段**（*TCP Segment*）。



### 网络层

网络层最常使用的是 IP 协议（*Internet Protocol*），IP 协议会将传输层的报文作为数据部分，再加上 IP 包头组装成 IP 报文，如果 IP 报文大小超过 MTU（以太网中一般为 1500 字节）就会**再次进行分片**，得到一个即将发送到网络的 **IP 报文**。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/%E9%94%AE%E5%85%A5%E7%BD%91%E5%9D%80%E8%BF%87%E7%A8%8B/12.jpg)

### 网络接口层

网络接口层主要为网络层提供「链路级别」传输的服务，负责在以太网、WiFi 这样的底层网络上发送原始数据包，工作在网卡这个层次，使用 **MAC 地址**来标识网络上的设备。



![OSI 参考模型与 TCP/IP 的关系](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzcuanBn?x-oss-process=image/format,png)



# TCP

![TCP 头格式](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzYuanBn?x-oss-process=image/format,png)



**序列号**：在建立连接时由计算机生成的随机数作为其初始值，通过 SYN 包传给接收端主机，每发送一次数据，就「累加」一次该「数据字节数」的大小。**用来解决网络包乱序问题。**

**确认应答号**：指下一次「期望」收到的数据的序列号，发送端收到这个确认应答以后可以认为在这个序号以前的数据都已经被正常接收。**用来解决丢包的问题。**

**控制位：**

- *ACK*：该位为 `1` 时，「确认应答」的字段变为有效，TCP 规定除了最初建立连接时的 `SYN` 包之外该位必须设置为 `1` 。

- *RST*：该位为 `1` 时，表示 TCP 连接中出现异常必须强制断开连接。

- *SYN*：该位为 `1` 时，表示希望建立连接，并在其「序列号」的字段进行序列号初始值的设定。

- *FIN*：该位为 `1` 时，表示今后不会再有数据发送，希望断开连接。当通信结束希望断开连接时，通信双方的主机之间就可以相互交换 `FIN` 位为 1 的 TCP 段。

  ​


 TCP 是一个工作在**传输层**的**可靠**数据传输的服务，它能确保接收端接收的网络包是**无损坏、无间隔、非冗余和按序的。**

TCP 是**面向连接的、可靠的、基于字节流**的传输层通信协议。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzguanBn?x-oss-process=image/format,png)

- **面向连接**：一定是「一对一」才能连接，不能像 UDP 协议可以一个主机同时向多个主机发送消息，也就是一对多是无法做到的；

- **可靠的**：无论的网络链路中出现了怎样的链路变化，TCP 都可以保证一个报文一定能够到达接收端；

- **字节流**：用户消息通过 TCP 协议传输时，消息可能会被操作系统「分组」成多个的 TCP 报文，如果接收方的程序如果不知道「消息的边界」，是无法读出一个有效的用户消息的。并且 TCP 报文是「有序的」，当「前一个」TCP 报文没有收到的时候，即使它先收到了后面的 TCP 报文，那么也不能扔给应用层去处理，同时对「重复」的 TCP 报文会自动丢弃。

  ​

Tcp四元组可以确定唯一一个连接，四元组包括如下：

- 源地址
- 源端口
- 目的地址
- 目的端口

![TCP 四元组](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzEwLmpwZw?x-oss-process=image/format,png)



服务器监听了一个端口，那么它的tcp最大连接数就相当于**客户端的ip数量*客户端的端口数量**

客户端的 IP 数最多为 `2` 的 `32` 次方，客户端的端口数最多为 `2` 的 `16` 次方，也就是服务端单机最大 TCP 连接数，约为 `2` 的 `48` 次方。

当然，服务端最大并发 TCP 连接数远不能达到理论上限，会受以下因素影响：

- **文件描述符限制**

  ，每个 TCP 连接都是一个文件，如果文件描述符被占满了，会发生 too many open files。Linux 对可打开的文件描述符的数量分别作了三个方面的限制：

  - **系统级**：当前系统可打开的最大数量，通过 cat /proc/sys/fs/file-max 查看；
  - **用户级**：指定用户可打开的最大数量，通过 cat /etc/security/limits.conf 查看；
  - **进程级**：单个进程可打开的最大数量，通过 cat /proc/sys/fs/nr_open 查看；

- **内存限制**，每个 TCP 连接都要占用一定内存，操作系统的内存是有限的，如果内存资源被占满后，会发生 OOM。




# UDP

UDP 不提供复杂的控制机制，利用 IP 提供面向「无连接」的通信服务。

UDP 协议真的非常简，头部只有 `8` 个字节（ 64 位），UDP 的头部格式如下：

![UDP 头部格式](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzEyLmpwZw?x-oss-process=image/format,png)

- 目标和源端口：主要是告诉 UDP 协议应该把报文发给哪个进程。
- 包长度：该字段保存了 UDP 首部的长度跟数据的长度之和。
- 校验和：校验和是为了提供可靠的 UDP 首部和数据而设计，防止收到在网络传输中受损的 UDP包。



**TCP 和 UDP 区别：**

*1. 连接*

- TCP 是面向连接的传输层协议，传输数据前先要建立连接。
- UDP 是不需要连接，即刻传输数据。

*2. 服务对象*

- TCP 是一对一的两点服务，即一条连接只有两个端点。
- UDP 支持一对一、一对多、多对多的交互通信

*3. 可靠性*

- TCP 是可靠交付数据的，数据可以无差错、不丢失、不重复、按需到达。
- UDP 是尽最大努力交付，不保证可靠交付数据。

*4. 拥塞控制、流量控制*

- TCP 有拥塞控制和流量控制机制，保证数据传输的安全性。
- UDP 则没有，即使网络非常拥堵了，也不会影响 UDP 的发送速率。

*5. 首部开销*

- TCP 首部长度较长，会有一定的开销，首部在没有使用「选项」字段时是 `20` 个字节，如果使用了「选项」字段则会变长的。
- UDP 首部只有 8 个字节，并且是固定不变的，开销较小。

*6. 传输方式*

- TCP 是流式传输，没有边界，但保证顺序和可靠。
- UDP 是一个包一个包的发送，是有边界的，但可能会丢包和乱序。

*7. 分片不同*

- TCP 的数据大小如果大于 MSS 大小，则会在传输层进行分片，目标主机收到后，也同样在传输层组装 TCP 数据包，如果中途丢失了一个分片，只需要传输丢失的这个分片。
- UDP 的数据大小如果大于 MTU 大小，则会在 IP 层进行分片，目标主机收到后，在 IP 层组装完数据，接着再传给传输层。




# TCP 连接建立

> TCP 三次握手过程和状态变迁

TCP 是面向连接的协议，所以使用 TCP 前必须先建立连接，而**建立连接是通过三次握手来进行的**。三次握手的过程如下图：

![TCP 三次握手](https://cdn.jsdelivr.net/gh/xiaolincoder/ImageHost4/%E7%BD%91%E7%BB%9C/tcp%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B.drawio.png)

- 一开始，客户端和服务端都处于 `CLOSED` 状态。先是服务端主动监听某个端口，处于 `LISTEN` 状态

![第一个报文—— SYN 报文](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzE1LmpwZw?x-oss-process=image/format,png)

- 客户端会随机初始化序号（`client_isn`），将此序号置于 TCP 首部的「序号」字段中，同时把 `SYN` 标志位置为 `1` ，表示 `SYN` 报文。接着把第一个 SYN 报文发送给服务端，表示向服务端发起连接，该报文不包含应用层数据，之后客户端处于 `SYN-SENT` 状态。

![第二个报文 —— SYN + ACK 报文](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzE2LmpwZw?x-oss-process=image/format,png)

- 服务端收到客户端的 `SYN` 报文后，首先服务端也随机初始化自己的序号（`server_isn`），将此序号填入 TCP 首部的「序号」字段中，其次把 TCP 首部的「确认应答号」字段填入 `client_isn + 1`, 接着把 `SYN` 和 `ACK` 标志位置为 `1`。最后把该报文发给客户端，该报文也不包含应用层数据，之后服务端处于 `SYN-RCVD` 状态。

![第三个报文 —— ACK 报文](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzE3LmpwZw?x-oss-process=image/format,png)

- 客户端收到服务端报文后，还要向服务端回应最后一个应答报文，首先该应答报文 TCP 首部 `ACK` 标志位置为 `1` ，其次「确认应答号」字段填入 `server_isn + 1` ，最后把报文发送给服务端，这次报文可以携带客户到服务器的数据，之后客户端处于 `ESTABLISHED` 状态。
- 服务器收到客户端的应答报文后，也进入 `ESTABLISHED` 状态。

从上面的过程可以发现**第三次握手是可以携带数据的，前两次握手是不可以携带数据的**，这也是面试常问的题。

一旦完成三次握手，双方都处于 `ESTABLISHED` 状态，此时连接就已建立完成，客户端和服务端就可以相互发送数据了。



TCP 的连接状态查看，在 Linux 可以通过 `netstat -napt` 命令查看。

![TCP 连接状态查看](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzE4LmpwZw?x-oss-process=image/format,png)



TCP 建立连接时，通过三次握手**能防止历史连接的建立，能减少双方不必要的资源开销，能帮助双方同步初始化序列号**。序列号能够保证数据包不重复、不丢弃和按序传输。



不使用「两次握手」和「四次握手」的原因：

- 「两次握手」：无法防止历史连接的建立，会造成双方资源的浪费，也无法可靠的同步双方序列号；
- 「四次握手」：三次握手就已经理论上最少可靠连接建立，所以不需要使用更多的通信次数。





第一次握手丢失，触发超时重发机制，客户端会重新发送SYN报文。每次超时时间是上次等待时间的2倍。

超时最大次数由操作系统内核控制。默认是5.

第二次握手丢失，服务器接受到客户端的SYN报文以后，返回对应的SYN+ACK报文，进入SYN_RCVD状态。

等待客户端的第三次握手，但是由于第二次包丢失，服务器会触发超时重传机制，重新发送包给客户端。超时最大次数由操作系统内核控制。默认是5。同时，由于客户端不知道服务器接收到第一次包没有，所以也会触发超时重传。

第三次握手丢失，客户端接收到服务器的SYN-ACK报文，发送第三次ACK报文给服务器，由于包丢失，客户端是不会重新发送ACK包给服务器的。所以等服务器触发超时重传，重发第二次包给客户端。客户端再发送ACK包给服务器。



### SYN攻击

我们都知道 TCP 连接建立是需要三次握手，假设攻击者短时间伪造不同 IP 地址的 `SYN` 报文，服务端每接收到一个 `SYN` 报文，就进入`SYN_RCVD` 状态，但服务端发送出去的 `ACK + SYN` 报文，无法得到未知 IP 主机的 `ACK` 应答，久而久之就会**占满服务端的半连接队列**，使得服务器不能为正常用户服务。



我们先来看下 Linux 内核的 `SYN` 队列（半连接队列）与 `Accpet` 队列（全连接队列）是如何工作的？

![正常流程](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzI2LmpwZw?x-oss-process=image/format,png)

正常流程：

- 当服务端接收到客户端的 SYN 报文时，会将其加入到内核的「 SYN 队列」；

- 接着发送 SYN + ACK 给客户端，等待客户端回应 ACK 报文；

- 服务端接收到 ACK 报文后，从「 SYN 队列」移除放入到「 Accept 队列」；

- 应用通过调用 `accpet()` socket 接口，从「 Accept 队列」取出连接。

  ​

如果不断受到 SYN 攻击，就会**导致 SYN 队列（半连接队列）被占满，从而导致无法在建立新的连接**。



> 如何防御 SYN 攻击？

这里给出几种防御 SYN 攻击的方法：

- 增大半连接队列；
- 开启 tcp_syncookies 功能
- 减少 SYN+ACK 重传次数



*方式一：增大半连接队列*

在前面源码和实验中，得知**要想增大半连接队列，我们得知不能只单纯增大 tcp_max_syn_backlog 的值，还需一同增大 somaxconn 和 backlog，也就是增大全连接队列**。否则，只单纯增大 tcp_max_syn_backlog 是无效的。

**当超过了 TCP 最大全连接队列，服务端则会丢掉后续进来的 TCP 连接**

**TCP 全连接队列的最大值取决于 somaxconn 和 backlog 之间的最小值，也就是 min(somaxconn, backlog)**

增大 tcp_max_syn_backlog 和 somaxconn 的方法是修改 Linux 内核参数：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8D%8A%E8%BF%9E%E6%8E%A5%E5%92%8C%E5%85%A8%E8%BF%9E%E6%8E%A5/41.jpg)

增大 backlog 的方式，每个 Web 服务都不同，比如 Nginx 增大 backlog 的方法如下：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8D%8A%E8%BF%9E%E6%8E%A5%E5%92%8C%E5%85%A8%E8%BF%9E%E6%8E%A5/42.jpg)

最后，改变了如上这些参数后，要重启 Nginx 服务，因为半连接队列和全连接队列都是在 listen() 初始化的。



*方式二：开启 tcp_syncookies 功能*

```
net.ipv4.tcp_syncookies = 1
```

![tcp_syncookies 应对 SYN 攻击](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzI5LmpwZw?x-oss-process=image/format,png)

- 当 「 SYN 队列」满之后，后续服务器收到 SYN 包，不进入「 SYN 队列」；
- 计算出一个 `cookie` 值，再以 SYN + ACK 中的「序列号」返回客户端，
- 服务端接收到客户端的应答报文时，服务器会检查这个 ACK 包的合法性。如果合法，直接放入到「 Accept 队列」。
- 最后应用通过调用 `accpet()` socket 接口，从「 Accept 队列」取出的连接




*方式三：减少 SYN+ACK 重传次数*

当服务端受到 SYN 攻击时，就会有大量处于 SYN_REVC 状态的 TCP 连接，处于这个状态的 TCP 会重传 SYN+ACK ，当重传超过次数达到上限后，就会断开连接。

那么针对 SYN 攻击的场景，我们可以减少 SYN+ACK 的重传次数，以加快处于 SYN_REVC 状态的 TCP 连接断开。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8D%8A%E8%BF%9E%E6%8E%A5%E5%92%8C%E5%85%A8%E8%BF%9E%E6%8E%A5/44.jpg)




# TCP 连接断开



双方都可以主动断开连接，断开连接后主机中的「资源」将被释放，四次挥手的过程如下图：

![客户端主动关闭连接 —— TCP 四次挥手](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzMwLmpwZw?x-oss-process=image/format,png)

- 客户端打算关闭连接，此时会发送一个 TCP 首部 `FIN` 标志位被置为 `1` 的报文，也即 `FIN` 报文，之后客户端进入 `FIN_WAIT_1` 状态。
- 服务端收到该报文后，就向客户端发送 `ACK` 应答报文，接着服务端进入 `CLOSED_WAIT` 状态。
- 客户端收到服务端的 `ACK` 应答报文后，之后进入 `FIN_WAIT_2` 状态。
- 等待服务端处理完数据后，也向客户端发送 `FIN` 报文，之后服务端进入 `LAST_ACK` 状态。
- 客户端收到服务端的 `FIN` 报文后，回一个 `ACK` 应答报文，之后进入 `TIME_WAIT` 状态
- 服务器收到了 `ACK` 应答报文后，就进入了 `CLOSED` 状态，至此服务端已经完成连接的关闭。
- 客户端在经过 `2MSL` 一段时间后，自动进入 `CLOSED` 状态，至此客户端也完成连接的关闭。

你可以看到，每个方向都需要**一个 FIN 和一个 ACK**，因此通常被称为**四次挥手**。

这里一点需要注意是：**主动关闭连接的，才有 TIME_WAIT 状态。**



> 为什么 TIME_WAIT 等待的时间是 2MSL？

`MSL` 是 Maximum Segment Lifetime，**报文最大生存时间**，它是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃。因为 TCP 报文基于是 IP 协议的，而 IP 头中有一个 `TTL` 字段，是 IP 数据报可以经过的最大路由数，每经过一个处理他的路由器此值就减 1，当此值为 0 则数据报将被丢弃，同时发送 ICMP 报文通知源主机。

MSL 与 TTL 的区别： MSL 的单位是时间，而 TTL 是经过路由跳数。所以 **MSL 应该要大于等于 TTL 消耗为 0 的时间**，以确保报文已被自然消亡。

**TTL 的值一般是 64，Linux 将 MSL 设置为 30 秒，意味着 Linux 认为数据报文经过 64 个路由器的时间不会超过 30 秒，如果超过了，就认为报文已经消失在网络中了**。

TIME_WAIT 等待 2 倍的 MSL，比较合理的解释是： 网络中可能存在来自发送方的数据包，当这些发送方的数据包被接收方处理后又会向对方发送响应，所以**一来一回需要等待 2 倍的时间**。

比如，如果被动关闭方没有收到断开连接的最后的 ACK 报文，就会触发超时重发 `FIN` 报文，另一方接收到 FIN 后，会重发 ACK 给被动关闭方， 一来一去正好 2 个 MSL。

`2MSL` 的时间是从**客户端接收到 FIN 后发送 ACK 开始计时的**。如果在 TIME-WAIT 时间内，因为客户端的 ACK 没有传输到服务端，客户端又接收到了服务端重发的 FIN 报文，那么 **2MSL 时间将重新计时**。

在 Linux 系统里 `2MSL` 默认是 `60` 秒，那么一个 `MSL` 也就是 `30` 秒。**Linux 系统停留在 TIME_WAIT 的时间为固定的 60 秒**。



> TIME_WAIT 过多有什么危害？

过多的 TIME-WAIT 状态主要的危害有两种：

- 第一是内存资源占用；
- 第二是对端口资源的占用，一个 TCP 连接至少消耗「发起连接方」的一个本地端口；

**如果「发起连接方」的 TIME_WAIT 状态过多，占满了所有端口资源，则会导致无法创建新连接。**

客户端（发起连接方）受端口资源限制：

- 客户端TIME_WAIT过多，就会导致端口资源被占用，因为端口就 65536 个，被占满就会导致无法创建新的连接。

服务端（被动连接方）受系统资源限制：

- 由于一个四元组表示 TCP 连接，理论上服务端可以建立很多连接，因为服务端只监听一个端口，不会因为 TCP 连接过多而导致端口资源受限。但是 TCP 连接过多，会占用系统资源，比如文件描述符、内存资源、CPU 资源、线程资源等。

**如果服务端要避免过多的 TIME_WAIT 状态的连接，就永远不要主动断开连接，让客户端去断开，由分布在各处的客户端去承受 TIME_WAIT**。



# 重传机制

TCP 针对数据包丢失的情况，会用**重传机制**解决。

接下来说说常见的重传机制：

- 超时重传
- 快速重传
- SACK
- D-SACK



### 超时重传

重传机制的其中一个方式，就是在发送数据时，设定一个定时器，当超过指定的时间后，没有收到对方的 `ACK` 确认应答报文，就会重发该数据，也就是我们常说的**超时重传**。

TCP 会在以下两种情况发生超时重传：

- 数据包丢失
- 确认应答丢失

`RTT` 指的是**数据发送时刻到接收到确认的时刻的差值**，也就是包的往返时间。

超时重传时间是以 `RTO` （Retransmission Timeout 超时重传时间）表示。

**超时重传时间 RTO 的值应该略大于报文往返 RTT 的值**。



### 快速重传

TCP 还有另外一种**快速重传（Fast Retransmit）机制**，它**不以时间为驱动，而是以数据驱动重传**。

![快速重传机制](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/10.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

在上图，发送方发出了 1，2，3，4，5 份数据：

- 第一份 Seq1 先送到了，于是就 Ack 回 2；
- 结果 Seq2 因为某些原因没收到，Seq3 到达了，于是还是 Ack 回 2；
- 后面的 Seq4 和 Seq5 都到了，但还是 Ack 回 2，因为 Seq2 还是没有收到；
- **发送端收到了三个 Ack = 2 的确认，知道了 Seq2 还没有收到，就会在定时器过期之前，重传丢失的 Seq2。**
- 最后，收到了 Seq2，此时因为 Seq3，Seq4，Seq5 都收到了，于是 Ack 回 6 。

所以，快速重传的工作方式是当收到三个相同的 ACK 报文时，会在定时器过期之前，重传丢失的报文段。

快速重传机制只解决了一个问题，就是超时时间的问题，但是它依然面临着另外一个问题。就是**重传的时候，是重传之前的一个，还是重传所有的问题。**所以需要下边的**SACK **方法



### SACK 方法

还有一种实现重传机制的方式叫：`SACK`（ Selective Acknowledgment 选择性确认）。

这种方式需要在 TCP 头部「选项」字段里加一个 `SACK` 的东西，它**可以将缓存的地图发送给发送方**，这样发送方就可以知道哪些数据收到了，哪些数据没收到，知道了这些信息，就可以**只重传丢失的数据**。

如下图，发送方收到了三次同样的 ACK 确认报文，于是就会触发快速重发机制，通过 `SACK` 信息发现只有 `200~299` 这段数据丢失，则重发时，就只选择了这个 TCP 段进行重复。

![选择性确认](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/11.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

如果要支持 `SACK`，必须双方都要支持。在 Linux 下，可以通过 `net.ipv4.tcp_sack` 参数打开这个功能（Linux 2.4 后默认打开）



### Duplicate SACK

Duplicate SACK 又称 `D-SACK`，其主要**使用了 SACK 来告诉「发送方」有哪些数据被重复接收了。**

*栗子一号：ACK 丢包*

![ACK 丢包](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/12.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

- 「接收方」发给「发送方」的两个 ACK 确认应答都丢失了，所以发送方超时后，重传第一个数据包（3000 ~ 3499）
- **于是「接收方」发现数据是重复收到的，于是回了一个 SACK = 3000~3500**，告诉「发送方」 3000~3500 的数据早已被接收了，因为 ACK 都到了 4000 了，已经意味着 4000 之前的所有数据都已收到，所以这个 SACK 就代表着 `D-SACK`。
- 这样「发送方」就知道了，数据没有丢，是「接收方」的 ACK 确认报文丢了。

*栗子二号：网络延时*

![网络延时](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/13.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

- 数据包（1000~1499） 被网络延迟了，导致「发送方」没有收到 Ack 1500 的确认报文。
- 而后面报文到达的三个相同的 ACK 确认报文，就触发了快速重传机制，但是在重传后，被延迟的数据包（1000~1499）又到了「接收方」；
- **所以「接收方」回了一个 SACK=1000~1500，因为 ACK 已经到了 3000，所以这个 SACK 是 D-SACK，表示收到了重复的包。**
- 这样发送方就知道快速重传触发的原因不是发出去的包丢了，也不是因为回应的 ACK 包丢了，而是因为网络延迟了。

可见，`D-SACK` 有这么几个好处：

1. 可以让「发送方」知道，是发出去的包丢了，还是接收方回应的 ACK 包丢了;
2. 可以知道是不是「发送方」的数据包被网络延迟了;
3. 可以知道网络中是不是把「发送方」的数据包给复制了;

在 Linux 下可以通过 `net.ipv4.tcp_dsack` 参数开启/关闭这个功能（Linux 2.4 后默认打开）



# 滑动窗口

窗口大小就是指**无需等待确认应答，而可以继续发送数据的最大值**。

窗口的实现实际上是操作系统开辟的一个缓存空间，发送方主机在等到确认应答返回之前，必须在缓冲区中保留已发送的数据。如果按期收到确认应答，此时数据就可以从缓存区清除。

![用滑动窗口方式并行处理](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/15.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

图中的 ACK 600 确认应答报文丢失，也没关系，因为可以通过下一个确认应答进行确认，只要发送方收到了 ACK 700 确认应答，就意味着 700 之前的所有数据「接收方」都收到了。这个模式就叫**累计确认**或者**累计应答**。



> 窗口大小由哪一方决定？

TCP 头里有一个字段叫 `Window`，也就是窗口大小。

**这个字段是接收端告诉发送端自己还有多少缓冲区可以接收数据。于是发送端就可以根据这个接收端的处理能力来发送数据，而不会导致接收端处理不过来。**

所以，通常窗口的大小是由接收方的窗口大小来决定的。

发送方发送的数据大小不能超过接收方的窗口大小，否则接收方就无法正常接收到数据。

> 发送方的滑动窗口

我们先来看看发送方的窗口，下图就是发送方缓存的数据，根据处理的情况分成四个部分，其中深蓝色方框是发送窗口，紫色方框是可用窗口：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/16.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

- \#1 是已发送并收到 ACK确认的数据：1~31 字节
- \#2 是已发送但未收到 ACK确认的数据：32~45 字节
- \#3 是未发送但总大小在接收方处理范围内（接收方还有空间）：46~51字节
- \#4 是未发送但总大小超过接收方处理范围（接收方没有空间）：52字节以后



> 程序是如何表示发送方的四个部分的呢？

TCP 滑动窗口方案使用三个指针来跟踪在四个传输类别中的每一个类别中的字节。其中两个指针是绝对指针（指特定的序列号），一个是相对指针（需要做偏移）。

![SND.WND、SND.UN、SND.NXT](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/19.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

- `SND.WND`：表示发送窗口的大小（大小是由接收方指定的）；
- `SND.UNA`：是一个绝对指针，它指向的是已发送但未收到确认的第一个字节的序列号，也就是 #2 的第一个字节。
- `SND.NXT`：也是一个绝对指针，它指向未发送但可发送范围的第一个字节的序列号，也就是 #3 的第一个字节。
- 指向 #4 的第一个字节是个相对指针，它需要 `SND.UNA` 指针加上 `SND.WND` 大小的偏移量，就可以指向 #4 的第一个字节了。

那么可用窗口大小的计算就可以是：

**可用窗口大 = SND.WND -（SND.NXT - SND.UNA）**



> 接收方的滑动窗口

接下来我们看看接收方的窗口，接收窗口相对简单一些，根据处理的情况划分成三个部分：

- \#1 + #2 是已成功接收并确认的数据（等待应用进程读取）；
- \#3 是未收到数据但可以接收的数据；
- \#4 未收到数据并不可以接收的数据；

![接收窗口](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/20.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

其中三个接收部分，使用两个指针进行划分:

- `RCV.WND`：表示接收窗口的大小，它会通告给发送方。

- `RCV.NXT`：是一个指针，它指向期望从发送方发送来的下一个数据字节的序列号，也就是 #3 的第一个字节。

- 指向 #4 的第一个字节是个相对指针，它需要 `RCV.NXT` 指针加上 `RCV.WND` 大小的偏移量，就可以指向 #4 的第一个字节了。

  ​

# 流量控制

**TCP 提供一种机制可以让「发送方」根据「接收方」的实际接收能力控制发送的数据量，这就是所谓的流量控制。**

> 那操心系统的缓冲区，是如何影响发送窗口和接收窗口的呢？

- 客户端作为发送方，服务端作为接收方，发送窗口和接收窗口初始大小为 `360`；
- 服务端非常的繁忙，当收到客户端的数据时，应用层不能及时读取数据。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/22.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)



根据上图的流量控制，说明下每个过程：

1. 客户端发送 140 字节数据后，可用窗口变为 220 （360 - 140）。
2. 服务端收到 140 字节数据，**但是服务端非常繁忙，应用进程只读取了 40 个字节，还有 100 字节占用着缓冲区，于是接收窗口收缩到了 260 （360 - 100）**，最后发送确认信息时，将窗口大小通告给客户端。
3. 客户端收到确认和窗口通告报文后，发送窗口减少为 260。
4. 客户端发送 180 字节数据，此时可用窗口减少到 80。
5. 服务端收到 180 字节数据，**但是应用程序没有读取任何数据，这 180 字节直接就留在了缓冲区，于是接收窗口收缩到了 80 （260 - 180）**，并在发送确认信息时，通过窗口大小给客户端。
6. 客户端收到确认和窗口通告报文后，发送窗口减少为 80。
7. 客户端发送 80 字节数据后，可用窗口耗尽。
8. 服务端收到 80 字节数据，**但是应用程序依然没有读取任何数据，这 80 字节留在了缓冲区，于是接收窗口收缩到了 0**，并在发送确认信息时，通过窗口大小给客户端。
9. 客户端收到确认和窗口通告报文后，发送窗口减少为 0。



当服务端系统资源非常紧张的时候，操心系统可能会直接减少了接收缓冲区大小，这时应用程序又无法及时读取缓存数据，那么这时候就有严重的事情发生了，会出现数据包丢失的现象。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/23.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

说明下每个过程：

1. 客户端发送 140 字节的数据，于是可用窗口减少到了 220。
2. **服务端因为现在非常的繁忙，操作系统于是就把接收缓存减少了 120 字节，当收到 140 字节数据后，又因为应用程序没有读取任何数据，所以 140 字节留在了缓冲区中，于是接收窗口大小从 360 收缩成了 100**，最后发送确认信息时，通告窗口大小给对方。
3. 此时客户端因为还没有收到服务端的通告窗口报文，所以不知道此时接收窗口收缩成了 100，客户端只会看自己的可用窗口还有 220，所以客户端就发送了 180 字节数据，于是可用窗口减少到 40。
4. 服务端收到了 180 字节数据时，**发现数据大小超过了接收窗口的大小，于是就把数据包丢失了。**
5. 客户端收到第 2 步时，服务端发送的确认报文和通告窗口报文，尝试减少发送窗口到 100，把窗口的右端向左收缩了 80，此时可用窗口的大小就会出现诡异的负值。

所以，如果发生了先减少缓存，再收缩窗口，就会出现丢包的现象。

**为了防止这种情况发生，TCP 规定是不允许同时减少缓存又收缩窗口的，而是采用先收缩窗口，过段时间再减少缓存，这样就可以避免了丢包情况。**



### 窗口关闭

**如果窗口大小为 0 时，就会阻止发送方给接收方传递数据，直到窗口变为非 0 为止，这就是窗口关闭。**

TCP 为每个连接设有一个持续定时器，**只要 TCP 连接一方收到对方的零窗口通知，就启动持续计时器。**

如果持续计时器超时，就会发送**窗口探测 ( Window probe ) 报文（ TCP Keep-Alive 报文）**，而对方在确认这个探测报文时，给出自己现在的接收窗口大小。

![窗口探测](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/25.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

- 如果接收窗口仍然为 0，那么收到这个报文的一方就会重新启动持续计时器；
- 如果接收窗口不是 0，那么死锁的局面就可以被打破了。

窗口探测的次数一般为 3 次，每次大约 30-60 秒（不同的实现可能会不一样）。如果 3 次过后接收窗口还是 0 的话，有的 TCP 实现就会发 `RST` 报文来中断连接



### 糊涂窗口综合症

**如果接收方腾出几个字节并告诉发送方现在有几个字节的窗口，而发送方会义无反顾地发送这几个字节，这就是糊涂窗口综合症**。

要解决糊涂窗口综合症，就解决上面两个问题就可以了

- 让接收方不通告小窗口给发送方

- 让发送方避免发送小数据

  ​

> 怎么让接收方不通告小窗口呢？

接收方通常的策略如下:

当「窗口大小」小于 min( MSS，缓存空间/2 ) ，也就是小于 MSS 与 1/2 缓存大小中的最小值时，就会向发送方通告窗口为 `0`，也就阻止了发送方再发数据过来。

等到接收方处理了一些数据后，窗口大小 >= MSS，或者接收方缓存空间有一半可以使用，就可以把窗口打开让发送方发送数据过来。



为了解决 ACK 传输效率低问题，所以就衍生出了 **TCP 延迟确认**。

TCP 延迟确认的策略：

- 当有响应数据要发送时，ACK 会随着响应数据一起立刻发送给对方
- 当没有响应数据要发送时，ACK 将会延迟一段时间，以等待是否有响应数据可以一起发送
- 如果在延迟等待发送 ACK 期间，对方的第二个数据报文又到达了，这时就会立刻发送 ACK





> 怎么让发送方避免发送小数据呢？

发送方通常的策略:

使用 Nagle 算法，该算法的思路是延时处理，它满足以下两个条件中的一条才可以发送数据：

- 没有已发送未确认报文时，立刻发送数据。
- 存在未确认报文时，直到「没有已发送未确认报文」或「数据长度达到 MSS 大小」时，再发送数据。

只要没满足上面条件中的一条，发送方一直在囤积数据，直到满足上面的发送条件。



# 拥塞控制

**在网络出现拥堵时，如果继续发送大量数据包，可能会导致数据包时延、丢失等，这时 TCP 就会重传数据，但是一重传就会导致网络的负担更重，于是会导致更大的延迟以及更多的丢包，这个情况就会进入恶性循环被不断地放大....**

**拥塞控制**，控制的目的就是**避免「发送方」的数据填满整个网络。**



> 什么是拥塞窗口？和发送窗口有什么关系呢？

**拥塞窗口 cwnd**是发送方维护的一个的状态变量，它会根据**网络的拥塞程度动态变化的**。

我们在前面提到过发送窗口 `swnd` 和接收窗口 `rwnd` 是约等于的关系，那么由于加入了拥塞窗口的概念后，此时**发送窗口的值是swnd = min(cwnd, rwnd)，也就是拥塞窗口和接收窗口中的最小值。**

拥塞窗口 `cwnd` 变化的规则：

- 只要网络中没有出现拥塞，`cwnd` 就会增大；

- 但网络中出现了拥塞，`cwnd` 就减少；

  ​

> 那么怎么知道当前网络是否出现了拥塞呢？

其实只要「发送方」没有在规定时间内接收到 ACK 应答报文，也就是**发生了超时重传，就会认为网络出现了用拥塞。**



> 拥塞控制有哪些控制算法？

拥塞控制主要是四个算法：

- 慢启动
- 拥塞避免
- 拥塞发生
- 快速恢复



### 慢启动

TCP 在刚建立连接完成后，首先是有个慢启动的过程，这个慢启动的意思就是一点一点的提高发送数据包的数量。

慢启动的算法记住一个规则就行：**当发送方每收到一个 ACK，拥塞窗口 cwnd 的大小就会加 1。**

这里假定拥塞窗口 `cwnd` 和发送窗口 `swnd` 相等，下面举个栗子：

- 连接建立完成后，一开始初始化 `cwnd = 1`，表示可以传一个 `MSS` 大小的数据。
- 当收到一个 ACK 确认应答后，cwnd 增加 1，于是一次能够发送 2 个
- 当收到 2 个的 ACK 确认应答后， cwnd 增加 2，于是就可以比之前多发2 个，所以这一次能够发送 4 个
- 当这 4 个的 ACK 确认到来的时候，每个确认 cwnd 增加 1， 4 个确认 cwnd 增加 4，于是就可以比之前多发 4 个，所以这一次能够发送 8 个。

![慢启动算法](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/27.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

可以看出慢启动算法，发包的个数是**指数性的增长**。

> 那慢启动涨到什么时候是个头呢？

有一个叫慢启动门限 `ssthresh` （slow start threshold）状态变量。

- 当 `cwnd` < `ssthresh` 时，使用慢启动算法。
- 当 `cwnd` >= `ssthresh` 时，就会使用「拥塞避免算法」



### 拥塞避免算法

前面说道，当拥塞窗口 `cwnd` 「超过」慢启动门限 `ssthresh` 就会进入拥塞避免算法。

一般来说 `ssthresh` 的大小是 `65535` 字节。

那么进入拥塞避免算法后，它的规则是：**每当收到一个 ACK 时，cwnd 增加 1/cwnd。**

接上前面的慢启动的栗子，现假定 `ssthresh` 为 `8`：

- 当 8 个 ACK 应答确认到来时，每个确认增加 1/8，8 个 ACK 确认 cwnd 一共增加 1，于是这一次能够发送 9 个 `MSS` 大小的数据，变成了**线性增长。**

![拥塞避免](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/28.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

所以，我们可以发现，拥塞避免算法就是将原本慢启动算法的指数增长变成了线性增长，还是增长阶段，但是增长速度缓慢了一些。

就这么一直增长着后，网络就会慢慢进入了拥塞的状况了，于是就会出现丢包现象，这时就需要对丢失的数据包进行重传。

当触发了重传机制，也就进入了「拥塞发生算法」



### 拥塞发生

当网络出现拥塞，也就是会发生数据包重传，重传机制主要有两种：

- 超时重传
- 快速重传



> 发生超时重传的拥塞发生算法

当发生了「超时重传」，则就会使用拥塞发生算法。

这个时候，ssthresh 和 cwnd 的值会发生变化：

- `ssthresh` 设为 `cwnd/2`，
- `cwnd` 重置为 `1`

![拥塞发送 —— 超时重传](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/29.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

接着，就重新开始慢启动，慢启动是会突然减少数据流的。这真是一旦「超时重传」，马上回到解放前。但是这种方式太激进了，反应也很强烈，会造成网络卡顿。



> 发生快速重传的拥塞发生算法

还有更好的方式，前面我们讲过「快速重传算法」。当接收方发现丢了一个中间包的时候，发送三次前一个包的 ACK，于是发送端就会快速地重传，不必等待超时再重传。

TCP 认为这种情况不严重，因为大部分没丢，只丢了一小部分，则 `ssthresh` 和 `cwnd` 变化如下：

- `cwnd = cwnd/2` ，也就是设置为原来的一半;
- `ssthresh = cwnd`;
- 进入快速恢复算法



进入快速恢复算法如下：

- 拥塞窗口 `cwnd = ssthresh + 3` （ 3 的意思是确认有 3 个数据包被收到了）；
- 重传丢失的数据包；
- 如果再收到重复的 ACK，那么 cwnd 增加 1；
- 如果收到新数据的 ACK 后，把 cwnd 设置为第一步中的 ssthresh 的值，原因是该 ACK 确认了新的数据，说明从 duplicated ACK 时的数据都已收到，该恢复过程已经结束，可以回到恢复之前的状态了，也即再次进入拥塞避免状态；

![快速重传和快速恢复](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/%E6%8B%A5%E5%A1%9E%E5%8F%91%E7%94%9F-%E5%BF%AB%E9%80%9F%E9%87%8D%E4%BC%A0.drawio.png?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

也就是没有像「超时重传」一夜回到解放前，而是还在比较高的值，后续呈线性增长。



# TCP三次握手优化

1. 客户端优化

   客户端在发送请求包的时候，如果包丢失，会超时重传，控制超时重传次数可以优化tcp错误暴露时间

2. 服务端优化

   - 增加tcp半连接队列长度，设置syncookie
   - 增加tcp全连接队列长度
   - 调整超时重传的次数

3. 绕过三次握手

   开启tcp fast  open功能，可以减少三次握手的交互时间 

![三次握手优化策略](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%82%E6%95%B0/24.jpg)



# TCP四次挥手优化

1. 主动方优化
   - 调整主动方发起FIN包的重传次数tcp_orphan_retries
   - 当主动方收到 ACK 报文后，连接就进入 FIN_WAIT2 状态，根据关闭的方式不同，优化的方式也不同：
     - 如果这是 close 函数关闭的连接，那么它就是孤儿连接。如果 `tcp_fin_timeout` 秒内没有收到对方的 FIN 报文，连接就直接关闭。同时，为了应对孤儿连接占用太多的资源，`tcp_max_orphans` 定义了最大孤儿连接的数量，超过时连接就会直接释放。
     - 反之是 shutdown 函数关闭的连接，则不受此参数限制；
   - 当主动方接收到 FIN 报文，并返回 ACK 后，主动方的连接进入 TIME_WAIT 状态。这一状态会持续 1 分钟，为了防止 TIME_WAIT 状态占用太多的资源，`tcp_max_tw_buckets` 定义了最大数量，超过时连接也会直接释放。
   - 当 TIME_WAIT 状态过多时，还可以通过设置 `tcp_tw_reuse` 和 `tcp_timestamps` 为 1 ，将 TIME_WAIT 状态的端口复用于作为客户端的新连接，注意该参数只适用于客户端。
2. 被动方优化
   - 回复ACK包后进入了CLOST_WAIT状态，等待进程调用close函数关闭连接。如果出现大量CLOSE_WAIT状态应该检查程序是否没有正确调用close函数
   - 调整被动发发送FIN包后的重传次数tcp_orphan_retries



![四次挥手的优化策略](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%82%E6%95%B0/39.jpg)



# TCP传输数据的性能提升

![数据传输的优化策略](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%82%E6%95%B0/49.jpg)

TCP 可靠性是通过 ACK 确认报文实现的，又依赖滑动窗口提升了发送速度也兼顾了接收方的处理能力。

可是，默认的滑动窗口最大值只有 64 KB，不满足当今的高速网络的要求，要想提升发送速度必须提升滑动窗口的上限，在 Linux 下是通过设置 `tcp_window_scaling` 为 1 做到的，此时最大值可高达 1GB。

滑动窗口定义了网络中飞行报文的最大字节数，当它超过带宽时延积时，网络过载，就会发生丢包。而当它小于带宽时延积时，就无法充分利用网络带宽。因此，滑动窗口的设置，必须参考带宽时延积。

内核缓冲区决定了滑动窗口的上限，缓冲区可分为：发送缓冲区 tcp_wmem 和接收缓冲区 tcp_rmem。

Linux 会对缓冲区动态调节，我们应该把缓冲区的上限设置为带宽时延积。发送缓冲区的调节功能是自动打开的，而接收缓冲区需要把 tcp_moderate_rcvbuf 设置为 1 来开启。其中，调节的依据是 TCP 内存范围 tcp_mem。

但需要注意的是，如果程序中的 socket 设置 SO_SNDBUF 和 SO_RCVBUF，则会关闭缓冲区的动态整功能，所以不建议在程序设置它俩，而是交给内核自动调整比较好。

有效配置这些参数后，既能够最大程度地保持并发性，也能让资源充裕时连接传输速度达到最大值。



# 为什么TCP初始化序列号不一样？

主要原因是为了防止历史报文被下一个相同四元组的连接接收。

如果能正常四次挥手，由于 TIME_WAIT 状态会持续 2 MSL 时长，历史报文会在下一个连接之前就会自然消失。

但是，我们并不能保证每次连接都能通过四次挥手来正常关闭连接。

假设每次建立连接，客户端和服务端的初始化序列号都是从 0 开始：

![图片](https://img-blog.csdnimg.cn/img_convert/158a17b4460afc0b13eff129dfa931a6.png)

过程如下：

- 客户端和服务端建立一个 TCP 连接，在客户端发送数据包被网络阻塞了，而此时服务端的进程重启了，于是就会发送 RST 报文来断开连接。

- 紧接着，客户端又与服务端建立了与上一个连接相同四元组的连接；

- 在新连接建立完成后，上一个连接中被网络阻塞的数据包正好抵达了服务端，刚好该数据包的序列号正好是在服务端的接收窗口内，所以该数据包会被服务端正常接收，就会造成数据错乱。

  ​

每次建立连接客户端和服务端的初始化序列号都「不一样」，就有大概率因为历史报文的序列号「不在」对方接收窗口，从而很大程度上避免了历史报文，比如下图：

![图片](https://img-blog.csdnimg.cn/img_convert/ac7b6d00b01a414d37aceaa011e931fe.png)



**序列号和初始化序列号并不是无限递增的（上限4G），会发生回绕为初始值的情况，这意味着无法根据序列号来判断新老数据**。

为了解决这个问题，所以我们使用了tcp_timestamps参数，它能够精确计算RTT(一个请求发送到接受响应经过的时间)，并且能够给防止序列号回绕（PAWS）。

试看下面的示例，假设 TCP 的发送窗口是 1 GB，并且使用了时间戳选项，发送方会为每个 TCP 报文分配时间戳数值，我们假设每个报文时间加 1，然后使用这个连接传输一个 6GB 大小的数据流。

![图片](https://img-blog.csdnimg.cn/img_convert/1d497c38621ebc44ee3d8763fd03da67.png)

32 位的序列号在时刻 D 和 E 之间回绕。假设在时刻B有一个报文丢失并被重传，又假设这个报文段在网络上绕了远路并在时刻 F 重新出现。如果 TCP 无法识别这个绕回的报文，那么数据完整性就会遭到破坏。

使用时间戳选项能够有效的防止上述问题，如果丢失的报文会在时刻 F 重新出现，由于它的时间戳为 2，小于最近的有效时间戳（5 或 6），因此防回绕序列号算法（PAWS）会将其丢弃。

防回绕序列号算法要求连接双方维护最近一次收到的数据包的时间戳（Recent TSval），每收到一个新数据包都会读取数据包中的时间戳值跟 Recent TSval 值做比较，**如果发现收到的数据包中时间戳不是递增的，则表示该数据包是过期的，就会直接丢弃这个数据包**。

**客户端和服务端的初始化序列号都是随机生成，能很大程度上避免历史报文被下一个相同四元组的连接接收，然后又引入时间戳的机制，从而完全避免了历史报文被接收的问题。**



# SYN报文什么时候会被丢弃

1. **开启tcp_tw_recycle和tcp_timestamps** 

   **net.ipv4.tcp_tw_recycle**，如果开启该选项的话，允许处于 TIME_WAIT 状态的连接被快速回收；

   如果同时开启了recycle 和 timestamps 选项，则会开启一种称之为「 per-host 的 PAWS 机制」。

   开启了 recycle 和 timestamps 选项，就会开启一种叫 per-host 的 PAWS 机制。**per-host 是对「对端 IP 做 PAWS 检查」**，而非对「IP + 端口」四元组做 PAWS 检查。

   Per-host PAWS 机制利用TCP option里的 timestamp 字段的增长来判断串扰数据，而 timestamp 是根据客户端各自的 CPU tick 得出的值。

   当客户端 A 通过 NAT 网关和服务器建立 TCP 连接，然后服务器主动关闭并且快速回收 TIME-WAIT 状态的连接后，**客户端 B 也通过 NAT 网关和服务器建立 TCP 连接，注意客户端 A 和 客户端 B 因为经过相同的 NAT 网关，所以是用相同的 IP 地址与服务端建立 TCP 连接，如果客户端 B 的 timestamp 比 客户端 A 的 timestamp 小，那么由于服务端的 per-host 的 PAWS 机制的作用，服务端就会丢弃客户端主机 B 发来的 SYN 包**。

2. 半连接队列或者全连接队列满了，会丢弃发送的SYN包



# 已建立的TCP连接，收到SYN包会发生什么？

已经建立的 TCP 连接，客户端中途宕机了，而服务端此时也没有数据要发送，一直处于 establish 状态，客户端恢复后，向服务端建立连接，此时服务端会怎么处理？

1. 客户端的SYN报文的端口号跟历史连接不同，那么会创建新的tcp连接。那么处于 establish 状态的服务端会怎么处理呢？ **如果服务端有数据发送给旧客户端，那么旧客户端的内核会返回RST报文，服务器收到后就释放连接，如果没有数据发送，在一定时间后，tcp的保活机制keep-alive会发送探测报文，探测到客户端已经关闭，那么服务器就是释放掉连接**。
2. 客户端的SYN报文的端口号跟历史连接相同，**处于 establish 状态的服务端如果收到了客户端的 SYN 报文（注意此时的 SYN 报文其实是乱序的，因为 SYN 报文的初始化序列号其实是一个随机数），会回复一个携带了正确序列号和确认号的 ACK 报文，这个 ACK 被称之为 Challenge ACK。接着，客户端收到这个 Challenge ACK，发现序列号并不是自己期望收到的，于是就会回 RST 报文，服务端收到后，就会释放掉该连接。**



那么如何关闭一个tcp连接？

**我们可以伪造一个四元组相同的 SYN 报文，来拿到“合法”的序列号！**

如果处于 establish 状态的服务端，收到四元组相同的 SYN 报文后，**会回复一个 Challenge ACK，这个 ACK 报文里的「确认号」，正好是服务端下一次想要接收的序列号，说白了，就是可以通过这一步拿到服务端下一次预期接收的序列号。**

**然后用这个确认号作为 RST 报文的序列号，发送给服务端，此时服务端会认为这个 RST 报文里的序列号是合法的，于是就会释放连接！**



# 四次挥手中收到乱序的 FIN 包会如何处理？

![img](https://img-blog.csdnimg.cn/ccabc2f21b014c6c9118cd29ae11c18c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5p6XY29kaW5n,size_20,color_FFFFFF,t_70,g_se,x_16)

**在 FIN_WAIT_2 状态时，如果收到乱序的 FIN 报文，那么就被会加入到「乱序队列」，并不会进入到 TIME_WAIT 状态。**

**等再次收到前面被网络延迟的数据包时，会判断乱序队列有没有数据，然后会检测乱序队列中是否有可用的数据，如果能在乱序队列中找到与当前报文的序列号保持的顺序的报文，就会看该报文是否有 FIN 标志，如果发现有 FIN 标志，这时才会进入 TIME_WAIT 状态。**



# 在 TIME_WAIT 状态的 TCP 连接，收到 SYN 后会发生什么？

![图片](https://img-blog.csdnimg.cn/img_convert/39d0d04adf72fe3d37623acff9ae2507.png)

如果双方开启了时间戳机制：

- 如果客户端的 SYN 的「序列号」比服务端「期望下一个收到的序列号」要**大**，**并且**SYN 的「时间戳」比服务端「最后收到的报文的时间戳」要**大**。那么就会重用该四元组连接，跳过 2MSL 而转变为 SYN_RECV 状态，接着就能进行建立连接过程。
- 如果客户端的 SYN 的「序列号」比服务端「期望下一个收到的序列号」要**小**，**或者**SYN 的「时间戳」比服务端「最后收到的报文的时间戳」要**小**。那么就会**再回复一个第四次挥手的 ACK 报文，客户端收到后，发现并不是自己期望收到确认号，就回 RST 报文给服务端**。

在 TIME_WAIT 状态，收到 RST 会断开连接吗？

- 如果 `net.ipv4.tcp_rfc1337` 参数为 0，则提前结束 TIME_WAIT 状态，释放连接。
- 如果 `net.ipv4.tcp_rfc1337` 参数为 1，则会丢掉该 RST 报文。



# TCP 连接，一端断电和进程崩溃有什么区别？

**断电：**

客户端断电了，服务端是**无法感知到的**，在加上服务端没有开启 TCP keepalive，又没有数据交互的情况下，**服务端的 TCP 连接将会一直处于 ESTABLISHED 连接状态**，直到服务端重启进程。

所以，我们可以得知一个点，在没有使用 TCP 保活机制且双方不传输数据的情况下，一方的 TCP 连接处在 ESTABLISHED 状态，并不代表另一方的连接还一定正常。



**进程崩溃：**

使用 kill -9 来模拟进程崩溃的情况，发现**在 kill 掉进程后，服务端会发送 FIN 报文，与客户端进行四次挥手**。



**客户端主机宕机，又迅速重启:**

在客户端主机宕机后，服务端向客户端发送的报文会得不到任何的响应，在一定时长后，服务端就会触发**超时重传**机制，重传未得到响应的报文。

服务端重传报文的过程中，客户端主机重启完成后，客户端的内核就会接收重传的报文，然后根据报文的信息传递给对应的进程：

- 如果客户端主机上**没有**进程监听该 TCP 报文的目标端口号，那么客户端内核就会**回复 RST 报文，重置该 TCP 连接*；
- 如果客户端主机上**有**进程监听该 TCP 报文的目标端口号，由于客户端主机重启后，之前的 TCP 连接的数据结构已经丢失了，客户端内核里协议栈会发现找不到该 TCP 连接的 socket 结构体，于是就会**回复 RST 报文，重置该 TCP 连接**。

所以，只要有一方重启完成后，收到之前 TCP 连接的报文，都会回复 RST 报文，以断开连接



**客户端主机宕机，一直没有重启:**

​	服务器会一直进行超时重传，直到达到重传上线，断开连接。 **tcp_retries2** 字段控制重传次数。