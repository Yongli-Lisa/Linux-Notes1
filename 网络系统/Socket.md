网络基础概念：   
<img src="https://github.com/Yongli-Lisa/Linux-Notes1/blob/7a98ac69a9979b5a15cd97b0f5d63819dcc2ea3f/Img/%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1/OSI%E6%A8%A1%E5%9E%8B.PNG" width="300px">
&emsp;
&emsp;
&emsp;

Socket概念：   
Linux系统中，二层到四层都是在 Linux 内核里面处理的，应用层都是用户态的。   
应用层和内核互通的机制，就是通过 Socket 系统调用。Socket属于操作系统的概念，而非网络协议分层的概念。   
<img src="https://github.com/Yongli-Lisa/Linux-Notes1/blob/cc012dded2b0dd0382dfe6057b937066cbde7fa6/Img/%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1/Socket%E9%80%9A%E4%BF%A1.PNG" width="600px">   
&emsp;
&emsp;
&emsp;
Socket系统调用：   
<img src="https://github.com/Yongli-Lisa/Linux-Notes1/blob/00aade7183a7dd5ffe6519bd2f5dfcbbf5814f16/Img/%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1/Socket%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8.PNG" width="600px">
&emsp;
&emsp;
&emsp;
SOCK_STREAM 是面向数据流的，协议 IPPROTO_TCP 属于这种类型。   
SOCK_DGRAM 是面向数据报的，协议 IPPROTO_UDP 属于这种类型。如果在内核里面看的话，IPPROTO_ICMP 也属于这种类型。   
SOCK_RAW 是原始的 IP 包，IPPROTO_IP 属于这种类型。
&emsp;
在内核中，为每个 Socket 维护两个队列。   
一个是已经建立了连接的队列，这时候连接三次握手已经完毕，处于 established 状态；一个是还没有完全建立连接的队列，这个时候三次握手还没完成，处于 syn_rcvd 的状态。    
服务端调用 accept 函数，其实是在第一个队列中拿出一个已经完成的连接进行处理。如果还没有完成就阻塞等待。
&emsp;
&emsp;
&emsp;

发送数据包：   
<img src="https://github.com/Yongli-Lisa/Linux-Notes1/blob/7a98ac69a9979b5a15cd97b0f5d63819dcc2ea3f/Img/%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1/%E5%8F%91%E9%80%81%E6%95%B0%E6%8D%AE%E5%8C%85.PNG" width="800px">   
&emsp;
&emsp;
&emsp;

TCP:   
连接的建立过程，也即三次握手，是 TCP 层的动作，是在内核完成的，应用层不需要参与。   
服务端只需要调用 accept，等待内核完成了至少一个连接的建立，才返回。如果没有一个连接完成了三次握手，accept 就一直等待；如果有多个客户端发起连接，并且在内核里面完成了多个三次握手，建立了多个连接，这些连接会被放在一个队列里面。accept 会从队列里面取出一个来进行处理。如果想进一步处理其他连接，需要调用多次 accept，所以 accept 往往在一个循环里面。   
接下来，客户端可以通过 connect 函数发起连接。  
一旦握手成功，服务端的 accept 就会返回另一个 socket。   
监听的 socket 和真正用来传送数据的 socket，是两个 socket，一个叫作监听 socket，一个叫作已连接 socket。成功连接建立之后，双方开始通过 read 和 write 函数来读写数据，就像往一个文件流里面写东西一样。   
总结：   
服务端和客户端都调用 socket，得到文件描述符；   
服务端调用 listen，进行监听；   
服务端调用 accept，等待客户端连接；   
客户端调用 connect，连接服务端；   
服务端 accept 返回用于传输的 socket 的文件描述符；   
客户端调用 write 写入数据；服务端调用 read 读取数据。   
<img src="https://github.com/Yongli-Lisa/Linux-Notes1/blob/41e28270bbb1164308efdec579ae0aee4732cf36/Img/%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1/TCP.PNG" width="300px">
&emsp;
&emsp;
&emsp;

UDP：   
<img src="https://github.com/Yongli-Lisa/Linux-Notes1/blob/41e28270bbb1164308efdec579ae0aee4732cf36/Img/%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1/UDP.PNG" width="300px">
