#### 网络通信socket

socket翻译为套接字，可以理解为IP地址与端口号的组合。

socket提供了流（stream）和数据报（datagram）两种通信机制

流socket(SOCK_STREAM)和数据报socket(SOCK_DGRAM)。

流socket基于TCP协议，是一个有序、可靠、双向字节流的通道，传输数据不会丢失、不会重复、顺序也不会错乱。

数据报socket基于UDP协议，不需要建立和维持连接，可能会丢失或错乱。

大概通讯过程：

c/s 模型：

-  服务端： 
  - 创建流式socket ： socket();
  - 指定IP地址和端口： bind();
  - 将socket设置为监听模式： listen();
  - 开始接收客户端的连接：accept();
  - 接收/发送数据： recv()/send();
  - 关闭socket连接，释放资源：close()；
- 客户端：
  - 创建流式socket: socket();
  - 向服务器发起连接请求：connect();
  - 发送和接收数据：send()/recv();
  - 关闭socket连接释放资源： close()；