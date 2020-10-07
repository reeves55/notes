# JUC并发包



## Atomic类

原子操作类，核心都是通过Unsafe的CAS操作来更新核心属性的值，其实核心也就几个方法：

```java
// 原子更新引用类型值
public final native boolean compareAndSwapObject(Object target, long offset, Object expect, Object update);

// 原子更新 int 值
public final native boolean compareAndSwapInt(Object target, long offset, int expect, int update);

// 原子更新 long 值
public final native boolean compareAndSwapLong(Object target, long offset, long expect, long update);
```





### 基本类型的原子类包装

数据包含在原子类当中，通过调用原子类对象的方法并发地改变原子类对象的状态



#### AtomicBoolean

使用 ```private volatile int value;``` 猜测可能是因为boolean类型的值在不同虚拟机可能会有不同的实现，hotspot虚拟机在处理boolean类型的时候都不是固定的形式，单个boolean类型的变量是使用 int 类型来存储值得，但是boolean数组则是使用 byte 来存储它的值的，而ini类型在虚拟机规范里就已经写清楚了占用4个字节，32位



#### AtomicInteger

核心的数据操作  ```unsafe.compareAndSwapInt(this, valueOffset, expect, update);```











### 原子更新工具类

原子更新工具类原子性更新其他对象属性的值，工具类自己不包含数据状态，但是 ⚠️ 需要注意，更新的目标对象的属性必须满足特定条件，

```AtomicIntegerFieldUpdater```        =>   ```volatile int```

```AtomicLongFieldUpdater```              =>   ```volatile long```

```AtomicReferenceFieldUpdater```    =>   ```volatile XXX```



#### AtomicIntegerFieldUpdater





#### AtomicLongFieldUpdater





#### AtomicReferenceFieldUpdater









## Lock+Condition(管程)

Lock接口、Condition接口，以及Lock接口的三种实现：ReentrantLock、ReentrantReadWriteLock、StamptedLock





### Lock



#### AbstractQueuedSynchronizer









## 并发集合











## ExecutorService

任务执行器，



### 阻塞队列



