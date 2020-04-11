

# Tomcat工作原理



Tomcat两大核心功能：

1. 接收客户端请求；
2. 将客户端请求封装后交给Servlet处理（Servlet容器规范）；



![img](https://images2015.cnblogs.com/blog/665375/201601/665375-20160119184923890-1995839223.png)



几大核心组件：

* Server：



## Tomcat如何启动的？









## Tomcat如何处理请求的？

tomcat基于connector组件来接收客户端的请求，一个connector对象和一个端口绑定，同时也绑定一个应用层协议比如HTTP/1.1，connector监听这个端口上所有的请求，connector采用多线程方式处理请求，一个连接请求一个线程进行处理，最多创建 ```maxThreads```个活动线程，请求继续增多时，把请求放到请求队列里去，最多放```acceptAcount```个请求，再继续多请求，将返回 ```connection refused```。



> connector组件的作用：用于接收请求并将请求封装成Request 和Response后交给Container来处理





1. **准备阶段** - 解析server.xml，创建Connector对象

   解析tomcat自身配置文件server.xml，遇到<Connector>节点时，创建Connector对象，协议类型作为构造参数，一个典型的<Connector>节点如下：

   ```xml
   <Connector port="8080" protocol="HTTP/1.1"
                  connectionTimeout="20000"
                  redirectPort="8443" />
   ```

   在创建Connector对象过程中，需要注意，Connector会根据协议类型来接着创建一个ProtocolHandler对象，并赋值给自身的一个成员变量，处理HTTP/1.1协议的protocolHandler是 ```org.apache.coyote.http11.Http11NioProtocol```类

   ![image-20200212094326106](/Users/xiao/Library/Application Support/typora-user-images/image-20200212094326106.png)

   

   Http11NioProtocol对象的创建过程如下：

   ```java
   AbstractProtocol(AbstractEndpoint<S,?> endpoint){ this.endpoint = endpoint;...}
     |
   AbstractHttp11Protocol(AbstractEndpoint<S,?> endpoint)
     |
   AbstractHttp11JsseProtocol(AbstractJsseEndpoint<S,?> endpoint)
     |
   Http11NioProtocol(){ super(new NioEndpoint());}
   ```

   

   ![image-20200212110831898](/Users/xiao/Library/Application Support/typora-user-images/image-20200212110831898.png)

   

   如图，Connector内部也是有一定复杂度的结构的，实际上接收客户端请求的是Endpoint类，





2. **启动阶段** - start Connector，接收并处理连接请求

   tomcat准备完成，开始启动的时候，启动链是这样子的：

   ```java
   // tomcat启动链
   Bootstrap.main()
     |
   Bootstrap.start()
     |
   Catalina.start()
     |
   StandardServer.startInternal()
     |
   StandardService.startInternal() // 多个service循环start
     |
   StandardEngine.startInternal() && StandardThreadExecutor.startInternal() && MapperListener.start() && Connector.startInternal() // 多个Executor和多个Connector循环start
     
   // 对于Connector.startInternal()启动链：
   Connector.startInternal()
     |
   ProtocolHandler.start()
     |
   AbstractEndpoint.start()
     |
   AbstractEndpoint.bind() && AbstractEndpoint.startInternal()
   ```

   一个connector内部是什么样的呢？

   

   ![image-20200213135652912](/Users/xiao/Library/Application Support/typora-user-images/image-20200213135652912.png)

   







参考：

https://docs.oracle.com/javase/7/docs/api/java/nio/channels/Selector.html

http://tutorials.jenkov.com/java-nio/buffers.html







tomcat有三种不同类型的connector：

* BIO connector
* NIO connector
* APR/native connector





