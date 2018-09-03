---
title: Socket入门及使用
date: 2017-02-20 00:09:46
categories: iOS
tags: [iOS]
comments: false
---

## Socket 简介

 以前听到Socket编程，觉得它是比较高深的编程知识，但是只要弄清Socket编程的工作原理，神秘的面纱也就揭开了。

一个生活中的场景。你要打电话给一个朋友，先拨号，朋友听到电话铃声后提起电话，这时你和你的朋友就建立起了连接，就可以讲话了。等交流结束，挂断电话结束此次交谈。

- Socket 是对 TCP/IP 协议族的一种封装，不属于协议范畴，是应用层与TCP/IP协议族通信的中间软件抽象层。从设计模式的角度看来，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

- Socket 还可以认为是一种网络间不同计算机上的进程通信的一种方法，利用三元组（ip地址，协议，端口）就可以唯一标识网络中的进程，网络中的进程通信可以利用这个标志与其它进程进行交互。

- Socket 起源于 Unix ，Unix/Linux 基本哲学之一就是“一切皆文件”，都可以用“打开(open) –> 读写(write/read) –> 关闭(close)”模式来进行操作。因此 Socket 也被处理为一种特殊的文件。

<!--more-->

## Socket 原理

### 套接字（Socket）概念

 套接字（Socket）是通信的基石，是支持TCP/IP协议的网络通信的基本操作单元。它是网络通信过程中端点的抽象表示，包含进行网络通信必须的五种信息：**连接协议，服务器地址、服务器端口、客户端地址、客户端端口**。

大多数连接都是可靠的TCP连接。创建TCP连接时，主动发起连接的叫客户端，被动响应连接的叫服务器。服务器会有大量来自客户端的连接，所以，为了区别不同的应用程序进程和连接，许多计算机操作系统为应用程序与TCP／IP协议交互提供了套接字(Socket)接口。应用层可以和传输层通过Socket接口，区分来自不同应用程序进程或网络连接的通信，实现数据传输的并发服务。

### 建立socket连接

建立Socket连接至少需要一对套接字，其中一个运行于客户端，称为ClientSocket，另一个运行于服务器端，称为ServerSocket。

套接字之间的连接过程分为三个步骤：**服务器监听，客户端请求，连接确认**。

服务器监听：服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态，等待客户端的连接请求。

客户端请求：指客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址和端口号，然后就向服务器端套接字提出连接请求。

连接确认：当服务器端套接字监听到或者说接收到客户端套接字的连接请求时，就响应客户端套接字的请求，建立一个新的线程，把服务器端套接字的描述发给客户端，一旦客户端确认了此描述，双方就正式建立连接。而服务器端套接字继续处于监听状态，继续接收其他客户端套接字的连接请求

### Socket连接与TCP连接

创建Socket连接时，可以指定使用的传输层协议，Socket可以支持不同的传输层协议（TCP或UDP），当使用TCP协议进行连接时，该Socket连接就是一个TCP连接。

### Socket连接与HTTP连接

通常情况下Socket连接就是TCP连接，因此Socket连接一旦建立，连接是长连接，理论上客户端和服务器端一旦建立连接将不会主动断开此连接。Socket连接属于请求-响应形式，服务端可主动将消息推送给客户端。但在实际网络应用中，客户端到服务器之间的通信往往需要穿越多个中间节点，例如路由器、网关、防火墙等，大部分防火墙默认会关闭长时间处于非活跃状态的连接而导致Socket 连接断连，因此需要通过轮询告诉网络，该连接处于活跃状态。

HTTP是基于请求-响应形式并且是短连接，并且是无状态的协议。不仅在请求时需要先建立连接，而且需要客户端向服务器发出请求后，服务器端才能回复数据。

很多情况下，需要服务器端主动向客户端推送数据，保持客户端与服务器数据的实时与同步。此时若双方建立的是Socket连接，服务器就可以直接将数据传送给客户端；若双方建立的是HTTP连接，则服务器需要等到客户端发送一次请求后才能将数据传回给客户端，因此，客户端定时向服务器端发送连接请求，不仅可以保持在线，同时也是在“询问”服务器是否有新的数据，如果有就将数据传给客户端。

## Socket使用

服务器端：

- socket() 创建套接字
- bind() 绑定一个端口
- listen() 监听端口
- accept() 等待客户端发送的connect请求，如果接受到就返回一个已经建立连接的SOCKET,否则继续等待
- read()/write() 数据交换
- close() 关闭连接

客户端：

- socket(),新建套接字
- connect(),向服务器端发出连接请求，如果成功则这个socket就已经与服务器端建立连接
- 利用已经建立连接的socket采用send()或recv()函数与服务器端进行数据传递
- close(),关闭socket

在iOS中以NSStream(流)来发送和接收数据,可以设置流的代理，对流状态的变化做出相应的动作(连接建立，接收到数据，连接关闭）。

NSStream：数据流的父类，用于定义抽象特性，例如：打开、关闭代理，NSStream继承自CFStream(CoreFoundation)
NSInputStream：NSStream的子类，用于读取输入
NSOutputStream：NSSTream的子类，用于写输出。

服务端先不提，客户端代码大概如下：

```
//需要导入<arpa/inet.h>，<netdb.h>
- (void)test
{
NSString * host =@"123.33.33.1";
NSNumber * port = @1233;

// 创建 socket
int socketFileDescriptor = socket(AF_INET, SOCK_STREAM, 0);
if (-1 == socketFileDescriptor) {
    NSLog(@"创建失败");
    return;
}

// 获取 IP 地址 
struct hostent * remoteHostEnt = gethostbyname([host UTF8String]);
if (NULL == remoteHostEnt) {
    close(socketFileDescriptor);
     NSLog(@"%@",@"无法解析服务器的主机名");
    return;
}

struct in_addr * remoteInAddr = (struct in_addr *)remoteHostEnt->h_addr_list[0];

// 设置 socket 参数
struct sockaddr_in socketParameters;
socketParameters.sin_family = AF_INET;
socketParameters.sin_addr = *remoteInAddr;
socketParameters.sin_port = htons([port intValue]);

// 连接 socket
int ret = connect(socketFileDescriptor, (struct sockaddr *) &socketParameters, sizeof(socketParameters));
if (-1 == ret) {
    close(socketFileDescriptor);
    NSLog(@"连接失败");
    return;
}

NSLog(@"连接成功");
}
```

大概就是这样，因为是C语言的，所以看起来不是很方便，一般开发中都会使用比较简单的方法，iOS开发中常使用github上的开源类库第三方库[**CocoaAsyncSocket**](https://github.com/robbiehanson/CocoaAsyncSocket)来简化开发，CocoaAsyncSocket是支持TCP和UDP的。代码大概如下：

```
- (IBAction)connectToServer:(id)sender {
// 1.与服务器通过三次握手建立连接
NSString *host = @"133.33.33.1";
int port = 1212;

//创建一个socket对象
_socket = [[GCDAsyncSocket alloc] initWithDelegate:self delegateQueue:dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)];

//连接
NSError *error = nil;
[_socket connectToHost:host onPort:port error:&error];

if (error) {
    NSLog(@"%@",error);
}
}


#pragma mark -socket的代理
#pragma mark 连接成功
-(void)socket:(GCDAsyncSocket *)sock didConnectToHost:(NSString *)host port:(uint16_t)port{
NSLog(@"%s",__func__);
}


#pragma mark 断开连接
-(void)socketDidDisconnect:(GCDAsyncSocket *)sock withError:(NSError *)err{
if (err) {
    NSLog(@"连接失败");
}else{
    NSLog(@"正常断开");
}
}


#pragma mark 数据发送成功
-(void)socket:(GCDAsyncSocket *)sock didWriteDataWithTag:(long)tag{
NSLog(@"%s",__func__);

//发送完数据手动读取，-1不设置超时
[sock readDataWithTimeout:-1 tag:tag];
}

#pragma mark 读取数据
-(void)socket:(GCDAsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag{
NSString *receiverStr = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
NSLog(@"%s %@",__func__,receiverStr);
}
```


