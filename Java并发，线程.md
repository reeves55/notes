#Java并发，线程







数据操作不一定会引起线程并发问题，<span style="color:red">只有对共享数据的非原子性操作会引发多线程并发问题</span>。

如何让内存中的数据在多线程环境当中保持正确性，关键点在于对线程进行同步，



```java
package java.util.concurrent.atomic;
```

并发数据类型，这些数据类型不仅封装了某种类型的值，还封装了这些值 <span style="color:red">**在多线程环境中的同步操作**</span>，atomic包下的数据类型，<span style="color:red">基于CPU的CAS原子指令，实现同步操作</span>。

* ```AtomicInteger```：