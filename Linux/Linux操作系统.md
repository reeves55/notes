# Linux操作系统





























| 常见面试题❓                                                  |
| ------------------------------------------------------------ |
| 请描述一下用多线程怎么实现生产者消费者模型                   |
| 知道nginx的惊群现象吗？怎么解决？                            |
| 请说一下epoll的内核实现，都涉及哪些数据结构？                |
| select和epoll的区别？                                        |
| fork（）都会做哪些复制？                                     |
| 什么是写时拷贝？Fork以后，父进程打开的文件指针位置在子进程里面是否一样？ |
| 你项目中为什么使用进程池？而不是用线程池？不同场景怎么选择请列举一些例子！ |
| tcp/ip的四层协议，为什么要有传输层和网络层？                 |
| tcp/ip三次握手和四次挥手过程以及信令流程，画出来！           |
| tcp三次握手哪一个阶段会抛出异常？为什么不能两次握手，说下原因？ |
| 什么是虚拟进程？                                             |
| Linux下进程都有哪些通信方式？项目中使用全双工和半双工通信的区别？ |
| 进程和线程的区别，那你知道的都说一下！                       |
| 什么是同步/异步？你项目中写的半同步/半异步是什么意思？       |
| epoll的ET/LT模式在实现上有什么区别？内核上是两种模式是怎么实现的？ |
| vi的基本命令？                                               |
| Linux上查看系统内存使用情况的命令？                          |
| Linux上查看系统版本的命令？进程状态的命令？系统所启动服务的命令？ |
| Linux上查看linuxCPU的命令？                                  |
| 进程池和线程池的具体实现写一下！                             |
| Linux  调试核心转储文件，程序断点是如何实现的(问我会不会汇编)? |
| fwrire和write的区别，sendfile的内部实现?                     |
| libevent、Rector模式、服务器网络模型                         |
| 服务器瓶颈的定位？怎么测试定位？如何设计解决瓶颈问题？       |
| IO瓶颈的解决方案都有哪些？                                   |
| Linux怎么调试内存？                                          |
| 描述符对于服务器有什么用，感觉是TCP底层（不太会）            |
| epoll的触发模式                                              |
| 页缓存他说的pagecache？Linux内核物理页面的页面缓存机制       |
| Linux进程虚拟地址空间的分布                                  |
| epoll触发模式（二面又问了，问了2遍）                         |
| Linux进程核心转储文件的调试coredump                          |
| 进程通信                                                     |
| Libevent的实现机制                                           |
| Linu查看进程堆栈命令                                         |
| LRU的实现策略                                                |
| malloc底层的实现是什么？                                     |
| 半同步/半异步模式和work-master模式是什么？                   |
| 进程间通信共享内存的底层原理是什么？                         |
| 说说gdb常用的调试命令都有哪些？                              |
| Linux的内存分布（4G空间）？                                  |
| tcp三次握手，四次挥手，为什么是三次？为什么是四次？time_wait出现在什么时候，它的作用是什么？画出tcp报头？tcp的滑动窗口满，返回什么？ |
| 进程间通信有几种方式？你都在什么情况用到？                   |
| 页面置换算法有哪些                                           |
| 负载均衡常用算法                                             |
| 心跳包机制                                                   |
| send的返回值是什么？你刨析过什么源码？你了解哪些游戏框架？   |
| 线程同步的机制（四种锁，信号量，屏障，条件变量）             |
| 自旋锁的存在的问题以及自旋锁的底层实现                       |
| 读写锁的特点，底层实现                                       |
| 一堆数据，需要线程同步，如何实现，比较方法的优劣             |
| 自己对虚拟内存的理解,把你知道的都说出来！                    |
| tcp和udp的区别，要实现一个简单的聊天程序，选那个？           |
| epoll的两种模式的特点                                        |
| 进程和线程的区别（一直问还有没有补充的二）                   |
| select和epoll的区别？（epoll内核源码看过，从内核实现角度回答，所以回答的不错） |
| Linux相关CPU，内存，网络方面相关指令                         |
| 父子进程fork时，打开的文件的偏移量是否是相同的？             |
| 详细说明Linux虚拟地址空间                                    |
| time_wait的危害，三次握手，四次断开                          |
| epoll的ET模式时，如果数据只读了一半，也就是缓冲区的数据   只读了一点，然后又有新事件来了，怎么办? |
| Linux内核解决惊群问题                                        |
| 管道为什么是半双工的？                                       |
| Linux下文件的组织形式？                                      |
| Linux下有哪些锁机制？信号量的原理和进程间的通信？            |
| 三次握手和四次挥手的状态转换，问的很细，timewait，clostwait的特点 |
| tcp/udp协议的区别？                                          |
| 线程和进程的区别                                             |
| tcp/ip协议的拥塞控制是怎样的                                 |