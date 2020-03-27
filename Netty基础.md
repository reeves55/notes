# Netty基础

Netty是一个网络框架，在一个分布式微服务框架中，netty的重要性不言而喻，微服务框架中，所有服务之间都得靠网络通信来进行协作沟通，RPC框架dubbo等，底层网络通信也是基于netty来做。



## Why Netty

微服务之间的网络调用可能并发度非常大，那网络层组件就需要能够处理高并发的网络请求，Java NIO是一个不错的网络I/O模型，



## Netty线程模型

首先，Java NIO 网络I/O事件有几种？

* **OP_ACCEPT**：有新的就绪TCP连接，也就是和服务器完成了三次握手🤝的TCP连接，此时的连接由操作系统维护在TCP连接就绪链表中，<span style="color:red">注意，只有 listen socket 才有这个事件</span>
* **OP_CONNECT**：客户端发起的TCP连接已经建立，也就是收到了服务器的 SYN + ACK 消息
* **OP_WRITE**：发送缓冲区未满，可写入数据
* **OP_READ**：接收缓冲区不空，可以读数据



Netty基于多线程来处理网络事件，具体采用的是称作 **主从Reactor的线程模型**，首先，Reactor是一种网络I/O模型，这种模型使用一个线程去监听和处理多个socket上的网络I/O事件并处理，也就是I/O多路复用

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200325234945622.png" alt="image-20200325234945622" style="zoom:25%;" />



而 **主从Reactor模型** 指的是，在服务端应用中，由一个Reactor线程单独去处理listen socket上的 ```OP_ACCEPT``` 事件，而由其他Reactor线程去处理已经连接好的 I/O socket 上的 ```OP_READ```  和 ```OP_WRITE``` 事件，由于 ```OP_ACCEPT``` 事件使用单独的Reactor线程去处理，因此并发连接处理的性能大大提升了（和只用一个Reactor线程又去处理连接，又去处理I/O事件相比）

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/5249993-a67abc1374958c5d.jpg" alt="5249993-a67abc1374958c5d" style="zoom:55%;" />



## Netty核心组件及架构

netty的三层网络架构，包括：

* Reactor层：主要负责监听网络连接和网络读写事件，将网络层传输的数据读取到内存缓冲当中，然后触发各种网络事件（如连接创建、连接激活、读事件、写事件等），并将事件触发到Pipeline中，由Pipeline管理的职责链来进行后续的处理；
* Pipeline层：Pipeline封装了链式的Handler，
* Service层：



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/7687046-44a838e8aed36503.jpg" alt="7687046-44a838e8aed36503" style="zoom:100%; float:left" />





### 核心组件概述

无论是在服务器端还是在客户端，netty都会涉及到它包含的一些关键性的组件，包括：```NioSocketChannel```，```NioServerSocketChannel```，```NioEventLoopGroup```，```NioEventLoop```，```ChannelPipeline```，

其中，



### Channel

channel是通信双方进行数据交换的通道，根据 **交换数据的协议和特性** 不同，当然就不能只有一种channel，netty支持的channel种类有：

* UnixChannel（类Unix系统中通过文件描述符进行通信）
* DatagramChannel（支持UDP/IP协议）
* AbstractChannel（基本骨架，模板模式）
* DuplexChannel（双通道channel，input和output可单独关闭）
* UdtChannel（支持UDT协议）
* Http2StreamChannel（支持HTTP2协议）
* ServerChannel（一种接收客户端连接的channel）
* SctpChannel（支持SCTP/IP协议）



```Channel``` 接口定义了很多重要方法：

```c
Channel
|- ChannelId id();
|- EventLoop eventLoop();
|- ChannelPipeline pipeline();
|- Channel read();
|- ChannelFuture bind(SocketAddress localAddress);
|- ChannelFuture bind(SocketAddress localAddress, ChannelPromise promise);
|- ChannelFuture connect(SocketAddress remoteAddress);
|- ChannelFuture connect(SocketAddress remoteAddress, SocketAddress localAddress);
|- ChannelFuture connect(SocketAddress remoteAddress, ChannelPromise promise);
|- ChannelFuture connect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise);
|- ChannelFuture disconnect();
|- ChannelFuture disconnect(ChannelPromise promise);
|- ChannelFuture close();
|- ChannelFuture close(ChannelPromise promise);
|- ChannelFuture deregister();
|- ChannelFuture deregister(ChannelPromise promise);
|- ChannelOutboundInvoker read();
|- ChannelFuture write(Object msg);
|- ChannelFuture write(Object msg, ChannelPromise promise);
|- ChannelOutboundInvoker flush();
|- ChannelFuture writeAndFlush(Object msg, ChannelPromise promise);
|- ChannelPromise newPromise();
|- ChannelProgressivePromise newProgressivePromise();
|- ChannelFuture newSucceededFuture();
|- ChannelFuture newFailedFuture(Throwable cause);
|- ChannelPromise voidPromise();
|- Unsafe unsafe();
|- Channel parent();
|- ChannelConfig config();
|- boolean isOpen();
|- boolean isRegistered();
|- boolean isActive();
|- ChannelMetadata metadata();
|- SocketAddress localAddress();
|- ChannelFuture closeFuture();
|- boolean isWritable();
|- long bytesBeforeUnwritable();
|- long bytesBeforeWritable();
|- ByteBufAllocator alloc();
|- Channel flush();

// 辅助接口
Unsafe
|- RecvByteBufAllocator.Handle recvBufAllocHandle();
|- SocketAddress localAddress();
|- SocketAddress remoteAddress();
|- void register(EventLoop eventLoop, ChannelPromise promise);
|- void bind(SocketAddress localAddress, ChannelPromise promise);
|- void connect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise);
|- void disconnect(ChannelPromise promise);
|- void close(ChannelPromise promise);
|- void closeForcibly();
|- void deregister(ChannelPromise promise);
|- void beginRead();
|- void write(Object msg, ChannelPromise promise);
|- void flush();
|- ChannelPromise voidPromise();
|- ChannelOutboundBuffer outboundBuffer();
```



#### AbstractChannel

```Channel``` 实际上是一个聚合了好几个关键组件的类型，netty中的网络I/O核心操作都是围绕 ```Channel``` 来进行的。

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200325150113811.png" alt="image-20200325150113811" style="zoom:55%;" />

而 ```Channel``` 的核心操作在模板类 ```AbstractChannel``` 中都是直接交给 pipeline 来操作的，如下

```java
@Override
public ChannelFuture bind(SocketAddress localAddress) {
    // 直接交给pipeline来做
    return pipeline.bind(localAddress);
}

@Override
public ChannelFuture connect(SocketAddress remoteAddress, SocketAddress localAddress) {
  	// 直接交给pipeline来做  
  	return pipeline.connect(remoteAddress, localAddress);
}

@Override
public Channel read() {
  	// 直接交给pipeline来做
    pipeline.read();
    return this;
}

@Override
public ChannelFuture write(Object msg) {
  	// 直接交给pipeline来做
    return pipeline.write(msg);
}

//...
```



####TCP/IP Channel

TCP/IP 协议两种核心的channel，分别是 ```SocketChannel``` 和 ```ServerSocketChannel``` ，一个是用来 I/O 的channel（服务器端客户端都使用），一个是专门用来监听tcp连接的channel（只有服务器端使用）。

![image-20200326004213245](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200326004213245.png)



先看 ```SocketChannel```，它的几个实现都是在 ```AbstractChannel``` 骨架的基础上拓展出来的，

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200326150234205.png" alt="image-20200326150234205" style="zoom:55%;" />





再看 ```ServerSocketChannel``` ，也是在 ```AbstractChannel``` 骨架的基础上拓展出来的，

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200326150257043.png" alt="image-20200326150257043" style="zoom:55%;" />





#### 如何使用



### Channel.Unsafe



#### register





### ChannelPipeline

既然前面带了个```Channel```，那必定和前面的Channel是有关系的，```ChannelPipeline``` 在功能上是一个事件处理组件，专门用来处理 Channel上发生的网络事件（```OP_ACCEPT```, ```OP_CONNECT```, ```OP_WRITE```, ```OP_READ```），比较特别的是，```ChannelPipeline```内部是一个链表结构，链表中每个节点都是一个处理器，一个事件可以被多个处理器处理，只要这些处理器对该事件感兴趣。这也是netty应用中，可以添加自定义事件处理器的地方。

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200326172705486.png" alt="image-20200326172705486" style="zoom:50%;float:left" />



那什么时候会用到 ```ChannelPipeline``` 来处理呢❓既然ChannelPipeline是处理Channel上发生的事件的，那触发Channel事件的地方，就是要和Pipeline打交道的地方，上面说的 OP_ACCEPT、OP_CONNECT等等事件，在Netty框架当中，Pipeline关心的事件不只有这几种，具体说，以下事件都是Pipeline中可以传播的事件：

* ```ChannelRegistered```
* ```ChannelUnregistered```
* ```ChannelActive```
* ```ChannelInactive```
* ```ExceptionCaught```
* ```UserEventTriggered```
* ```ChannelRead```
* ```ChannelReadComplete```
* ```ChannelWritabilityChanged```



这些事件都是Pipeline被动触发的，得有一个线程去实际上fire这些uijm



### EventLoop

EventLoop的基本继承类图如下，常用的EventLoop都是继承自 ```SingleThreadEventLoop``` ，

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200326112445470.png" alt="image-20200326112445470" style="zoom:50%;" />



EventLoop实际上是一个 **“多面手”**，它主要有两种职能，一个是使用Reacotr模型监听注册在其上的socket的事件，并处理；另外一个是task执行器，它可以执行提交给它的可运行的任务。从 ```SingleThreadEventLoop``` 继承关系上来看，它的这个单线程，不只做一种事。

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200326112352762.png" alt="image-20200326112352762" style="zoom:50%;" />



#### NioEventLoop

```NioEventLoop``` 当中的 ```run``` 方法是NioEventLoop的核心方法











#### 启动时机



![image-20200326145026415](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200326145026415.png)







需要把ServerSocketChannel注册到NioEventLoop的Selector上，在NioEventLoop过程中，会不断地使用Selector获取到当前它所监视的socket有没有事件，如果出现事件，则调用指定SocketChannel中ChannelPipeline的事件触发方法，

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200325010553258.png" alt="image-20200325010553258" style="zoom:50%;" />















### Netty服务端

一个典型的服务端程序如下，```ServerBootstrap``` 相当于一个构造器Builder，在启动服务端时，才把```ServerBootstrap``` 中各个组件合并到一起，组成一个有机的生命体，开始运作

```java
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();

try {
    // 1. 创建服务端实例
    ServerBootstrap b = new ServerBootstrap();
    b.group(bossGroup, workerGroup)
            .channel(NioServerSocketChannel.class)
            .handler(new SimpleServerHandler())
            .childHandler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {
                }
            });
		
  	// 2. 启动服务端
    ChannelFuture f = b.bind(8888).sync();

    f.channel().closeFuture().sync();
} finally {
    bossGroup.shutdownGracefully();
    workerGroup.shutdownGracefully();
}

private static class SimpleServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channelActive");
    }

    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channelRegistered");
    }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        System.out.println("handlerAdded");
    }
}
```



















### Channel



#### 两个关键的Channel

客户端 ```NioSocketChannel``` 和服务器端 ```NioServerSocketChannel```

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/NioSocketChannel.png" alt="NioSocketChannel" style="zoom:27%;float:left" /><img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/NioServerSocketChannel.png" alt="NioServerSocketChannel" style="zoom:27%;float:left" />



















### ChannelHandler 和 ChannelHandlerContext

Netty中，有两种 ```ChannelHandler```，一种是 ```ChannelOutboundHandler```，一种是 ```ChannelInboundHandler```，

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/ChannelOutboundHandler.png" alt="ChannelOutboundHandler" style="zoom:40%;" />

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/ChannelInboundHandler.png" alt="ChannelInboundHandler" style="zoom:40%;" />



实际上，pipeline接口的默认实现类 ```DefaultPipeline``` 内部包含的是 ```ChannelHandlerContext``` 链，而不是 ```ChannelHandler``` 链，但是 ```ChannelHandlerContext``` 内部包含了 ```ChannelHandler``` 实例。

![ChannelHandlerContext](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/ChannelHandlerContext.png)



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/DefaultChannelHandlerContext.png" alt="DefaultChannelHandlerContext" style="zoom:40%;" />



```DefaultChannelHandlerContext``` 内部就包含了一个 ```ChannelHandler``` 对象

```java
final class DefaultChannelHandlerContext extends AbstractChannelHandlerContext {
	// ChannelHandlerContext 当中包含了 ChannelHandler实例
  private final ChannelHandler handler;
  
  //....
}
```



### DefaultPipeline



















netty当中的 ```Channel``` 是对jdk ```SocketChannel``` 的封装，功能是进行网络I/O操作，由于是进行网络I/O操作，所以有不少方法都提供了异步执行的版本，异步执行的版本都需要传入一个额外的 ```ChannelPromise``` 参数，这些方法在执行过程中，会同步执行结果和状态到 ```ChannelPromise``` 对象中，这样，主线程就能通过 ```ChannelPromise``` 对象来异步获取操作执行结果了。

```java
// 绑定端口
ChannelFuture bind(SocketAddress localAddress, ChannelPromise promise);

// 连接服务器端
ChannelFuture connect(SocketAddress remoteAddress, ChannelPromise promise);

// 断开网络连接
ChannelFuture disconnect(ChannelPromise promise);

// 关闭Channel
ChannelFuture close(ChannelPromise promise);

// 撤销注册
ChannelFuture deregister(ChannelPromise promise);

// 发送数据
ChannelFuture write(Object msg, ChannelPromise promise);
```

















































### Pipeline链式处理

```ChannelPipeline``` 是 ```ChannelHandler``` 的容器，负责 ```ChannelHandler``` 的管理和事件拦截与调度。



#### Pipeline事件处理流程



```shell
                                               I/O Request
                                          via Channel or
                                      ChannelHandlerContext
                                                    |
+---------------------------------------------------+---------------+
|                           ChannelPipeline         |               |
|                                                  \|/              |
|    +----------------------------------------------+----------+    |
|    |                   ChannelHandler  N                     |    |
|    +----------+-----------------------------------+----------+    |
|              /|\                                  |               |
|               |                                  \|/              |
|    +----------+-----------------------------------+----------+    |
|    |                   ChannelHandler N-1                    |    |
|    +----------+-----------------------------------+----------+    |
|              /|\                                  .               |
|               .                                   .               |
| ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
|          [method call]                      [method call]         |
|               .                                   .               |
|               .                                  \|/              |
|    +----------+-----------------------------------+----------+    |
|    |                   ChannelHandler  2                     |    |
|    +----------+-----------------------------------+----------+    |
|              /|\                                  |               |
|               |                                  \|/              |
|    +----------+-----------------------------------+----------+    |
|    |                   ChannelHandler  1                     |    |
|    +----------+-----------------------------------+----------+    |
|              /|\                                  |               |
+---------------+-----------------------------------+---------------+
                |                                  \|/
+---------------+-----------------------------------+---------------+
|               |                                   |               |
|       [ Socket.read() ]                    [ Socket.write() ]     |
|                                                                   |
|  Netty Internal I/O Threads (Transport Implementation)            |
+-------------------------------------------------------------------+
```









一个 ```NioServerSocketChannel``` 实例有哪些属性❓

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200325100200575.png" alt="image-20200325100200575" style="zoom:50%;float:left" />

有几个关键属性，



第2个步骤，绑定端口并启动服务器端，由于这个操作实际上是提交到线程池来运行的，什么时候完成操作不确定，所以这里使用了 ```sync``` 方法来等待这个步骤完成，```ChannelFuture f``` 实际上是一个 ```DefaultPromise``` 对象，其实bind这一步还能拆成两个小步骤：1. ```initAndRegister```；2. ```doBind0```；这两个步骤是有顺序的，必须执行完第一步，才能执行第二步。

```initAndRegister``` 步骤创建服务器端的ServerSocketChannel，并且注册到EventLoop中去，



```java
final ChannelFuture initAndRegister() {
    Channel channel = null;
    try {
        // 创建ServerSocketChannel
        channel = channelFactory.newChannel();
        init(channel);
    } catch (Throwable t) {
        if (channel != null) {
            // channel can be null if newChannel crashed (eg SocketException("too many open files"))
            channel.unsafe().closeForcibly();
            // as the Channel is not registered yet we need to force the usage of the GlobalEventExecutor
            return new DefaultChannelPromise(channel, GlobalEventExecutor.INSTANCE).setFailure(t);
        }
        // as the Channel is not registered yet we need to force the usage of the GlobalEventExecutor
        return new DefaultChannelPromise(new FailedChannel(), GlobalEventExecutor.INSTANCE).setFailure(t);
    }

    ChannelFuture regFuture = config().group().register(channel);
    if (regFuture.cause() != null) {
        if (channel.isRegistered()) {
            channel.close();
        } else {
            channel.unsafe().closeForcibly();
        }
    }

    return regFuture;
}
```





#### DefaultPromise

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/DefaultChannelPromise.png" alt="DefaultChannelPromise" style="zoom:50%;float:left" />

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200324225906301.png" alt="image-20200324225906301" style="zoom:60%;float:left" />



这算是一个线程池任务的执行结果，实现了 ```Future``` 接口，既然是线程池任务执行结果，那怎么知道任务执行结束了呢，通过它的 ```public ChannelPromise sync() throws InterruptedException``` 方法等待任务执行结束，同步阻塞，该方法调用了父类 ```DefaultPromise``` 类的 ```sync()```方法：

```java
@Override
public Promise<V> sync() throws InterruptedException {
    await();
    rethrowIfFailed();
    return this;
}

@Override
public Promise<V> await() throws InterruptedException {
    if (isDone()) {
        return this;
    }

    if (Thread.interrupted()) {
        throw new InterruptedException(toString());
    }

    checkDeadLock();

    synchronized (this) {
        while (!isDone()) {
            incWaiters();
            try {
                // 就是Object.wait()方法
                wait();
            } finally {
                decWaiters();
            }
        }
    }
    return this;
}
```



那么任务执行结束了，是怎么通知主线程的呢，其实就是在把任务提交到线程池的时候，把promise对象也传到任务对象当中去，









### EventLoop





<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/EventLoopGroup.png" alt="EventLoopGroup" style="zoom:45%;float:left" />





<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/NioEventLoopGroup.png" alt="NioEventLoopGroup" style="zoom:50%;float:left" />



一个实例化的 ```NioEventLoop``` 实例是这个样子的：

```c
NioEventLoopGroup
|- children(EventExecutor[8])
|  |- NioEventLoop@1
|  |- NioEventLoop@2
|  |- NioEventLoop@3
|  |- NioEventLoop@4
|- readonlyChildren(Set<EventExecutor>(8))
|  |- NioEventLoop@1
|  |- NioEventLoop@2
|  |- NioEventLoop@3
|  |- NioEventLoop@4
|- terminatedChildren(AtomicInteger(0))
|- terminationFuture(DefaultPromise)
|- chooser(DefaultEventExecutorChooserFactory)
|  |- idx(AtomicInteger(0))
|  |- executors(EventExecutor[8])
|  |  |- NioEventLoop@1
|  |  |- NioEventLoop@2
|  |  |- NioEventLoop@3
|  |  |- NioEventLoop@4
```

首先，NioEventLoop实际上是一种EventExecutor





```java
// eventloop 线程的数量默认是CPU核心数的2倍
DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt("io.netty.eventLoopThreads", NettyRuntime.availableProcessors() * 2));


// MultithreadEventLoopGroup构造方法
// 参数值分别为
// nThreads: 0
// executor: null
// args: 
protected MultithreadEventLoopGroup(int nThreads, Executor executor, Object... args) {
    super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, executor, args);
}
```









## 资料



#### 单线程Reactor模型和多线程Reactor模型

单线程Reactor模式，一个reactor线程，这个reactor线程里面其实有一个selector，由于只有一个reactor线程，所以这个selector既监听 ```OP_ACCEPT``` 事件，又监听 ```OP_READ``` 和 ```OP_WRITE``` 事件，而且重要的是，有了读事件，selector线程还得去处理这个事件，这个时候selector是无法处理 ```OP_ACCEPT``` 事件的，也就是没法再处理新连接。

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/5249993-a5f8399bf59b25c6.jpg" alt="5249993-a5f8399bf59b25c6" style="zoom:50%;float:left" />

多线程reactor模型当中，原来的selector线程专门处理listen socket上的OP_ACCEPT事件，原来的OP_READ和OP_WRITE事件就交给了新创建的 Worker Thread Pool，这个线程池当中有多个selector线程，这些线程只关注listen socket进行accept创建的新socket的OP_READ和OP_WRITE事件。这样，读写处理和接收连接两个操作就解耦分离开了，

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/5249993-5318716bb8f8cfda.jpg" alt="5249993-5318716bb8f8cfda" style="zoom:55%;float:left" />

