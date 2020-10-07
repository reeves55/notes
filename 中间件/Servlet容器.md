# Servlet容器





## Tomcat

Tomcat既是一个http服务器，也是一个实现Servlet规范的Servlet容器，可以提供静态资源服务，也可以通过运行特定的Servlet来提供动态内容服务。



### 两大组件

一个复杂系统如何构建？答案就是将系统进行拆分，拆分成不同的组件，每个组件完成特定部分的功能，再把这些组件串联起来，实现系统提供的功能。拆分后，组件之间要尽可能解耦，这样我们的系统才能有一定灵活性，不会牵一发而动全身。

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200719100150632.png" alt="image-20200719100150632" style="zoom:40%;" />





#### 连接器 - socket

连接器要处理的问题有：

1. 监听网络端口，接受连接请求；
2. 读取请求的字节流数据；
3. 按照应用层协议将字节流解析成应用层协议内容，并抽象成Tomcat Request对象；
4. 将Tomcat Request对象转换成容器中使用的ServletRequest对象，并传递给容器；
5. ...
6. 将响应写入socket；





<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200719100906358.png" alt="image-20200719100906358" style="zoom:45%;" />











#### 容器 - servlet







<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200719095852874.png" alt="image-20200719095852874" style="zoom:50%;" />











<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200719100219247.png" alt="image-20200719100219247" style="zoom:50%;" />