# Tcp三次握手和四次挥手

## 前言

TCP 属于传输层协议，是面向有连接，可靠的流协议。面对有连接这个特性，TCP 就有建立连接和断开连接的过程。我们分别了解建立连接和断开连接的流程以及当中的一些疑问。

## TCP 建立连接和断开连接流程

首先我们来看下这张经典的流程图：

![img](../resources/network-tcp3.jpg)

握手过程可以简化为下面的四次交互：

1. Client 端首先发送一个 SYN 包，告诉 Server 端我的初始序列号是 X；Client 端进入了 SYN-SENT（同步已发送状态）状态。

2. Server 端收到 SYN 包后回复给 Client 一个 ACK 确认包，告诉 Client 说我收到了；Server 端进入了SYN-RCVD（同步收到）状态。

3. 接着 Server 端也需要告诉 Client 端自己的初始序列号，于是 Server 也发送一个 SYN 包告诉 Client 我的初始序列号是Y；

4. Client 端收到后，回复 Server 一个 ACK 确认包说我知道了。之后 Client 和 Server 进入ESTABLISHED（已建立连接）状态。


重点：Server 的 ACK 确认包和接下来的 SYN 包合成一个SYN ACK包一起发送的，没必要分别单独发送，这样子三次握手在进行最少次交互的情况下完成了两端的资源分配和初始化序列号的交换。

对于挥手过程，就不复述了，可以参阅：[TCP连接的释放（四次挥手）](https://blog.csdn.net/qzcsu/article/details/72861891)。

## TCP 的三次握手改成两次握手可以吗？

不可以，一句话，主要防止已经失效的连接请求报文突然又传送到了服务器，从而产生错误。

如果使用的是两次握手建立连接，假设有这样一种场景，客户端发送了第一个请求连接并且没有丢失，只是因为在网络结点中滞留的时间太长了，由于TCP的客户端迟迟没有收到确认报文，以为服务器没有收到，此时重新向服务器发送这条报文，此后客户端和服务器经过两次握手完成连接，传输数据，然后关闭连接。此时此前滞留的那一次请求连接，网络通畅了到达了服务器，这个报文本该是失效的，但是，两次握手的机制将会让客户端和服务器再次建立连接，这将导致不必要的错误和资源的浪费。

如果采用的是三次握手，就算是那一次失效的报文传送过来了，服务端接受到了那条失效报文并且回复了确认报文，但是客户端不会再次发出确认。由于服务器收不到确认，就知道客户端并没有请求连接。

[TCP的三次握手与四次挥手（详解+动图）](https://blog.csdn.net/qzcsu/article/details/72861891)

## TCP 四次挥手能不能变成三次挥手呢？

> 答案是可能的。
>
> TCP是全双工通信，Client 在自己已经不会在有新的数据要发送给 Server 后，可以发送 FIN 信号告知 Server，这边已经终止 Client 到对端 Server 那边的数据传输。但是，这个时候对端 Server 可以继续往 Client 这边发送数据包。于是，两端数据传输的终止在时序上是独立并且可能会相隔比较长的时间，这个时候就必须最少需要2+2 = 4 次挥手来完全终止这个连接。但是，如果Server在收到Client的FIN包后，在也没数据需要发送给Client了，那么对Client的ACK包和Server自己的FIN包就可以合并成为一个包发送过去，这样四次挥手就可以变成三次了(似乎linux协议栈就是这样实现的)。
>
> [不为人知的网络编程(一)：浅析TCP协议中的疑难杂症(上篇)](http://www.52im.net/thread-1003-1-1.html)