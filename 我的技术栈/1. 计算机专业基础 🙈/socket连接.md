# linux内核 - socket连接

这里的所有内容都是linux内核当中的，和用户态进程没有任何关系



## 三次握手

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/v2-deea5fd11df64b45c7f325a90cef52bf_1440w.jpg" alt="img" style="zoom:80%;" />



---

```阶段0```：客户端和服务端创建socket

不管是客户端还是服务端，都是通过下面的函数创建一个socket，得到 socketfd

```c
#include <sys/socket.h>
int socket(int family, int type, int protocol);
```

当然linux系统并不是只有一个socket文件描述符，在内存当中会创建一系列socket相关对象



**消耗**：操作系统内存资源（一个socket相关对象约占用3KB内存），客户端服务端都需要创建

**状态**：*CLOSE* 状态





---

```阶段1```：服务器端调用listen函数，开始监听某个端口



 listen函数会将一个套接字标识为监听套接字，用来接收连接请求，同时会创建两个队列

1. 未完成连接队列（）：收到客户端的SYN包，但还没收到客户端的ACK包，会将这样的连接放到未连接队列当中，一旦接收到了客户端的ACK包，就移到已完成连接队列当中；
2. 已完成连接队列（）：保存已经完成TCP三次握手的连接，accept函数就是从这个队列取走就绪的客户端连接



**消耗**：未完成连接队列、已完成连接队列

**状态**：服务端进入 *LISTEN* 状态



**问题**：连接队列长度限制，TCP SYN洪泛攻击（DDos）

backlog如果设置的太小，当有大并发客户端请求时，由于应用程序可能没有及时accept将linux内核的就绪连接取走，那么就可能出现未完成连接队列和已完成连接队列都满的情况，这个时候，操作系统会拒绝连接，或者直接忽略连接请求，等待客户端重新发起连接。