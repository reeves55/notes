# Java基础



## 面向对象编程思想



Java是一个面向对象的编程语言，当然就具备面向对象的几大特性：抽象、封装、继承、多态。面向对象是一种程序设计思想，是为了能够构建出大型的复杂的应用程序，且让程序具有可拓展性、可维护性和代码复用，当然，程序的可拓展性和可维护性是需要良好的程序设计写法的，不是用了面向对象思想就能写出这种好的程序。

面向对象把用来解决问题的算法分解，从算法中抽象出多个组件（对象），每个对象所解决的问题或者包含的功能都是不一样的，通过这些对象之间的交互，完成算法要实现的功能。概况起来，就是：

1. 抽象出问题域中的组件
2. 设计主流程



所以第一个问题是如何从算法中抽象出来需要哪些对象/组件❓





什么是主流程❓

程序的入口，算法的开始，就是执行某个对象的某个功能，这个功能可以理解为 start，暂且把这个功能叫做入口功能吧，包含这个入口功能的对象叫做入口对象， ，这个入口功能会用到其他对象，于是，对象之间的协作就开始了。就像SpringBoot程序，XXXApplication就是一个入口对象。



> 如何实现代码的复用❓

代码复用无非就是利用已知对象（组件）来构建新的组件，复用的方式有多种，继承可以，组合也可以，建议多用组合，少用继承



> 如何解决对象（组件）之间的耦合性和程序功能可变性之间的矛盾❓

这就是多态的用武之地了，实现程序多态性可以利用继承和接口，把变化的部分掩盖，提供不变的部分给使用者，不改变使用者的代码，却可以任意更换被使用者的类型。



## 类、接口、对象

类就是算法中我们分割，抽象出来的功能组件，封装最好的体现就是它。它包含状态以及某些功能，具体来说就是类成员变量和类方法。

抽象类









## 数据类型

所以现在在程序设计语言的角度上来看看Java的数据类型，程序设计语言使我们和计算机进行沟通的桥梁和工具，我们把我们的思考逻辑变成代码，然后代码变成计算机指令。通过计算机程序这个中介，我们得以让计算机按照我们的思想进行计算，并得出我们想要的结果。

数据类型是什么呢，无非就是把我们人类世界用到的数据分分类，人类世界用的不叫数据，叫信息，信息在计算机世界里翻译过来就是数据，信息有种类，数据也就有了种类，每种类型的数据都有其特点，程序设计语言给我们定义了数据类型的概念用来方便我们选择我们想要用哪种类型的数据

信息和数据之间可以相互转换，但有一个前提是必需的，那就是数据一定要有规则。没有规则的数据不能转换成信息，也就不包含任何价值。计算机上存储的都叫数据，但这些数据是否都有意义，则取决于我们有没有“翻译”这些数据的方法。举个🌰，一段密文，在没有解密方法之前，密文只是数据，它没法转换成人类可以理解的，所以它不是信息，但是一旦有了解密的方法，看似乱七八糟的密文立马就能变成人们可以理解的信息。所以重点在于：数据的解析方法，也可以当成数据的“翻译”方法。

以这个角度来看数据类型，程序设计语言定义的种种类型，实际上也就是定义了一种种解析数据的规则，int类型，在把数值写入到内存时，会把10进制的值先“翻译”二进制的01串再存储进去，在读取的时候，会把内存当中的01串数据当成一个10进制数的二进制表示形式，所以会按照2进制到10进制的转换方式把01串“翻译”成我们能理解的数字。char类型，会把2个字节16位的01串当成Unicode编码当中的数字坐标，然后通过查找Unicode数字坐标和字符的对应关系，给我们呈现可以认识的字符，而不是01二进制串。

很容易理解的是，这种转换规则是我们可以自己定义的，所以才有了像protoBuffer，它定义了信息如何转换成二进制01串，也同样定义了01串如何转换成特定类型的信息。数据类型也没有什么神秘的，它就是语言设计者定义的一些规则而已。

有了数据类型这种理解就好说明什么是强类型语言，什么是弱类型语言了，强类型语言意味着你在程序当中一个变量必须声明为一种特定的数据类型，之后这个变量的值就只能存储这一种类型的值了，如果你对这个变量做的操作超过了变量所属数据类型允许的操作范围之外，那么程序在编译期间就会报错，这个专业点叫类型安全，这样你就不可能在本来存储int整型数据的内存空间上存储float浮点数，因为变量已经定义成int类型，那么对内存当中的01串的理解方式都是按照整型的规则来的，而浮点数在内存当中01串的表示规则和整型是完全不一样的，强行把它解释成整型只能得到完全没有意义的值，所以强类型语言是绝对不允许这样做的；而弱类型语言不一样，弱类型语言基本指的是脚本语言，它没有数据类型这么一说，你想定义什么数据就定义什么数据，一个var关键字就搞定，var a = 1; var a = {key:"value"};var a = 1.5...，这都可以，你可以对一个变量赋予不同数据类型的值，比如var a=1; a=3.5; 这并不会引起错误，它的规则是什么样的呢，就是变量它只代表一片内存空间的地址，他不管你要往这片内存空间里面塞什么，加入你塞进去的是整型int，那么这片内存区域存储的就是数字的二进制01串，如果你往里面塞的是一个浮点数float，那么这片内存区域存储的就变成了浮点数在内存当中表示的01串。当你要用这个变量的时候，这片内存区域当中的01串该按照什么规则去理解呢？是不是问题就来了，既然动态语言不管你把变量赋值成什么类型，那么我怎么知道我究竟赋值了什么类型呢？如果不知道赋值了什么数据类型，那岂不是自己把自己给整懵了嘛，哎，这个问题可以用javascript来举🌰，js 在底层存储变量的时候，会在变量的机器码的低位1-3位存储其类型信息👉

- 000：对象
- 010：浮点数
- 100：字符串
- 110：布尔
- 1：整数

这就是动态语言知道当前变量所对应的内存空间里究竟存储着什么类型数据的秘密。



参考：

https://juejin.im/post/5b0b9b9051882515773ae714

https://lenciel.com/2016/09/types-in-programming-languages/





### 高级数学计算

#### 1. 取整



#### 2. 取余



#### 3. 次方（x^y）



#### 4. 平方根立方根



#### 5. 三角函数



#### 6. 随机数







## 集合 & Map





#### HashMap



HashMap中table的延迟创建，“懒加载”

HashMap的所有构造函数（除了用一个已有Map当做参数的构造函数外）就只做了设置loadFactor和threshold（initialCapacity * loadFactor）的值，没有对核心的Node数组进行new的操作，那什么时候进行new操作呢？在resize操作的时候：

```java
        if (oldCap > 0) {
          // 此时，table不为空，已经有数据
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
				// 此时
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            // 此时， table为空，且 threshold==0，使用默认的 initial_capacity和默认的load_factor
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }

				...
        
        
```



**resize操作**

* 什么时候触发resize❓

  1. 核心数据结构table==null 或者 table.length==0

  2. 每次向HashMap进行put操作之后，都会检测table中的元素个数是否超过阈值

     ```java
     float DEFAULT_LOAD_FACTOR = 0.75f;
     
     // threshold = capacity * load factor
     if (++size > threshold)
        resize();
     ```

     

* resize时候究竟做了什么❓

  要点1：对于old table数组中的每一行（数组元素+数组元素作为头结点的链表或树），在resize之后，节点的位置有两种可能，假设这一行在数组位置为 index：1. 节点的hash <= old capacity，这种节点在resize之后还是属于这一行；2. 节点的hash > old capacity，这种节点resize之后，就换了行位置了，就放在 old capacity + index 这个位置





```java
      final Node<K,V>[] resize() {
          Node<K,V>[] oldTab = table;
          int oldCap = (oldTab == null) ? 0 : oldTab.length;
          int oldThr = threshold;
          int newCap, newThr = 0;
          if (oldCap > 0) {
              if (oldCap >= MAXIMUM_CAPACITY) {
                  threshold = Integer.MAX_VALUE;
                  return oldTab;
              }
              else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                       oldCap >= DEFAULT_INITIAL_CAPACITY)
                  newThr = oldThr << 1; // double threshold
          }
          else if (oldThr > 0) // initial capacity was placed in threshold
              newCap = oldThr;
          else {               // zero initial threshold signifies using defaults
              newCap = DEFAULT_INITIAL_CAPACITY;
              newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
          }
          if (newThr == 0) {
              float ft = (float)newCap * loadFactor;
              newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                        (int)ft : Integer.MAX_VALUE);
          }
          threshold = newThr;
          @SuppressWarnings({"rawtypes","unchecked"})
              Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
          table = newTab;
          if (oldTab != null) {
              for (int j = 0; j < oldCap; ++j) {
                  Node<K,V> e;
                  if ((e = oldTab[j]) != null) {
                      oldTab[j] = null;
                      if (e.next == null)
                          newTab[e.hash & (newCap - 1)] = e;
                      else if (e instanceof TreeNode)
                          ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                      else { // preserve order
                          Node<K,V> loHead = null, loTail = null;
                          Node<K,V> hiHead = null, hiTail = null;
                          Node<K,V> next;
                          do {
                              next = e.next;
                              if ((e.hash & oldCap) == 0) {
                                  if (loTail == null)
                                      loHead = e;
                                  else
                                      loTail.next = e;
                                  loTail = e;
                              }
                              else {
                                  if (hiTail == null)
                                      hiHead = e;
                                  else
                                      hiTail.next = e;
                                  hiTail = e;
                              }
                          } while ((e = next) != null);
                          if (loTail != null) {
                              loTail.next = null;
                              newTab[j] = loHead;
                          }
                          if (hiTail != null) {
                              hiTail.next = null;
                              newTab[j + oldCap] = hiHead;
                          }
                      }
                  }
              }
          }
          return newTab;
      }
```











参考：

https://tech.meituan.com/2016/06/24/java-hashmap.html

















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





## 并发编程

多线程的运行时必要开销来自于线程上下文切换，线程上下文切换会造成两种开销：

1. CPU需要花费时间来保存和恢复线程上下文信息；
2. 缓存失效

当多个线程间存在访问共享内存的情况时，需要使用一定的同步机制来避免并发问题，同步机制会导致：

1. 抑制编译器优化代码；
2. 刷新或者使CPU缓存失效；
3. 在共享内存总线上增加同步流量



对于一个对象来说，线程安全包括两个方面：

1. 自己所在线程访问一个其他线程也可能访问的共享变量时，协调好访问顺序；
2. 多个线程同时来访问自己的状态变量时，保证自己状态的正确性；



**线程安全的类**：在多线程并发访问类的时候，类仍然能够保持状态的正确性，而不需要调用者对访问其属性的操作进行同步保证，则这个类是线程安全的，这个时候类本身封装了内部变量访问的同步机制。



数据操作不一定会引起线程并发问题，<span style="color:red">只有对共享数据的非原子性操作会引发多线程并发问题</span>。

如何让内存中的数据在多线程环境当中保持正确性，关键点在于对线程进行同步，



```java
package java.util.concurrent.atomic;
```

并发数据类型，这些数据类型不仅封装了某种类型的值，还封装了这些值 <span style="color:red">**在多线程环境中的同步操作**</span>，atomic包下的数据类型，<span style="color:red">基于CPU的CAS原子指令，实现同步操作</span>。

* ```AtomicInteger```：





### 线程

线程是操作系统的概念，而不是CPU中的概念，CPU只负责执行指令，操作系统负责给CPU分配具体执行的线程。即：

* 线程在操作系统层次上创建
* CPU时间片由操作系统切分，一个时间片之后，操作系统进行线程调度



如果系统当中运行的线程数和CPU核数相等，理论上没有一个线程需要进行切换，都能在一个CPU核上一直运行。





#### Linux线程是什么样的？





posix thread lib in linux：

https://www.cs.cmu.edu/afs/cs/academic/class/15492-f07/www/pthreads.html





Linux线程池实现思路：

一个任务队列，每个任务指明了任务需要执行的函数和参数

一堆线程，每个线程都死循环尝试从任务队列中取出任务，然后执行任务中指定的函数，并传递给函数指定的参数





Java一个线程是如何重复利用的







#### TreadPoolExecutor





```java
// ctl 状态控制变量，表示两个值
// 1. workerCount: 已经启动并且不会停止的线程数量
// 2. runState: 线程池当前所处的生命周期状态
```





负数使用相应正数经过反码运算之后表示

```java
// -127的表示
// 相应正数为 01111111
// 1. 取反
// 1000000
// 2. 加1
// 10000001 <- 这就是-127实际存储的值

// 负数解码
// 1. 减1
// 10000000
// 2. 取反
// 01111111 <- 这是负数的正数值，没有符号位，就是127，因此此负数表示的是-127
```





#### Spring RequestContextHolder

用来保存当前的HTTP请求信息的，是对HttpServletRequest的一个包装，关键方法有：

```java
// 线程安全的 requestAttributesHolder
// 每个线程都有一个 RequestAttributes对象
private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
            new NamedThreadLocal<RequestAttributes>("Request attributes");

// 获取当前请求相关信息  你   s'd
public static RequestAttributes getRequestAttributes() {
    RequestAttributes attributes = requestAttributesHolder.get();
    if (attributes == null) {
        attributes = inheritableRequestAttributesHolder.get();
    }
    return attributes;
}

public static void setRequestAttributes(@Nullable
                                        RequestAttributes attributes,
                                        boolean inheritable);

public static RequestAttributes currentRequestAttributes()
                                                  throws IllegalStateException;

public static void resetRequestAttributes();
```



RequestAtrributes是一个接口，接口方法有：

| 返回值     | 方法描述                                                     |
| ---------- | ------------------------------------------------------------ |
| `Object`   | `getAttribute(String name, int scope)`Return the value for the scoped attribute of the given name, if any. |
| `String[]` | `getAttributeNames(int scope)`Retrieve the names of all attributes in the scope. |
| `String`   | `getSessionId()`Return an id for the current underlying session. |
| `Object`   | `getSessionMutex()`Expose the best available mutex for the underlying session: that is, an object to synchronize on for the underlying session. |
| `void`     | `registerDestructionCallback(String name, Runnable callback, int scope)`Register a callback to be executed on destruction of the specified attribute in the given scope. |
| `void`     | `removeAttribute(String name, int scope)`Remove the scoped attribute of the given name, if it exists. |
| `Object`   | `resolveReference(String key)`Resolve the contextual reference for the given key, if any. |
| `void`     | `setAttribute(String name, Object value, int scope)`Set the value for the scoped attribute of the given name, replacing an existing value (if any). |



**Spring框架在DispatchServlet处理一个HTTP请求的逻辑中，执行具体请求处理方法前，会调用RequestContextHolder中的setRequestAttributes()方法：**

```java
// DispatchServlet从父类FrameServlet中继承的方法
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        long startTime = System.currentTimeMillis();
        Throwable failureCause = null;

        LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
        LocaleContext localeContext = buildLocaleContext(request);

        RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
        ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
        asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

        //关键方法：将RequestAttributes设置到RequestContextHolder
        initContextHolders(request, localeContext, requestAttributes);

        try {
            //具体的业务逻辑
            doService(request, response);
        }
        catch (ServletException ex) {
            failureCause = ex;
            throw ex;
        }
        catch (IOException ex) {
            failureCause = ex;
            throw ex;
        }
        catch (Throwable ex) {
            failureCause = ex;
            throw new NestedServletException("Request processing failed", ex);
        }

        finally {
            //重置RequestContextHolder之前设置RequestAttributes
            resetContextHolders(request, previousLocaleContext, previousAttributes);
            if (requestAttributes != null) {
                requestAttributes.requestCompleted();
            }

            if (logger.isDebugEnabled()) {
                if (failureCause != null) {
                    this.logger.debug("Could not complete request", failureCause);
                }
                else {
                    if (asyncManager.isConcurrentHandlingStarted()) {
                        logger.debug("Leaving response open for concurrent processing");
                    }
                    else {
                        this.logger.debug("Successfully completed request");
                    }
                }
            }

            publishRequestHandledEvent(request, response, startTime, failureCause);
        }
    }


// 在这里设置RequestContextHolder
    private void initContextHolders(
            HttpServletRequest request, LocaleContext localeContext, RequestAttributes requestAttributes) {

        if (localeContext != null) {
            LocaleContextHolder.setLocaleContext(localeContext, this.threadContextInheritable);
        }
        if (requestAttributes != null) {
            RequestContextHolder.setRequestAttributes(requestAttributes, this.threadContextInheritable);
        }
        if (logger.isTraceEnabled()) {
            logger.trace("Bound request context to thread: " + request);
        }
    }
```



参考：

https://www.jianshu.com/p/80165b7743cf





#### ThreadLocal的原理

ThreadLocal相当于一个容器，只不过只能放进去一个值而已，关键是这个值是每个线程都会持有一份的，ThreadLocal变量放在哪里都无所谓，重要的是它的get()方法和set()方法，get()方法在当前线程上下文中获取一个值，set()方法在当前线程上下文中设置一个值。



| 知识面       | 知识点                                                  | 描述                                                         |
| ------------ | ------------------------------------------------------- | ------------------------------------------------------------ |
| JVM 垃圾回收 | 四种```Reference```类型，强引用，软引用，弱引用，虚引用 | 软引用和弱引用虽然一般都作为缓存场景中，但是软引用的缓存是在内存实在不够用的时候，回收掉，回收之前，都是一直有用的，而弱引用就不一样了，这种缓存其实可能只用几次就不用了不会一直用，所以及时回收才是正道 |



#### Object notify wait & synchronized

基于一个对象的锁，锁是什么概念，锁可以理解为打开新世界的钥匙，是一个令牌，只有拿到这个锁，才能做某些事情，所以我们用 ```synchronized(obj)```，```ReentrantLock lock = new ReentrantLock(); lock.lock();```，锁的使用必然和某个实体直接相关，那个实体就是锁，拥有实体的线程才能执行某一部分的代码，而实体仅有一份，两者结合，保证了线程同步执行。



Object的 ```notify ``` 和 ```wait``` 系列方法是用来和 ```synchronized``` 关键字配合使用，实现使用同一个资源的并发线程可以进行通知的操作。配合使用的方法就是 synchronized 锁住的对象和用来通知的对象是同一个对象

```java
Object obj = new Object();

// thread 1 后执行
Thread.sleep(1000);

synchronized(obj) {
  System.out.println("thread 1 get monitor and notify thread2");
	obj.notifyAll();
  System.out.println("leave thread 1");
}

// thread 2 先执行
synchronized(obj) {
  System.out.println("thread 2 will wait and handle out monitor");
  obj.wait();
  System.out.println("thread 2 continue to do something");
}

// 输出
// thread 2 will wait and handle out monitor
// thread 1 get monitor and notify thread2
// leave thread 1
// thread 2 continue to do something
```



#### synchronized锁状态切换

JDK1.6之前，synchronized的性能远不及ReentrantLock，那是因为synchronized只有重量级锁，JDK1.6，synchronized做了大量的锁优化，性能已经和ReentrantLock持平。



**偏向锁**：偏向锁是为了一个同步代码块一直只有一个线程在访问的情形，不存在竞争。好处是这个时候线程访问同步代码块只需要判断锁的对象头的偏向锁指定的线程ID是否是当前线程即可，不需要其他复杂操作，效率极高（偏向锁如何从偏向一个线程切换成偏向另一个线程呢？）



**轻量级锁**：轻量级锁是为了一个同步代码块存在线程竞争情况，但是竞争不激烈，或者交替访问不竞争时的情形。



**重量级锁**：重量级锁是为了一个同步代码块多个线程激烈竞争的情况，许多线程抢占Monitor，线程同步访问代码块。

**自旋锁**：



**锁消除**：



**锁粗化**：







双重检查锁的单例模式，属于懒加载









####字符串为什么在Java中设计成不可变，如何实现不可变的？

**为什么这么设计**：<span style="color:red">1）</span>程序当中大量使用字符串，如果字符串能够缓存起来，必定能提高程序运行效率，但是如果是所有缓存的字符串池，多个变量的值相同时，那么它们肯定指向字符串池当中同一个对象，这个时候如果字符串是可变的话，一个程序改变了字符串的值，其他的变量都受到了影响，所以字符串得是不可变的才行。<span style="color:red">2）</span>安全问题，Java使用字符串来表示类的权限定名，也是通过类的名字来加载类的，如果字符串是可变的，就可能产生原先要加载类```java.io.Writer``` 变成了加载 ```mil.vogoon.DiskErasingWriter```； <span style="color:red">3）</span>String不可变，那么字符串对象就可以缓存起hashcode的值，这样String作为HashMap的Key时，就不需要先计算对象的hashcode了，效率更高。



**如何实现**：<span style="color:red">1）</span>String类是final的，不可继承，用户无法改变String特点；<span style="color:red">2）</span>实际存储字符串的char[]是private的，char变量是final类型的，无法改变char[]中的任何一个字符的值。











### 线程池



1. 为什么需要线程池❓

   关键点：```线程资源复用```，```阻塞队列缓冲任务```，

   对于<span style="color:red">运行时间短 </span>且 <span style="color:red">频繁运行</span>的任务，适合使用线程池，可以达到很好的线程复用的效果，资源利用率更高；

   



#### ThreadPoolExecutor

常用构造方法为：

```java
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler)
```

1. ```corePoolSize```： 线程池维护线程的最少数量
2. ```maximumPoolSize```：线程池维护线程的最大数量
3. ```keepAliveTime```： 线程池维护线程所允许的空闲时间
4. ```unit```： 线程池维护线程所允许的空闲时间的单位
5. ```workQueue```： <span style="color:red">线程池所使用的缓冲队列</span>
6. ```threadFactory```：创建新线程的工厂
7. ```handler```： <span style="color:red">线程池对拒绝任务的处理策略</span>



handler的选择：

* ThreadPoolExecutor.AbortPolicy() ：抛出java.util.concurrent.RejectedExecutionException异常

* ThreadPoolExecutor.CallerRunsPolicy() : 重试添加当前的任务，他会自动重复调用execute()方法

* ThreadPoolExecutor.DiscardOldestPolicy() : 抛弃旧的任务
* ThreadPoolExecutor.DiscardPolicy() : 抛弃当前的任务





##### FutureTask

1. ```public void execute(Runnable command);```
2. ```Future<?> submit(Runnable task);```
3. ```public <T> Future<T> submit(Runnable task, T result);```
4. ```public <T> Future<T> submit(Callable<T> task);```



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200324221157738.png" alt="image-20200324221157738" style="zoom:55%;float:right;margin-right:40px" />

3个返回Future的方法实际上返回的都是 ```FutureTask``` 对象，既然FutureTask是线程池返回的，这个对象在线程池运行任务过程中，肯定会引用该对象，在主线程也是通过FutureTask来获取执行结果，就意味着这个对象至少被两个线程共享，线程池在任务执行结束肯定要唤醒等待在这个对象上的主线程，让主线程能够获取到结果返回继续运行，提交任务的时候是同步非阻塞的，在获取任务执行结果的时候是同步阻塞的。



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200324215900501.png" alt="image-20200324215900501" style="zoom:50%;" />









线程：

可以通过 ```setDamon(boolean on)``` ，把一个线程设置成守护线程，守护线程的存在不会影响JVM退出（只要有一个非守护线程运行，JVM就不会退出，否则JVM将会退出，JVM退出时，所有守护线程都会被停止）

线程组：

在Java当中，每一个线程都属于某个线程组，main线程属于 system 线程组，当我们在new Thread() 的时候，如果不指定当前线程属于哪个线程组，默认属于创建当前线程的线程所属的线程组，也就是 parent.getThreadGroup()，ThreadGroup可以批量interrupt组内线程。

![img](https://images2015.cnblogs.com/blog/801753/201510/801753-20151005180622909-789401754.png)





## 文件IO















## 网络编程



### InetAddress

对计算机 IP 地址的抽象，



![image-20200415140048096](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200415140048096.png)



什么是 DNS spoofing attacks（DNS欺骗攻击）

![img](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/102017_0742_Ettercap1.png)





参考：https://stackoverflow.com/questions/1256556/how-to-make-java-honor-the-dns-caching-timeout





### NIO



![image-20200321120909713](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200321120909713.png)





如果一个SocketChannel调用 register 方法注册到Selector上面会返回一个SelectionKey，这个SelectionKey注册在Selector上面，如果调用 cancel 方法取消注册时，SelectionKey并不会被移除，会被添加到Selector的cancelled-key set，等到下次运行 Selector.select 方法的时候才会清除这些被取消的SelectionKey



参考：

https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SelectionKey.html











form表单提交的数据，如果是通过 ```application/x-www-form-urlencoded``` 类型发送的，就是把表单数据转换成一个字符串，这个字符串把表单所有控件的值都囊括到一起，作为消息内容发送到服务端。而 ```multipart/form-data``` 不一样，这种类型下，消息内容中表单每个控件的值都是分隔开来的，各自成为一块。







###HttpUrlConnection



#### multipart/form-data request

主要的有两个点，一个是Content-Type HTTP header需要设置成

 ```Content-Type: multipart/form-data;boundary="boundary"``` ；

另一个是HTTP request body的格式要设置成：

```html
--boundary 
Content-Disposition: form-data; name="field1" 
Content-Type: XXX

value1 
--boundary 
Content-Disposition: form-data; name="field2"; filename="example.txt" 
Content-Type: XXX

value2

......

--{boundary}--\r\n (<-这个是结尾)
```



其中，```Content-Disposition``` 指明了这个part的相关信息

```
Content-Disposition: form-data; [name="xxx";] [filename="xxx"]
```



#### application/x-www-form-urlencoded request

要发送这种格式的http请求，最重要的就是设置http request body的格式是 param=value&...这种格式，并且整个字符串要使用url encode进行编码

```java
try{
  // create http connection from url
	String requestUrl = "<Url here>";
	String urlParameters  = "param1=data1&param2=data2&param3=data3";
  // url encode
  urlParameters = URLEncoder.encode(urlParameters, StandardCharsets.UTF_8);
	byte[] postData = urlParameters.getBytes();
	URL url = new URL(requestUrl);
	HttpURLConnection conn= (HttpURLConnection) url.openConnection(); 

	// set http request method
	conn.setRequestMethod("POST");

	// set http request header
	conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded"); 
	conn.setRequestProperty("charset", "utf-8");

	// indicate http has request body
	conn.setDoOutput(true);
  // indicate will read response body
  conn.setDoInput(true);

	// send http request body
	BufferedOutputStream wr = new BufferedOutputStream (connection.getOutputStream ());
	wr.write(postData);
	wr.flush ();

	// get response status code
  int statusCode = conn.getResponseCode();
  if (statusCode != 200) {
    return;
  }
  
  HttpResponse response = new HttpResponse();
  response.setHeaders(conn.getHeaderFields());
  response.setStatus(statusCode);
  
  // get response body
	InputStream is = connection.getInputStream();
  if (is != null) {
       ByteArrayOutputStream outStream = new ByteArrayOutputStream();
       byte[] buffer = new byte[1024];
       int len = 0;
       while ((len = is.read(buffer)) != -1) {
           outStream.write(buffer, 0, len);
       }
       response.setBody(outStream.toByteArray());
  }
  
} catch (IOException e) {
  
} finally {
  if(wr != null) {
    try{
      wr.close();
    }catch(IOException e) {
      e.printStackTrace();
    }
  }
  
  if(response != null) {
    try {
      response.close();
    }catch(IOException e) {
      e.printStackTrace();
    }
  }
}


```



上面代码使用了自定义的HttpResponse类：

```java
public class HttpResponse {
    private Map<String, List<String>> headers;
    private byte[] body;
    private String encoding;
    private int status;

    public HttpResponse() {
    }

    public Map<String, List<String>> getHeaders() {
        return headers;
    }

    public void setHeaders(Map<String, List<String>> headers) {
        this.headers = headers;
    }

    public byte[] getBody() {
        return body;
    }

    public void setBody(byte[] body) {
        this.body = body;
    }

    public String getEncoding() {
        return encoding;
    }

    public void setEncoding(String encoding) {
        this.encoding = encoding;
    }

    public int getStatus() {
        return status;
    }

    public void setStatus(int status) {
        this.status = status;
    }
}
```



并不是所有的XXXOutputStream都有Java代码级别的缓存，BufferedOutputStream就有缓存，它的flush方法会调用outputstream.write方法，把缓存的数据都写到流，但是DataOutputStream的flush方法就是直接调用OutputStream的flush方法，默认什么都不做。



Java HttpUrlConnection的connect()方法，是发起连接的意思，URL.openConnection()只是创建了一个HttpURLConnection请求对象，调用HttpURLConnection对象的getOutputStream()方法和getInputStream()方法都会隐式调用HttpURLConnection的connect()方法。







#### application/x-www-form-urlencoded 字符编码

这种类型的请求中，请求体，也就是消息内容部分的字符是经过url 编码后的字符，规范规定了，url 编码字符集包括：

> Url中只允许包含英文字母（a-zA-Z）、数字（0-9）、-_.~4个特殊字符以及所有保留字符，其中，保留字符包括 !	*	'	(	)	;	:	@	&	=	+	$	,	/	?	#	[	]

对于允许范围之外的字符，则需要使用允许的字符集对其进行编码。



ASCII码中保留字符的编码：

| !     | *     | "     | '     | (     | )     | ;     | :     | @     | &     |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| `%21` | `%2A` | `%22` | `%27` | `%28` | `%29` | `%3B` | `%3A` | `%40` | `%26` |
| **=** | **+** | **$** | **,** | **/** | **?** | **%** | **#** | **[** | **]** |
| `%3D` | `%2B` | `%24` | `%2C` | `%2F` | `%3F` | `%25` | `%23` | `%5B` | `%5D` |

ASCII码中不安全字符的编码：

> 空格、< > { } | \ ^ ` ~

不安全的字符采用百分号编码，就是 % + 字符字节十六进制字符串



非ASCII码字符的编码：

先对字符进行一次编码，比如UTF-8编码，然后对编码后的字节进行百分号编码，也就是每个字节编码成 % + 字节的十六进制字符串

<span style="color:red">注意：非ASCII码字符是没有统一的标准编码方式的</span>





#### HTTP协议数据包的编码方式

传输层TCP只传输字节流，HTTP数据包，包含的是文本内容，因此，接收到HTTP数据包后，应用层需要从二进制流中解析出HTTP数据包的文本内容，一旦涉及到字节和字符转换，就有编码

> HTTP 1.1 uses US-ASCII as basic character set for the request line in requests, the status line in responses (except the reason phrase) and the field names but allows any octet in the field values and the message body.

HTTP1.1协议规定，基本的字符编码是US-ASCI

参考：

https://blog.csdn.net/honghailiang888/article/details/49901993

https://stackoverflow.com/questions/818122/which-encoding-is-used-by-the-http-protocol

https://juejin.im/post/5d521575f265da03ee6a4bda





