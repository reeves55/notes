# Java多线程



![xXRMgQIAAAQIECBAgQIAAAQIECBAgQIA](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/xXRMgQIAAAQIECBAgQIAAAQIECBAgQIA.jpg)



为什么要使用多线程❓

第一，提高单个核心的CPU利用率，如果一个核心上只有一个线程，那么这个线程执行过程中如果因为I/O等操作发生阻塞，不再使用CPU，则CPU空闲，计算资源浪费，可以通过多线程，当正在执行的线程陷入阻塞状态时，占用CPU资源，提高CPU核心的吞吐量；

第二，利用CPU多核优势，不同核心并行运行任务；





线程并发问题❓

首先要明白一个问题，线程并发并非一定会对程序的正确性造成影响，



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200725232002567.png" alt="image-20200725232002567" style="zoom:50%;" />



**单核CPU**：没有可见性问题，CPU缓存只有一份，不同线程看到的都是一份数据，但是有原子性问题（假设一个线程在32位机器上修改一个long型变量的值，如果只修改了32位，就被切换到另外一个线程，那另外一个线程读取这个变量的值就是中间的错误值）

**多核CPU**：可见性问题、指令重排序问题、原子性问题



```CPU多副本缓存 => 可见性问题```

类似于分布式数据一致性的问题，两个核心都有同一个内存数据的副本缓存，一个核心修改了数据，必须经过同步操作才能让另外一个CPU核心知晓，否则数据就是不一致的

**解决方案**：<span style="color:green">禁用CPU缓存，都直接操作内存 / 多CPU核心缓存同步</span>





```非原子指令 + 线程切换 => 原子性问题```

高级语言执行的操作转换成CPU指令可能有多条，多条CPU指令的执行不具有原子性，再者，线程执行过程中随时可能被切换出CPU，导致语义上原子操作实际上分为多个阶 段来做，这个时候 <span style="color:red">别的线程就可以在数据的中间状态下对数据进行修改</span>

**解决方案**：<span style="color:green">线程互斥访问临界区，共享数据读写都在临界区中进行，由于线程串行进入临界区，所以没有线程会在上一个线程未完成操作前打断其执行过程，因此保证了语义上的原子操作</span>





```编译和CPU执行优化 => 顺序性问题```

指令重排导致程序执行的指令顺序可能并不按照编码时的顺序

**解决方案**：<span style="color:green">用标识告诉编译器 / CPU，此处不能重排序</span>





## 并发问题和解决方案



### Java内存模型

**Java 内存模型** 规范了 JVM 如何提供按需禁用缓存和编译优化的方法，可以解决内存可见性问题和顺序性问题，防止底层硬件优化导致并发程序运行正确性受到影响

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200725233113574.png" alt="image-20200725233113574" style="zoom:50%;" />



**Happens-Before原则**（JVM给Java程序员的承诺和契约）：

1. ```程序的顺序性规则```：在一个线程中，按照程序顺序，前面的操作 Happens-Before 于后续的任意操作
2. ```volatile 变量规则```：对一个 volatile 变量的写操作， Happens-Before 于后续对这个 volatile 变量的读操作
3. ```传递性```：如果 A Happens-Before B，且 B Happens-Before C，那么 A Happens-Before C
4. ```synchronized锁的规则```：对synchronized锁的解锁 Happens-Before 于后续对这个synchronized锁的加锁
5. ```线程 start() 规则```：线程 A 启动子线程 B 后，子线程 B 能够看到主线程在启动子线程 B 前的操作
6. ```线程 join() 规则```：线程 A 等待子线程 B 完成（ A 调用线程 B 的 join() 方法），当子线程 B 完成后（线程 A 中 join() 方法返回），A线程能够看到子线程B的操作



### Java管程

管程可以实现线程互斥，是解决并发原子性问题的方案，除了解决互斥问题之外，还可以实现多线程同步



#### synchronized

synchronized有相关的Happens-Before规则保证可见性和顺序性



#### Lock & Condition

相对于，优势在于：

1. ```能够响应中断```
2. ```支持超时```
3. ```非阻塞地获取锁```



```java
// 支持中断的 API
void lockInterruptibly() throws InterruptedException;

// 支持超时的 API
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

// 支持非阻塞获取锁的 API
boolean tryLock();
```





#### 管程保证可见性和顺序性

Java的Happens-before原则当中包含了synchronized关键字，```synchronized锁的规则```：对synchronized锁的解锁 Happens-Before 于后续对这个synchronized锁的加锁，但是Lock并没有直接相关的Happens-Before原则对应的上，其实Lock是利用了 volatile 相关的 Happens-Before 规则，加锁时会对volatile变量的值进行读写，释放锁的时候也会对volatile变量进行读写





### 锁带来的问题



#### 死锁

如果锁的力度太细，也就是说一个锁只能保护对一部分数据的原子操作，而在实际情况中，我们所需要语义上的原子性可能是需要对多个数据进行原子性操作，这种情况下就有可能导致死锁

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/A54oNxO8ttnnjcfje0ggQgAAEIAABCKw.jpg" alt="A54oNxO8ttnnjcfje0ggQgAAEIAABCKw" style="zoom:50%;" />



死锁的必要条件（吃着碗里，看着锅里的）：

* 互斥
* 占用并等待
* 不可抢占
* 循环等待：循环等待才会出现死锁，而循环等待是要看时机的，并不是一定会循环等待





##### 处理死锁

> 检测死锁，并恢复









> 避免死锁发生的几种策略



* 一次性申请完所有资源（原子性），如果拿不到所有资源，则等待

* 礼让他人，拿不到所有资源时，线程主动放弃已持有的资源（可能有活锁）

* 按序申请资源，所有线程加锁顺序相同，不会存在循环问题，也就没了死锁







#### 活锁

礼貌遇到了尴尬的情况，两个线程遇到了相互等待对方资源，于是礼貌地放弃自己已经持有的资源，然后重试，结果发现重试的时候，又出现了相互等待的情况，于是又放弃并重试，如果两个线程步调一致，那两个线程会永远处于礼让状态，就是活锁，解决活锁很简单，只需要两个线程进行随机等待一段时间再尝试获取资源即可



#### 饥饿

所谓“饥饿”指的是线程因无法访问所需资源而无法执行下去的情况。“不患寡，而患不均”，如果线程优先级“不均”，在 CPU 繁忙的情况下，优先级低的线程得到执行的机会很小，就可能发生线程“饥饿”；持有锁的线程，如果执行的时间过长，也可能导致“饥饿”问题。

解决“饥饿”问题的方案很简单，有三种方案：一是保证资源充足，二是公平地分配资源，三就是避免持有锁的线程长时间执行。





#### 性能下降

阿姆达尔（Amdahl）定律，代表了处理器并行运算之后效率提升的能力

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200726101327996.png" alt="image-20200726101327996" style="zoom:50%;" />

公式里的 n 可以理解为 CPU 的核数，p 可以理解为并行百分比，那（1-p）就是串行百分比了，也就是我们假设的 5%。我们再假设 CPU 的核数（也就是 n）无穷大，那加速比 S 的极限就是 20。也就是说，如果我们的串行率是 5%，那么我们无论采用什么技术，最高也就只能提高 20 倍的性能







## 并发工具 - 开箱即用



![ZKg5vo78AAAAASUVORK5CYII](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/ZKg5vo78AAAAASUVORK5CYII.jpg)





### 并发安全数据类型

直接把并发安全控制手段融合到数据结构当中，这样就能创造出并发安全的数据类型，使用上更加方便，



#### 原子类 (无锁)

基于CPU支持的CAS原子指令实现

![gRBjUyNcTt0AAAAASUVORK5CYII](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/gRBjUyNcTt0AAAAASUVORK5CYII.jpg)









#### 同步容器



基于 synchronized 这个同步关键字来将本来非线程安全的容器类变成线程安全的，相当于装饰器模式

```java
// 同步列表
List list = Collections.synchronizedList(new ArrayList());
// 同步Set
Set set = Collections.synchronizedSet(new HashSet());
// 同步Map
Map map = Collections.synchronizedMap(new HashMap());
```



Vector、Hashtable、Stack



#### 并发容器

![lnwRDIyggYsZOVn461zRDIBAQ4iPHPVt](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/lnwRDIyggYsZOVn461zRDIBAQ4iPHPVt.jpg)







##### CopyOnWriteArrayList

如果是读取操作，则直接读取数据对应位置的值即可，如果是修改数组中的数据，或者向数组增加/删除元素，则 ① 先拷贝一份数组，② 在新的数组上做修改

![j8JDeUsD1pGpwAAAABJRU5ErkJggg](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/j8JDeUsD1pGpwAAAABJRU5ErkJggg.jpg)



⚠️ 注意，CopyOnWriteArrayList适合写非常少的场景，而且如果一边写一边读的话，读的是快照，写之前的读操作读取的是快照，并非最新数据







##### ConcurrentHashMap





⚠️ 注意，ConcurrentHashMap不允许null key和null 值，其实map当中的null值是有二义性的，如果map.get("key")返回null，并不代表一定不存在这个key的映射，可能  key -> null，value就是null，map运行null值，因为map还可以调用map.contains("key")来辅助判断是否存在key -> null 映射，但是ConcurrentHashMap两次调用不具有原子性，可能中间状态发生变化









##### ConcurrentSkipListMap









##### CopyOnWriteArraySet











##### ConcurrentSkipListSet







单端阻塞队列

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/RKgYad dcOSkQAJkEBJBGi0KQkTI5EAC.jpg" alt="RKgYad dcOSkQAJkEBJBGi0KQkTI5EAC" style="zoom:50%;" />



双端阻塞队列

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/C6NNAiRAAiSQTgJU7KSzXVgrEiABEiAB.jpg" alt="C6NNAiRAAiSQTgJU7KSzXVgrEiABEiAB" style="zoom:50%;" />







##### ArrayBlockingQueue











##### LinkedBlockingQueue











##### SynchronousQueue











##### LinkedTransferQueue







##### PriorityBlockingQueue







##### DelayQueue











##### ConcurrentLinkedQueue













##### ConcurrentLinkedDeque













### 协作(等待唤醒)

一个完整的任务需要多个线程配合完成，既然需要配合，肯定就涉及到了先后问题，一个线程先做一部分，然后另外一个线程接着做另外一部分，但是由于线程运行之后，不加任何控制手段的话，线程是乱序执行的，要让线程按照某种顺序执行，就得让后运行的线程先阻塞，等到前一部分的工作完成了，再唤醒这个阻塞的线程，所以线程协作的关键就是等待唤醒



#### 1. 信号量 - Semaphore

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/H8b57CrHbUewgAAAABJRU5ErkJggg.jpg" alt="H8b57CrHbUewgAAAABJRU5ErkJggg" style="zoom:50%;" />

- init()：原子性操作，设置计数器的初始值
- down()：原子性操作，计数器的值减 1；如果此时计数器的值小于 0，则当前线程将被阻塞，否则当前线程可以继续执行
- up()：原子性操作，计数器的值加 1；如果此时计数器的值小于或者等于 0，则唤醒等待队列中的一个线程，并将其从等待队列中移除



信号量语义上其实就具有 ```限流``` 的意思（就像你去吃饭，座位就那么多，被坐满了，你就只能等着，等人家吃完，你再坐），只不过信号量限制的是同时运行的线程数量，它允许多个线程获得执行的许可，但是数量是有限的，当许可被用完，新来的线程只能等待。信号量还可以用来实现 ```池化资源``` ，也就是某些资源数量有限，抢完了就只能等别人不用了你再用，比如说数据库连接池、对象池等等





#### 2. 管程(Monitor)中的等待唤醒



##### synchronized

synchronized的等待唤醒机制是通过 wait 和 notify 来实现的，完整的 wait 和 notify方法如下：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200726085014259.png" alt="image-20200726085014259" style="zoom:80%;" />



核心其实就三个方法，```wait(long timeout)```、```notify()```、```notifyAll()```，具体用法：



wait操作会将当前获取到锁的线程阻塞，添加到锁对象的等待队列当中，并释放锁，⚠️ 但是需要注意的是，wait方法直接调用默认当前对象 this 是锁对象，如果锁对象不是当前对象，要使用 lock.wait 这种方式调用，另外一点要注意的是，wait操作会立马释放锁，不再向下执行语句，因为你要等待了嘛，就不会继续执行了

```java
Thread t1 = new Thread(() -> {
  	// 锁对象是lock
    synchronized (lock) {
        System.out.println("get lock...");
        try {
          	// 在锁对象上等待，而不能在其他对象上等待
            lock.wait(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
});
```





notify & notifyAll

⚠️ 注意，调用这两个方法并不会立马唤醒等待线程，而是调用这两个方法的线程退出同步代码块时，才会去唤醒等待线程，如果调用notify/notifyAll的线程无法退出同步代码块，则等待线程永远不会被唤醒，在hotspot虚拟机的实现里，notify唤醒等待的第一个线程 FIFO，但是虚拟机规范里面并没有说notify有这个规定。notifyAll会按照LIFO，先唤醒最后一个等待的线程，然后这个线程被唤醒后竞争锁，拿到锁之后，执行同步代码块，当线程执行完同步代码块了，再唤醒最后一个等待的线程，并非这些线程一起唤醒，没有惊群现象。



调用 Thread.sleep(long timeout) 时，线程不会释放锁，因为这个方法并不像 Object.wait() 必须是获取锁之后才能调用，在任何时候都能够让当前线程睡眠。





#### 3. 其他协作工具类

 

##### CountDownLatch

CountDownLatch 主要用来解决一个线程等待多个线程的场景，⚠️ 注意，CountDownLatch一旦数量减为0，就不能再使用了，一旦计数器减到 0，再有线程调用 await()，该线程会直接通过



##### CyclicBarrier

CyclicBarrier 是一组线程之间互相等待，而且比较好的一个点就是 <span style="color:green">CyclicBarrier 的计数器是可以循环利用的</span>





### 互斥

思考这么一个问题，并发的时候，不同线程所做的操作一定是不可以并行的么，其实并不是，比如说读-读操作，本身操作就不是互斥的，可以并行执行的，这个时候其实不需要把并发的线程串行化，根据场景不同，线程之间的互斥程度不同，可以使用不同的线程并发安全控制手段，就像是线程有竞争关系是不错，但是竞争也分激烈程度的，



#### 互斥锁(写多)











#### 读写锁(读多写少)

1. 允许多个线程同时读共享变量；
2. 只允许一个线程写共享变量；
3. 如果一个写线程正在执行写操作，此时禁止读线程读共享变量



读写锁允许多个线程同时读共享变量，但读写锁的写操作是互斥的，

⚠️ 注意：ReadWriteLock 并 <span style="color:red">不支持读锁升级为写锁，只支持写锁降级为读锁</span>

```java
// 读缓存
r.lock();         ①
try {
  v = m.get(key); ②
  if (v == null) {
    // 读锁还没有释放，此时获取写锁，会导致写锁永久等待，最终导致相关线程都被阻塞，永远也没有机会被唤醒
    w.lock();
    try {
  		// do something
    } finally{
      w.unlock();
    }
  }
} finally{
  r.unlock();     ③
}
```





#### StampedLock (jdk1.8)

StampedLock 不支持重入，另外，StampedLock 的悲观读锁、写锁都不支持条件变量，还有一点需要特别注意，那就是：如果线程阻塞在 StampedLock 的 readLock() 或者 writeLock() 上时，此时调用该阻塞线程的 interrupt() 方法，会导致 CPU 飙升



```java
final StampedLock sl = 
  new StampedLock();
 
// 乐观读
long stamp = 
  sl.tryOptimisticRead();
// 读入方法局部变量
......
// 校验 stamp
if (!sl.validate(stamp)){
  // 升级为悲观读锁
  stamp = sl.readLock();
  try {
    // 读入方法局部变量
    .....
  } finally {
    // 释放悲观读锁
    sl.unlockRead(stamp);
  }
}
// 使用方法局部变量执行业务操作
......
```





### 分工

























