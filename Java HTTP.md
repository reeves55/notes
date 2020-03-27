# Java HTTP

一个HTTP请求由4部分主体构成：

![](/Users/xiao/Downloads/http-message.jpg)



**请求行** *<回车符><换行符>*

**请求头字段名 : 请求头字段值** *<回车符><换行符>*

**...**

*<回车符><换行符>*

**请求体**



HTTP是超文本传输协议，传输的内容是字符文本格式。







## 不同Content-Type和请求体格式对应关系

> application/x-www-form-urlencoded
>



使用场景：表单提交参数

请求体格式：param1=value1&param2=value2&...

示例：name=lizhongcheng&age=24

Spring处理方式：解析http request body，这个时候request body实际上是byte数组，spring会解析byte数组，根据 &, = 等符号来解析参数的名字和值。还需要注意的是，此时的request body实际上是经过url encode的，所以解析之前首先是需要进行url decode。



> multipart/form-data











默认的http request body的编码是：

```java
private static final Charset DEFAULT_BODY_CHARSET = StandardCharsets.ISO_8859_1;
```

但是在处理请求的时候会从请求头里面的content-type里面尝试获取charset，如果有的话，就设置成请求当中指定的编码格式。



###注意

Java的Http server会操作操作系统级别的socket API，得到byte[]，这个数据就是应用层的数据，网络分层协议决定了传输层不关心应用层数据的格式，TCP只把应用层的数据当成字节流进行传输，操作系统的socket是对传输层协议的一层封装，不会对应用层数据进行解析，所以，Java http server需要自己处理字节数组格式的http应用层数据，即，按照http协议格式解析字节数组。HTTP1.1协议规定，请求行以及请求头中的属性都是使用US-ASCII码编码，而请求头的属性值和请求体的编码不是固定的。应用层可以直接使用US-ASCII编码规范来解析http字节流得到请求行和请求头，至于请求体，就要看http请求头当中有没有指定请求体的内容编码格式了。