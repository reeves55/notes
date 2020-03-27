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











### Channel.Unsafe



#### register

把Channel注册到EventLoopGroup上面，主要操作是由 ```AbstractChannel.AbstractUnsafe``` 来完成的

![Channel$Unsafe.register](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/Channel$Unsafe.register.png)



```AbstractChannel.AbstractUnsafe``` 类中的关键方法

```java
@Override
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
    if (eventLoop == null) {
        throw new NullPointerException("eventLoop");
    }
  	// AbstractChannel有一个属性 registered ，已经把Channel注册到一个EventLoopGroup 就不能重复注册或者注册到另外一个EventLoopGroup了
    if (isRegistered()) {
        promise.setFailure(new IllegalStateException("registered to an event loop already"));
        return;
    }
  	// 检查兼容性❓
    if (!isCompatible(eventLoop)) {
        promise.setFailure(new IllegalStateException("incompatible event loop type: " + eventLoop.getClass().getName()));
        return;
    }
		
  	// 把注册的EventLoop实例关联到AbstractChannel对象
    AbstractChannel.this.eventLoop = eventLoop;

  	// 这个方法可能是EventLoop线程调用，也可能其他线程调用，如果其他线程调用的话，可能出现多线程并发调用，可能产生并发问题，所以，如果外部线程调用这个方法，就转换成一个task交给EventLoop线程运行，避免多线程并发问题
    if (eventLoop.inEventLoop()) {
        register0(promise);
    } else {
        try {
            eventLoop.execute(new Runnable() {
                @Override
                public void run() {
                    register0(promise);
                }
            });
        } catch (Throwable t) {
            logger.warn(
                    "Force-closing a channel whose registration task was not accepted by an event loop: {}",
                    AbstractChannel.this, t);
            closeForcibly();
            closeFuture.setClosed();
            safeSetFailure(promise, t);
        }
    }
}


private void register0(ChannelPromise promise) {
    try {
        // check if the channel is still open as it could be closed in the mean time when the register call was outside of the eventLoop
      	// 如果Channel已经close了，是不能进行注册操作的
        if (!promise.setUncancellable() || !ensureOpen(promise)) {
            return;
        }
      
      	// neverRegistered 在 AbstractChannel中 默认为false
        boolean firstRegistration = neverRegistered;
      
      	// 这是 AbstractChannel 模板类中的抽象方法，具体逻辑由子类实现
        doRegister();
      
      	// 更新Channel注册状态
        neverRegistered = false;
        registered = true;

        // Pipeline中的 handlerAdded 事件只有在 Channel注册到Eventloop上之后才能触发，这里完成了注册操作，就应该通知 Pipeline了
        pipeline.invokeHandlerAddedIfNeeded();

        safeSetSuccess(promise);
      	// 触发 Pipeline中的 ChannelRegistered 事件
        pipeline.fireChannelRegistered();
        // 对于服务器端来说，active意味着已经bind成功，对于客户端来说 active意味着已经connected
        if (isActive()) {
            if (firstRegistration) {
              	// Channel可以多次进行de-register操作再register，但是Channel的状态只有一次从inActive -> Active，这里是为了保证 ChannelActive 事件只触发一次
                pipeline.fireChannelActive();
            } else if (config().isAutoRead()) {
                // 没看懂❓
                // See https://github.com/netty/netty/issues/4805
                beginRead();
            }
        }
    } catch (Throwable t) {
        // Close the channel directly to avoid FD leak.
        closeForcibly();
        closeFuture.setClosed();
        safeSetFailure(promise, t);
    }
}
```



针对于服务器端，可以看看 ```AbstractChannel``` 的子类 ```AbstractNioChannel``` 的具体实现（ ```NioServerSocketChannel``` 就继承了 ```AbstractNioChannel```）

```java
@Override
protected void doRegister() throws Exception {
    boolean selected = false;
    for (;;) {
        try {
          	// 这里把Channel注册到eventloop中的selector上，第三个参数为0表示，暂时没有监听任何事件
          	// 可以之后对Channel的 interestOps 进行更改，这里仅仅是注册
            selectionKey = javaChannel().register(eventLoop().unwrappedSelector(), 0, this);
            return;
        } catch (CancelledKeyException e) {
            if (!selected) {
              	// 如果在执行注册的时候出现这个异常，说明这个Channel之前被de-register过，由于de-register操作不会自动把socket从selector上移除，直到运行 selector.select 方法，如果EventLoop循环还没来得及运行到 selector.select ，这里就代它执行 selector.select
                eventLoop().selectNow();
                selected = true;
            } else {
                // We forced a select operation on the selector before but the SelectionKey is still cached
                // for whatever reason. JDK bug ?
                throw e;
            }
        }
    }
}
```



#### deregister





#### bind

bind操作把Channel绑定到指定IP地址和端口

```java
@Override
public final void bind(final SocketAddress localAddress, final ChannelPromise promise) {
    assertEventLoop();
		
  	// 还是要保证进行bind操作的Channel没有close
    if (!promise.setUncancellable() || !ensureOpen(promise)) {
        return;
    }

    // See: https://github.com/netty/netty/issues/576
    if (Boolean.TRUE.equals(config().getOption(ChannelOption.SO_BROADCAST)) &&
        localAddress instanceof InetSocketAddress &&
        !((InetSocketAddress) localAddress).getAddress().isAnyLocalAddress() &&
        !PlatformDependent.isWindows() && !PlatformDependent.maybeSuperUser()) {
        // Warn a user about the fact that a non-root user can't receive a
        // broadcast packet on *nix if the socket is bound on non-wildcard address.
        logger.warn(
                "A non-root user can't receive a broadcast packet if the socket " +
                "is not bound to a wildcard address; binding to a non-wildcard " +
                "address (" + localAddress + ") anyway as requested.");
    }

    boolean wasActive = isActive();
    try {
      	// 这是模板类当中的抽象方法，具体操作由子类实现
        doBind(localAddress);
    } catch (Throwable t) {
        safeSetFailure(promise, t);
        closeIfClosed();
        return;
    }

  	// 只有 Channel 状态从 inActive -> Active 才需要触发Pipeline中 ChannelActive 事件
  	// bind成功，就代表了Channel处于 Active 状态
    if (!wasActive && isActive()) {
        invokeLater(new Runnable() {
            @Override
            public void run() {
                pipeline.fireChannelActive();
            }
        });
    }

    safeSetSuccess(promise);
}
```



```NioServerSocketChannel``` 对 ```doBind``` 方法的实现

```java
@Override
protected void doBind(SocketAddress localAddress) throws Exception {
    if (PlatformDependent.javaVersion() >= 7) {
        javaChannel().bind(localAddress, config.getBacklog());
    } else {
        javaChannel().socket().bind(localAddress, config.getBacklog());
    }
}
```



#### connect

connect操作从客户端发起，连接到服务器端，可以设置连接的超时时间

```AbstractNioChannel.Unsafe```

```java
@Override
public final void connect(
        final SocketAddress remoteAddress, final SocketAddress localAddress, final ChannelPromise promise) {
  	// 首先还是要确保Channel没有close
    if (!promise.setUncancellable() || !ensureOpen(promise)) {
        return;
    }

    try {
      	// connectPromise是当前Channel connect操作的future对象，如果connectPromise不为null，说明连接正在进行中，不允许重复尝试connect
        if (connectPromise != null) {
            // Already a connect in process.
            throw new ConnectionPendingException();
        }

        boolean wasActive = isActive();
      	// doConnect方法是模板类中的抽象方法，由具体子类实现
        if (doConnect(remoteAddress, localAddress)) {
            fulfillConnectPromise(promise, wasActive);
        } else {
          	// 连接没有立即成功
            connectPromise = promise;
            requestedRemoteAddress = remoteAddress;

            // Schedule connect timeout.
            int connectTimeoutMillis = config().getConnectTimeoutMillis();
            if (connectTimeoutMillis > 0) {
              	// 交给EventLoop一个定时任务
                connectTimeoutFuture = eventLoop().schedule(new Runnable() {
                    @Override
                    public void run() {
                        ChannelPromise connectPromise = AbstractNioChannel.this.connectPromise;
                        ConnectTimeoutException cause =
                                new ConnectTimeoutException("connection timed out: " + remoteAddress);
                      	// 到了定时器超时时间，发现连接失败
                        if (connectPromise != null && connectPromise.tryFailure(cause)) {
                            close(voidPromise());
                        }
                    }
                }, connectTimeoutMillis, TimeUnit.MILLISECONDS);
            }

            promise.addListener(new ChannelFutureListener() {
                @Override
                public void operationComplete(ChannelFuture future) throws Exception {
                    if (future.isCancelled()) {
                        if (connectTimeoutFuture != null) {
                            connectTimeoutFuture.cancel(false);
                        }
                        connectPromise = null;
                        close(voidPromise());
                    }
                }
            });
        }
    } catch (Throwable t) {
        promise.tryFailure(annotateConnectException(t, remoteAddress));
        closeIfClosed();
    }
}


private void fulfillConnectPromise(ChannelPromise promise, boolean wasActive) {
    if (promise == null) {
        // Closed via cancellation and the promise has been notified already.
        return;
    }

    // Get the state as trySuccess() may trigger an ChannelFutureListener that will close the Channel. We still need to ensure we call fireChannelActive() in this case.
    boolean active = isActive();

    // trySuccess() will return false if a user cancelled the connection attempt.
    boolean promiseSet = promise.trySuccess();

    // Regardless if the connection attempt was cancelled, channelActive() event should be triggered,because what happened is what happened.
    if (!wasActive && active) {
      	// 触发 pipeline 中的 ChannelActive 事件
        pipeline().fireChannelActive();
    }

    // If a user cancelled the connection attempt, close the channel, which is followed by channelInactive().
    if (!promiseSet) {
        close(voidPromise());
    }
}
```



```NioSocketChannel.doConnect()``` 具体实现

```java
@Override
protected boolean doConnect(SocketAddress remoteAddress, SocketAddress localAddress) throws Exception {
    if (localAddress != null) {
        doBind0(localAddress);
    }

    boolean success = false;
    try {
        boolean connected = SocketUtils.connect(javaChannel(), remoteAddress);
        if (!connected) {
            selectionKey().interestOps(SelectionKey.OP_CONNECT);
        }
        success = true;
        return connected;
    } finally {
        if (!success) {
            doClose();
        }
    }
}


private void doBind0(SocketAddress localAddress) throws Exception {
    if (PlatformDependent.javaVersion() >= 7) {
        SocketUtils.bind(javaChannel(), localAddress);
    } else {
        SocketUtils.bind(javaChannel().socket(), localAddress);
    }
}
```



#### disconnect

客户端取消连接，看下AbstractChannel.AbstractUnsafe.disconnect()方法

```java
@Override
public final void disconnect(final ChannelPromise promise) {
    assertEventLoop();

    if (!promise.setUncancellable()) {
        return;
    }

    boolean wasActive = isActive();
    try {
        doDisconnect();
    } catch (Throwable t) {
        safeSetFailure(promise, t);
        closeIfClosed();
        return;
    }

  	// 当Channel连接状态从Active -> inActive时，触发pipeline中的 ChannelInactive事件
    if (wasActive && !isActive()) {
        invokeLater(new Runnable() {
            @Override
            public void run() {
                pipeline.fireChannelInactive();
            }
        });
    }

    safeSetSuccess(promise);
    closeIfClosed(); // doDisconnect() might have closed the channel
}
```



#### beginRead

```AbstractChannel.AbstractUnsafe``` 的 ```beginRead``` 方法

```java
@Override
public final void beginRead() {
    assertEventLoop();

    if (!isActive()) {
        return;
    }

    try {
      	// 这是模板类的抽象方法，具体操作由子类实现
        doBeginRead();
    } catch (final Exception e) {
        invokeLater(new Runnable() {
            @Override
            public void run() {
                pipeline.fireExceptionCaught(e);
            }
        });
        close(voidPromise());
    }
}
```



```AbstractNioChannel``` 的具体实现为：

```java
@Override
protected void doBeginRead() throws Exception {
    // Channel.read() or ChannelHandlerContext.read() was called
    final SelectionKey selectionKey = this.selectionKey;
    if (!selectionKey.isValid()) {
        return;
    }

  	// 标识当前Channel正在读取当中
    readPending = true;

    final int interestOps = selectionKey.interestOps();
    if ((interestOps & readInterestOp) == 0) {
      	// 向selector注册 OP_READ 感兴趣事件
        selectionKey.interestOps(interestOps | readInterestOp);
    }
}
```



#### write

Unsafe的write方法实际上只是把要发送的消息对象添加到 Channel 中的 ```ChannelOutboundBuffer outboundBuffer``` 中去，```ChannelOutboundBuffer``` 内部保存了一个 ```Entry``` 节点组成的单链表，每个节点保存着每次调用write方法传进来的msg对象，

```java
@Override
public final void write(Object msg, ChannelPromise promise) {
    assertEventLoop();

    ChannelOutboundBuffer outboundBuffer = this.outboundBuffer;
    if (outboundBuffer == null) {
        // If the outboundBuffer is null we know the channel was closed and so
        // need to fail the future right away. If it is not null the handling of the rest
        // will be done in flush0()
        // See https://github.com/netty/netty/issues/2362
        safeSetFailure(promise, WRITE_CLOSED_CHANNEL_EXCEPTION);
        // release message now to prevent resource-leak
        ReferenceCountUtil.release(msg);
        return;
    }

    int size;
    try {
        msg = filterOutboundMessage(msg);
        size = pipeline.estimatorHandle().size(msg);
        if (size < 0) {
            size = 0;
        }
    } catch (Throwable t) {
        safeSetFailure(promise, t);
        ReferenceCountUtil.release(msg);
        return;
    }

    outboundBuffer.addMessage(msg, size, promise);
}
```

















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



这些事件都是Pipeline被动触发的，得有一个线程去实际上fire这些事件

```java
@Override
ChannelPipeline fireChannelRegistered();

 @Override
ChannelPipeline fireChannelUnregistered();

@Override
ChannelPipeline fireChannelActive();

@Override
ChannelPipeline fireChannelInactive();

@Override
ChannelPipeline fireExceptionCaught(Throwable cause);

@Override
ChannelPipeline fireUserEventTriggered(Object event);

@Override
ChannelPipeline fireChannelRead(Object msg);

@Override
ChannelPipeline fireChannelReadComplete();

@Override
ChannelPipeline fireChannelWritabilityChanged();
```



#### addXXX

包括 addFirst、addLast、addBefore、addAfter 这几个方法，本质上都是一样的，只不过向ChannelHandlerContext链表插入元素的位置不同而已，步骤如下：

```java
@Override
public final ChannelPipeline addXXX(EventExecutorGroup group, String name, ChannelHandler handler) {
    final AbstractChannelHandlerContext newCtx;
  	// 普通链表操作，可能出现多线程并发问题，所以加锁
    synchronized (this) {
      	// 如果handler不是 @Sharable的，这个handler不支持被多次添加到ChannelPipeline中
        checkMultiplicity(handler);
      	// 如果name==null则会自动生成一个name，name不允许和已有handler name重复
      	// 平时我们直接调用 addXXX(handler) ，这个时候name默认是null
        name = filterName(name, handler);
				
      	// new DefaultChannelHandlerContext 来封装这个 handler
        newCtx = newContext(group, name, handler);

      	// 把 ChannelHandlerContext 添加到 Pipeline
        addXXX0(newCtx);

        // If the registered is false it means that the channel was not registered on an eventloop yet.
        // In this case we add the context to the pipeline and add a task that will call
        // ChannelHandler.handlerAdded(...) once the channel is registered.
      	// registered表示当前Pipeline是否已经被Channel注册，一旦注册，registered=true，且不会改变
      	// 在 Channel.Unsafe.register 章节可以看到，一旦Channel注册到Eventloop，就会调用Pipeline的 invokeHandlerAddedIfNeeded() 方法
        if (!registered) {
            newCtx.setAddPending();
          	// 等到Pipeline被注册之后，再触发当前Handler的handlerAdded事件
            callHandlerCallbackLater(newCtx, true);
            return this;
        }

        EventExecutor executor = newCtx.executor();
        if (!executor.inEventLoop()) {
            newCtx.setAddPending();
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    callHandlerAdded0(newCtx);
                }
            });
            return this;
        }
    }
    callHandlerAdded0(newCtx);
    return this;
}
```



Channel一旦注册到EventLoop上，就会通过调用 ```invokeHandlerAddedIfNeeded()``` 方法来通知Pipeline

```java
final void invokeHandlerAddedIfNeeded() {
    assert channel.eventLoop().inEventLoop();
    if (firstRegistration) {
        firstRegistration = false;
        // We are now registered to the EventLoop. It's time to call the callbacks for the ChannelHandlers,
        // that were added before the registration was done.
        callHandlerAddedForAllHandlers();
    }
}

private void callHandlerAddedForAllHandlers() {
    final PendingHandlerCallback pendingHandlerCallbackHead;
    synchronized (this) {
        assert !registered;

        // This Channel itself was registered.
        registered = true;

        pendingHandlerCallbackHead = this.pendingHandlerCallbackHead;
        // Null out so it can be GC'ed.
        this.pendingHandlerCallbackHead = null;
    }

    // This must happen outside of the synchronized(...) block as otherwise handlerAdded(...) may be called while
    // holding the lock and so produce a deadlock if handlerAdded(...) will try to add another handler from outside
    // the EventLoop.
    PendingHandlerCallback task = pendingHandlerCallbackHead;
    while (task != null) {
        task.execute();
        task = task.next;
    }
}
```





###ChannelHandler

ChannelHandler是事件处理器，用于接收Pipeline中传播的事件并处理，但ChannelHandler并不是直接包含于Pipeline中，而是封装成ChannelHandlerContext对象，多个ChannelHandlerContext组成一个双向链表包含于Pipeline中。



####生命周期

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/temp.png" alt="temp" style="zoom:50%;" />





#### ChannelInitializer



```java
// 服务器端
ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup, workerGroup)
        .channel(NioServerSocketChannel.class)
        .handler(new SimpleServerHandler())
  			// 设置 I/O Channel关联的Pipeline中的 Handler
        .childHandler(new ChannelInitializer<SocketChannel>() {
            @Override
            public void initChannel(SocketChannel ch) throws Exception {
            }
        });

// 客户端
Bootstrap bootstrap = new Bootstrap();
bootstrap.group(group)
        .channel(NioSocketChannel.class)
        .option(ChannelOption.TCP_NODELAY, true)
  			// 设置客户端 Channel关联的Pipeline中的 Handler
        .handler(new ChannelInitializer<NioSocketChannel>() {
            protected void initChannel(NioSocketChannel ch) throws Exception {
                ch.pipeline().addLast(new ChildHandler());
            }
        });
```



在服务器端和客户端经常会用到的这个类是一个ChannelHandler

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/ChannelInitializer.png" alt="ChannelInitializer" style="zoom:40%;float:left" />

那它肯定在一个时机会被添加到Pipeline当中，重要的 ```initChannel``` 方法实际上是发生了两种事件的时候会被调用，一个是 ```handlerAdded``` ，另一个是 ```channelRegistered``` ，但是

```java
private boolean initChannel(ChannelHandlerContext ctx) throws Exception {
    if (initMap.putIfAbsent(ctx, Boolean.TRUE) == null) { // Guard against re-entrance.
        try {
          	// 这里调用的方法就是我们经常重写的 initChannel 方法
            initChannel((C) ctx.channel());
        } catch (Throwable cause) {
            // Explicitly call exceptionCaught(...) as we removed the handler before calling initChannel(...).
            // We do so to prevent multiple calls to initChannel(...).
            exceptionCaught(ctx, cause);
        } finally {
          	// 当前这个ChannelHandler在init之后就从Pipeline中移除了
            remove(ctx);
        }
        return true;
    }
    return false;
}


// 触发initChannel的事件
// handlerAdded：当一个ChannelHandler被成功添加到Pipeline，准备好处理事件时
@Override
public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
    if (ctx.channel().isRegistered()) {
        // This should always be true with our current DefaultChannelPipeline implementation.
        // The good thing about calling initChannel(...) in handlerAdded(...) is that there will be no ordering
        // surprises if a ChannelInitializer will add another ChannelInitializer. This is as all handlers
        // will be added in the expected order.
        initChannel(ctx);
    }
}

// 触发initChannel的事件
public final void channelRegistered(ChannelHandlerContext ctx) throws Exception {
    // Normally this method will never be called as handlerAdded(...) should call initChannel(...) and remove
    // the handler.
    if (initChannel(ctx)) {
        // we called initChannel(...) so we need to call now pipeline.fireChannelRegistered() to ensure we not
        // miss an event.
        ctx.pipeline().fireChannelRegistered();
    } else {
        // Called initChannel(...) before which is the expected behavior, so just forward the event.
        ctx.fireChannelRegistered();
    }
}
```









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

