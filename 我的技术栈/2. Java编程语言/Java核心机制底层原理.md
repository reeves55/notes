# Java核心机制底层原理



## .class文件的秘密





### 问题解答



> 子类继承父类，那子类的.class文件会有包含父类定义的变量和方法吗❓

**不会**，子类.class文件中只包含子类的变量定义和方法定义，如果子类当中有需要直接调用父类方法的地方，那么父类的方法描述符也会放在子类的常量池当中，注意，只是描述符，并不是方法本身，并标注是父类的某个方法。当访问父类的成员变量时，





## 字节码基础



访问当前类的成员变量 getField









## 线程和线程池







## 线程安全 - 锁







## JVM



### 内存管理

JVM也是一个进程，启动之后，也和普通进程一样，拥有进程地址空间，空间内布局也满足一样的结构，关键在于我们所说的JVM内存管理，指的是，<span style="color:red">JVM进程如何管理它在自己的进程地址空间申请使用的那块内存区域。</span>



64位系统，一个进程的虚拟内存空间布局如下：

![251606486466813](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/251606486466813.png)





```c
void* malloc(size_t size);
```



**<span style="color:red">Heap区域</span> 内存申请和释放**

JVM通过调用malloc()为Heap区域申请内存资源，通过free()释放内存，malloc申请空间应小于128KB（可在操作系统配置，128KB为默认值），新分配的堆区域在原本已分配区域的相邻上方，底层基于系统调用 brk() 进行内存申请：

![img](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/Center-20200315191253266.jpeg)



**<span style="color:red">Mapping Area区域</span> 内存申请和释放**

JVM通过调用malloc()为Mapping Area区域申请内存资源，通过free()释放内存，malloc申请空间应大于128KB（可在操作系统配置，128KB为默认值），新分配的堆区域在Mapping Area一块区域，底层基于系统调用 mmap() 进行内存申请，通过系统调用 munmap()释放内存：

![img](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/Center-20200315191936986.jpeg)



### JVM内存布局

其实我们说的新生代，老年代以及Metaspace都是在MappingArea分配内存空间的，JVM的堆并不是指的操作系统给进程分配的内存空间当中的堆。此 ”堆（JVM Heap）“ 非彼 ”堆（Process Heap Area）“。

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200315192859290.png" alt="image-20200315192859290" style="zoom:67%;" />



