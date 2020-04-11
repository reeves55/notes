# Linux网络编程

网络编程关注的是如何通过网络实现两端之间的数据传输。



## 建立连接

要想进行两端之间的数据传输，首先要在两台计算机之间建立TCP/UDP连接。如何建立连接呢？Linux中使用```socket```来实现端点的概念，一个```socket```就是数据传输中的一个端点。所以socket总是成对出现的，这样才能进行数据传输。

为了实现连接，有客户端和服务器的概念，要想建立连接，就必须有主动发起连接的和被动接受连接的。主动发起连接的叫客户端，被动连接的叫服务器端。



#### 1. 客户端准备



```c
#include<sys/socket.h>

/**
 * 1. 创建一个socket实例
 * 成功时返回文件描述符，失败时返回-1
 */
int socket(int domain, int type, int protocol);

/**
 * 2. 连接到服务器端
 * 成功时返回0，失败时返回-1
 * sockfd就是上面socket()函数返回的文件描述符
 */
int connect(int sockfd, struct sockaddr *serv_addr, socklen_t addrlen);


// 例子
client_socket = socket()
```





#### 2. 服务器端准备

