# Java基础



## NIO



![image-20200321120909713](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200321120909713.png)





如果一个SocketChannel调用 register 方法注册到Selector上面会返回一个SelectionKey，这个SelectionKey注册在Selector上面，如果调用 cancel 方法取消注册时，SelectionKey并不会被移除，会被添加到Selector的cancelled-key set，等到下次运行 Selector.select 方法的时候才会清除这些被取消的SelectionKey



参考：

https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SelectionKey.html









## 方法重载和方法重写

**重载**：①方法名相同，②参数不相同

**重写**：①方法名相同，②参数相同，③返回值类型 <= 父类返回值类型，④抛出异常类型 <= 父类抛出异常类型



参考：

https://www.cnblogs.com/myworld7/p/10398335.html





### HashMap

* 和HashTable基本相同，但是支持null key 和 null value
* 元素无序
* 集合遍历时，需要扫描整个HashMap中的数组长度加上每个数组bucket链接的元素，所以HashMap的capacity越大，遍历所需要的时间就越长，HashMap不知道数据bucket的哪些bucket是存储了元素的，哪些是没有的，所以需要遍历capacity长度的数组。
* 两个影响性能的关键因素：initial capacity 和 load factor，load factor是触发rehash的因素，rehash导致capacity增加一倍，**默认0.75，平衡性能和内存空间占用**。
* HashMap是非线程安全的，要么把HashMap包裹在一个类里面，然后类里面的方法用 ```synchronized ```修饰，要么就把一个HashMap在创建时就变成线程安全的 ```Map m = Collections.synchronizedMap(new HashMap(...));```
* HashMap有 ```fail-fast``` 特性







###Serializable

* Serializable 接口只是用来语义上表明一个类是可序列化的
* 可序列化类的所有子类型本身都是可序列化的
* 可序列化类有一个版本号，叫做 serialVersionUID，序列化运行时为每个可序列化类关联一个版本号，称为serialVersionUID，在反序列化过程中使用该版本号来 **验证序列化对象的发送方和接收方是否已为该对象加载了与序列化兼容的类**。如果反序列化的时候，接收方加载的类的serialVersionUID和发送方的不一样，就会报 ```InvalidClassException``` 反序列化错误。类可以自己声明 serialVersionUID 值，但必须是 static , final , long 类型的值。如果没有声明serialVersionUID值的话，虚拟机在序列化的时候会自动生成一个，但是强烈剪子自己设置，因为自动生成可能出现发送方和接收方同一个类的serialVersionUID值不相同的情况，导致反序列化失败。
* **Serializable使用场景**：需要将某个对象保存到磁盘上或者通过网络传输，那么这个类应该实现Serializable接口或者Externalizable接口之一。
* **transient** 关键字修饰类的属性，表示该属性不需要序列化，使用transient修饰的属性，java序列化时，会忽略掉此字段，所以反序列化出的对象，被transient修饰的属性是默认值。对于引用类型，值是null；基本类型，值是0；boolean类型，值是false。
* 自定义序列化：https://juejin.im/post/5ce3cdc8e51d45777b1a3cdf#h



### Cloneable

* 不要使用这个接口，没有任何意义，

* Object的clone()方法返回的是对象的浅拷贝

* copy constructor，一种对象copy策略

  ```java
  class DummyBean {
    private String dummy;
  
    public DummyBean(DummyBean another) {
      this.dummy = another.dummy; // you can access  
    }
  }
  ```

* 可以使用：

  ```java
  import org.apache.commons.lang.SerializationUtils;
  
  // 深拷贝
  SerializationUtils.clone(Object);
  ```












> 什么情况下使用 1 << 4 来表示16 ？

1 <<4 的值和16相同，但是 1<<4 还有另外一个形式上的含义，就是这个数是2的倍数，形式上隐含的意思这是16无法表示的，就像：

```java
int day = 86400;
// vs
int day = 60 * 60 * 24; // 86400
```









### new MyClass()

在任何一个地方new一个对象，就相当于执行了以下的字节码：

```java
new           #6                  // class com/pingan/fimax/plugin/sdk/PrimaryTypeTest
dup
invokespecial #7                  // Method "<init>":()V
```

解释一下三个JVM的指令：

* **new**：后面加常量池中某个值的索引，这个索引位置的值表示的就是一个类的描述信息，这样根据类的描述信息对类进行加载，然后实例化，在堆区域创建一个类的对象，**并把该对象的引用push到当前的操作数栈**。
* **dup**：把当前操作数栈的最上面的值拷贝一份push到当前操作数栈。
* **invokespecial**：调用实例的方法，后面跟一个常量池的索引值，这个索引位置的值就是一个方法的描述信息，那么究竟调用哪个实例的这个方法呢，就是操作数栈pop出来的值，也就是刚刚dup出来的对象的引用。

执行完上面3条指令之后，此时的操作数栈还保留着对象的引用。



