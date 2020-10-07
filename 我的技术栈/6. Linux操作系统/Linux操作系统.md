# Linux操作系统



## 用户态内核态



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200814140319683.png" alt="image-20200814140319683" style="zoom:50%;" />



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/70.png" alt="这里写图片描述" style="zoom:50%;" />



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/70-20200814140757049.png" alt="这里写图片描述" style="zoom:70%;" />





### 切换时机



1. 系统调用（软中断）：int 80H
2. 异常
3. 硬件中断













## 进程







## 网络I/O



Linux中的网络I/O模型一共有5种：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200321115505373.png" alt="image-20200321115505373" style="zoom:45%;float:left" />

这5种I/O模型里面，只有异步I/O是异步的，其他都是同步的，这里面讲的同步指的是在 <span style="color:red">有数据可读写的时候</span>，调用```read``` 或者 ```write``` 的线程会阻塞，直到读写完毕，而异步指的是调用操作的线程只管发起操作，具体 <span style="color:red">把数据从内核空间复制到用户空间</span> 由内核管理，等到读或者写完成，内核会调用我们注册的回调函数通知我们。

这几个I/O模型还有一个阻塞和非阻塞的概念，这里的非阻塞指的是调用 ```read``` 或者 ```write``` 的线程会立马知道结果，如果没有数据可读或者可写，那么就返回一个 ```EAGAIN``` 错误（注意POSIX规范并没有规定write方法在不可以写的时候一定以非阻塞的方式返回错误，也可能直接阻塞住），<span style="color:red">关注点在于是否有相应的事件，在监听是否有事件这件事儿上，是非阻塞的，可以立马知道结果</span>，而阻塞的意思就是，如果没有相应的事件，就一直等着，知道有相应事件，再接着读取数据。



参考：https://linux.die.net/man/7/socket





网络编程关注的是如何通过网络实现两端之间的数据传输。



### 建立连接

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

