# 面试题目
```
1. 进程之间可以通过哪些方式通讯？各有什么优缺点？
2. 什么是socket？
3. 同一台计算机上，不同进程可以同时间监听同一个端口吗？
3. tcp和udp的区别？
```

# socket基础

## 什么是socket

> 进程之间可以通过哪些方式通讯？

共享内存、socket、管道、消息队列、信号。

socket是一种操作系统提供的进程之间通讯机制，可以是同一台机器之内的进程之间的通讯，也可以是不同机器通过网络进行通讯。

那么如何去识别不同的进程，尤其是通过网络，和其他机器上的进程进行通讯的情况下。两个进程如果需要进行通讯最基本的一个前提能能够唯一的标示一个进程，在本地进程通讯中我们可以使用PID来唯一标示一个进程，但PID只在本地唯一，网络中的两个进程PID冲突几率很大，这时候我们需要另辟它径了，我们知道IP层的ip地址可以唯一标示主机，而TCP层协议和端口号可以唯一标示主机的一个进程，这样我们可以利用ip地址＋协议＋端口号唯一标示网络中的一个进程。

> 同一台计算机上，不同进程可以同一时间监听同一个端口吗？

由于网络中的一个进程是通过ip+协议+端口号进行区分的，所以在IP和端口号相同的情况下，使用不同协议的进程可以监听同一个端口，比如TCP和UDP的不同进程可以同时监听一个端口。

能够唯一标示网络中的进程后，它们就可以利用socket进行通信了，什么是socket呢？我们经常把socket翻译为套接字，Socket 是对 TCP/IP 协议族的一种封装。从设计模式的角度看来，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

![](../../assets/circle2/socket.jpg)

socket起源于UNIX，在Unix一切皆文件哲学的思想下，socket是一种"打开—读/写—关闭"模式的实现，服务器和客户端各自维护一个"文件"，在建立连接打开后，可以向自己文件写入内容供对方读取或者读取对方内容，通讯结束时关闭文件。



## socket通信流程

socket是"打开—读/写—关闭"模式的实现，以使用TCP协议通讯的socket为例，其交互流程大概是这样子的

![](../../assets/circle2/socket-process.png)

1. 服务器根据地址类型（ipv4,ipv6）、socket类型、协议创建socket
2. 服务器为socket绑定ip地址和端口号
3. 服务器socket监听端口号请求，随时准备接收客户端发来的连接，这时候服务器的socket并没有被打开
4. 客户端创建socket
5. 客户端打开socket，根据服务器ip地址和端口号试图连接服务器socket
6. 服务器socket接收到客户端socket请求，被动打开，开始接收客户端请求，直到客户端返回连接信息。这时候socket进入阻塞状态，所谓阻塞即accept()方法一直到客户端返回连接信息后才返回，开始接收下一个客户端谅解请求
7. 客户端连接成功，向服务器发送连接状态信息
8. 服务器accept方法返回，连接成功
9. 客户端向socket写入信息
10. 服务器读取信息
11. 客户端关闭
12. 服务器端关闭

# 相关问题

> tcp和udp的区别

TCP：是面向连接的流传输控制协议，具有高可靠性，确保传输数据的正确性，有验证重发机制，因此不会出现丢失或乱序。

UDP：是无连接的数据报服务，不对数据报进行检查与修改，无须等待对方的应答，会出现分组丢失、重复、乱序，但具有较好的实时性，UDP段结构比TCP的段结构简单，因此网络开销也小。

> 流量控制和网络拥塞

拥塞控制——网络拥塞现象是指到达通信子网中某一部分的分组数量过多,使得该部分网络来不及处理,以致引起这部分乃至整个网络性能下降的现象,严重时甚至会导致网络通信业务陷入停顿,即出现死锁现象。拥塞控制是处理网络拥塞现象的一种机制。

流量控制——数据的传送与接收过程当中很可能出现收方来不及接收的情况,这时就需要对发方进行控制,以免数据丢失。



# 接下来

了解了socket的概念，我们接下来将要使用PHP，进行基本的socket编程。这个项目中，我们创建一个简单的服务器和客户端，两者之间进行简单的通讯。



