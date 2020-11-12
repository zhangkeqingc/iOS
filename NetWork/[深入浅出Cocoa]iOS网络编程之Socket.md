# [深入浅出Cocoa]iOS网络编程之Socket



### 一，iOS网络编程层次模型



在前文《[深入浅出Cocoa之Bonjour网络编程](http://www.cnblogs.com/kesalin/archive/2011/09/15/cocoa_bonjour.html)》中我介绍了如何在Mac系统下进行 Bonjour 编程，在那篇文章中也介绍过 Cocoa 中网络编程层次结构分为三层，虽然那篇演示的是 Mac 系统的例子，其实对iOS系统来说也是一样的。iOS网络编程层次结构也分为三层：

- *Cocoa层：NSURL，Bonjour，Game Kit，WebKit*
- *Core Foundation层：\*基于 C 的\* CFNetwork 和 CFNetServices*
- *OS层:基于 C 的 BSD socket*

Cocoa层是最上层的基于 Objective-C 的 API，比如 URL访问，NSStream，Bonjour，GameKit等，这是大多数情况下我们常用的 API。Cocoa 层是基于 Core Foundation 实现的。

Core Foundation层：因为直接使用 socket 需要更多的编程工作，所以苹果对 OS 层的 socket 进行简单的封装以简化编程任务。该层提供了 CFNetwork 和 CFNetServices，其中 CFNetwork 又是基于 CFStream 和 CFSocket。

OS层：最底层的 BSD socket 提供了对网络编程最大程度的控制，但是编程工作也是最多的。因此，苹果建议我们使用 Core Foundation 及以上层的 API 进行编程。

本文将介绍如何在 iOS 系统下使用最底层的 socket 进行编程，这和在 window 系统下使用 C/C++ 进行 socket 编程并无多大区别。

本文源码：https://github.com/kesalin/iOSSnippet/tree/master/KSNetworkDemo

运行效果如下：

![img](https://img-my.csdn.net/uploads/201304/13/1365857293_3594.png)



### 二，BSD socket API 简介



BSD socket API 和 winsock API 接口大体差不多，下面将列出比较常用的 API：

| API接口                                                      | 讲解                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| int **socket**(int addressFamily, int type,int protocol) ；int **close**(int socketFileDescriptor) | socket 创建并初始化 socket，返回该 socket 的文件描述符，如果描述符为 -1 表示创建失败。 通常参数 addressFamily 是 IPv4(AF_INET) 或 IPv6(AF_INET6)。type 表示 socket 的类型，通常是流stream(SOCK_STREAM) 或数据报文datagram(SOCK_DGRAM)。protocol 参数通常设置为0，以便让系统自动为选择我们合适的协议，对于 stream socket 来说会是 TCP 协议(IPPROTO_TCP)，而对于 datagram来说会是 UDP 协议(IPPROTO_UDP)。 close 关闭 socket。 |
| int **bind**(int socketFileDescriptor,     sockaddr *addressToBind,      int addressStructLength) | 将 socket 与特定主机地址与端口号绑定，成功绑定返回0，失败返回 -1。成功绑定之后，根据协议(TCP/UDP)的不同，我们可以对 socket 进行不同的操作：UDP：因为 UDP 是无连接的，绑定之后就可以利用 UDP socket 传送数据了。TCP：而 TCP 是需要建立端到端连接的，为了建立 TCP 连接服务器必须调用 listen(int socketFileDescriptor, int backlogSize) 来设置服务器的缓冲区队列以接收客户端的连接请求，backlogSize 表示客户端连接请求缓冲区队列的大小。当调用 listen 设置之后，服务器等待客户端请求，然后调用下面的 accept 来接受客户端的连接请求。 |
| int **accept**(int socketFileDescriptor,                sockaddr *clientAddress, int              clientAddressStructLength) | 接受客户端连接请求并将客户端的网络地址信息保存到 clientAddress 中。当客户端连接请求被服务器接受之后，客户端和服务器之间的链路就建立好了，两者就可以通信了。 |
| int  **connect**(int socketFileDescriptor,                sockaddr *serverAddress, int                serverAddressLength) | 客户端向特定网络地址的服务器发送连接请求，连接成功返回0，失败返回 -1。当服务器建立好之后，客户端通过调用该接口向服务器发起建立连接请求。对于 UDP 来说，该接口是可选的，如果调用了该接口，表明设置了该 UDP socket 默认的网络地址。对 TCP socket来说这就是传说中三次握手建立连接发生的地方。注意：该接口调用会阻塞当前线程，直到服务器返回。 |
| hostent* **gethostbyname**(char *hostname)                   | 使用 DNS 查找特定主机名字对应的 IP 地址。如果找不到对应的 IP 地址则返回 NULL。 |
| int  **send**(int socketFileDescriptor, char *buffer, int bufferLength, int flags) | 通过 socket 发送数据，发送成功返回成功发送的字节数，否则返回 -1。一旦连接建立好之后，就可以通过 send/receive 接口发送或接收数据了。注意调用 connect 设置了默认网络地址的 UDP socket 也可以调用该接口来接收数据。 |
| int  **receive**(int socketFileDescriptor,                char *buffer, int bufferLength, int flags) | 从 socket 中读取数据，读取成功返回成功读取的字节数，否则返回 -1。一旦连接建立好之后，就可以通过 send/receive 接口发送或接收数据了。注意调用 connect 设置了默认网络地址的 UDP socket 也可以调用该接口来发送数据。 |
| int   **sendto**(int socketFileDescriptor,                char *buffer, int bufferLength, int                flags, sockaddr *destinationAddress, int                destinationAddressLength) | 通过UDP socket 发送数据到特定的网络地址，发送成功返回成功发送的字节数，否则返回 -1。由于 UDP 可以向多个网络地址发送数据，所以可以指定特定网络地址，以向其发送数据。 |
| int  **recvfrom**(int socketFileDescriptor,                   char *buffer, int bufferLength, int      flags, sockaddr *fromAddress, int       *fromAddressLength) | 从UDP socket 中读取数据，并保存发送者的网络地址信息，读取成功返回成功读取的字节数，否则返回 -1 。由于 UDP 可以接收来自多个网络地址的数据，所以需要提供额外的参数，以保存该数据的发送者身份。 |





### 三，服务器工作流程



有了上面的 socket API 讲解，下面来总结一下服务器的工作流程。

1. 服务器调用 socket(...) 创建socket；
2. 服务器调用 listen(...) 设置缓冲区；
3. 服务器通过 accept(...)接受客户端请求建立连接；
4. 服务器与客户端建立连接之后，就可以通过 send(...)/receive(...)向客户端发送或从客户端接收数据；
5. 服务器调用 close 关闭 socket；

由于 iOS 设备通常是作为客户端，因此在本文中不会用代码来演示如何建立一个iOS服务器，但可以参考前文：《[深入浅出Cocoa之Bonjour网络编程](http://www.cnblogs.com/kesalin/archive/2011/09/15/cocoa_bonjour.html)》看看如何在 Mac 系统下建立桌面服务器。

 

### 四，客户端工作流程

由于 iOS 设备通常是作为客户端，下文将演示如何编写客户端代码。先来总结一下客户端工作流程。

1. 客户端调用 socket(...) 创建socket；
2. 客户端调用 connect(...) 向服务器发起连接请求以建立连接；
3. 客户端与服务器建立连接之后，就可以通过 send(...)/receive(...)向客户端发送或从客户端接收数据；
4. 客户端调用 close 关闭 socket；

###  

### 五，客户端代码示例

下面的代码就实现了上面客户端的工作流程：

```
- (void)loadDataFromServerWithURL:(NSURL *)url
{
    NSString * host = [url host];
    NSNumber * port = [url port];
    
    // Create socket
    //
    int socketFileDescriptor = socket(AF_INET, SOCK_STREAM, 0);
    if (-1 == socketFileDescriptor) {
        NSLog(@"Failed to create socket.");
        return;
    }
    
    // Get IP address from host
    //
    struct hostent * remoteHostEnt = gethostbyname([host UTF8String]);
    if (NULL == remoteHostEnt) {
        close(socketFileDescriptor);
        
        [self networkFailedWithErrorMessage:@"Unable to resolve the hostname of the warehouse server."];
        return;
    }
    
    struct in_addr * remoteInAddr = (struct in_addr *)remoteHostEnt->h_addr_list[0];
    
    // Set the socket parameters
    //
    struct sockaddr_in socketParameters;
    socketParameters.sin_family = AF_INET;
    socketParameters.sin_addr = *remoteInAddr;
    socketParameters.sin_port = htons([port intValue]);
    
    // Connect the socket
    //
    int ret = connect(socketFileDescriptor, (struct sockaddr *) &socketParameters, sizeof(socketParameters));
    if (-1 == ret) {
        close(socketFileDescriptor);
        
        NSString * errorInfo = [NSString stringWithFormat:@" >> Failed to connect to %@:%@", host, port];
        [self networkFailedWithErrorMessage:errorInfo];
        return;
    }
    
    NSLog(@" >> Successfully connected to %@:%@", host, port);

    NSMutableData * data = [[NSMutableData alloc] init];
    BOOL waitingForData = YES;
    
    // Continually receive data until we reach the end of the data
    //
    int maxCount = 5;   // just for test.
    int i = 0;
    while (waitingForData && i < maxCount) {
        const char * buffer[1024];
        int length = sizeof(buffer);
        
        // Read a buffer's amount of data from the socket; the number of bytes read is returned
        //
        int result = recv(socketFileDescriptor, &buffer, length, 0);
        if (result > 0) {
            [data appendBytes:buffer length:result];
        }
        else {
            // if we didn't get any data, stop the receive loop
            //
            waitingForData = NO;
        }
        
        ++i;
    }
    
    // Close the socket
    //
    close(socketFileDescriptor);
    
    [self networkSucceedWithData:data];
}
```

前面说过，connect/recv/send 等接口都是阻塞式的，因此我们需要将这些操作放在非 UI 线程中进行。如下所示：

```
    NSThread * backgroundThread = [[NSThread alloc] initWithTarget:self
                                                          selector:@selector(loadDataFromServerWithURL:)
                                                            object:url];
    [backgroundThread start];
```

同样，在获取到数据或者网络异常导致任务失败，我们需要更新 UI，这也要回到 UI 线程中去做这个事情。如下所示：

```
- (void)networkFailedWithErrorMessage:(NSString *)message
{
    // Update UI
    //
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        NSLog(@"%@", message);

        self.receiveTextView.text = message;
        self.connectButton.enabled = YES;
        [self.networkActivityView stopAnimating];
    }];
}

- (void)networkSucceedWithData:(NSData *)data
{
    // Update UI
    //
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        NSString * resultsString = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@" >> Received string: '%@'", resultsString);
        
        self.receiveTextView.text = resultsString;
        self.connectButton.enabled = YES;
        [self.networkActivityView stopAnimating];
    }];
}
```

  



转载：https://blog.csdn.net/kesalin/article/details/8798039

