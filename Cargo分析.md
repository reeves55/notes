#Cargo分析

## requestContext

每调用一次Cargo，也就是调用Cargo的Invoke方法，都会创建一个新的requestContext



![image-20190925000029101](/Users/xiao/Library/Application Support/typora-user-images/image-20190925000029101.png)



可以看到这个requestContext包括三个部分，一个是请求上下文，请求上下文当中可以放一些整个请求需要用到的key-value键值对，stub就是chaincode执行的stub，也是cargo调用其他chaincode需要的一个stub，currentChannel就是当前的channel，是一个字符串。



![image-20190925000237116](/Users/xiao/Library/Application Support/typora-user-images/image-20190925000237116.png)



![image-20190925000313319](/Users/xiao/Library/Application Support/typora-user-images/image-20190925000313319.png)



Context当中有一个很重要的方法，就是Value



![image-20190925000521382](/Users/xiao/Library/Application Support/typora-user-images/image-20190925000521382.png)



这个方法可以从Context上下文对象当中读取上下文对象保存的key-value值。

创建了上下文对象之后，就可以把请求转发到调用其他的chaincode了，比如说 cabbage、citrus等等：



![image-20190925000754649](/Users/xiao/Library/Application Support/typora-user-images/image-20190925000754649.png)



话说这个routers里面究竟存储了什么呢，这就和chaincode和cargo的整合方式有关了，小明重写后的chaincode都有一个module目录，module里面都有一个init方法，在这个方法里面调用cargo实例的RegisterRouter方法来注册路由设置函数：



![image-20190925001616960](/Users/xiao/Library/Application Support/typora-user-images/image-20190925001616960.png)



就拿这个read.Configure来说吧



![image-20190925001714845](/Users/xiao/Library/Application Support/typora-user-images/image-20190925001714845.png)



cargo的RegisterRouter方法会把chaincode的路由设置函数都放到一个routeGroup里面



![image-20190925001834451](/Users/xiao/Library/Application Support/typora-user-images/image-20190925001834451.png)



然后cargo启动的时候，会执行routeGroup里面所有的路由设置函数



![image-20190925001937613](/Users/xiao/Library/Application Support/typora-user-images/image-20190925001937613.png)



执行路由注册函数



![image-20190925002015636](/Users/xiao/Library/Application Support/typora-user-images/image-20190925002015636.png)



router的Register方法则会把方法名映射到具体的方法处理函数，这就是dispatch时候，根据请求参数中的方法名找到相应的处理函数的根据：



![image-20190925002204879](/Users/xiao/Library/Application Support/typora-user-images/image-20190925002204879.png)



所以，最终的路由信息其实都存储在cargo的routers Map当中的