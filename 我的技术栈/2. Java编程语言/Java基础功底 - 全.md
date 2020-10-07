# Java基础功底 - 全



## 设计哲学



### 完全面向对象

面向对象特别适合使用领域模型驱动设计（DDD）的方式来开发软件，



#### 核心概念

关键字：```抽象```、```封装```、```继承```、```多态```



##### 封装

把数据封装到类当中，<span style="color:red">通过方法暴露功能，而非直接暴露数据</span>，

优点：

1. 类和类之间通过方法调用进行交互，语义更明确
2. 类当中可以对数据状态进行保护，增强软件的可靠性

缺点：

1. 如果不知道类的实现细节，或者没有详细文档，很难准确判断方法的功能效果



##### 继承

复用代码，表示逻辑概念上的继承关系

优点：

1. 代码和方法的复用，常见用法就是模板方法设计模式中 class A extends abstract class B

缺点：

1. 通过继承来扩展已有类的功能会导致很多问题，比如说java当中的 Stack extends Vector，继承相当于全盘接受，拿来主义，可能并不是完全适合自己



##### 多态

实现类与类之间解耦和程序动态性

优点：

1. 类与类解耦，不需要强耦合，提高了程序的可扩展性
2. 给程序运行时的动态性提供了基础





## Java面向对象



封装：```class```, ```abstract class```, ```private```, ```protected```, ```public```, ```default```

继承：```extends```（单继承）

多态：继承多态，```interface``` 多态



访问修饰符❓

**private**：类私有，只有类自己能访问

**protected**：package内+package外子类均能访问，重点突出继承

**default**：package内能访问，重点突出package

**public**：package内外均能访问



> 什么是主流程❓
>

程序的入口，算法的开始，就是执行某个对象的某个功能，这个功能可以理解为 start，暂且把这个功能叫做入口功能吧，包含这个入口功能的对象叫做入口对象， ，这个入口功能会用到其他对象，于是，对象之间的协作就开始了。就像SpringBoot程序，XXXApplication就是一个入口对象。



> 抽象类和接口❓

抽象类是指包含抽象方法の类，抽象方法使用abstract关键字修饰（默认public，public/protected），只声明方法，没有具体实现，子类在继承抽象类的时候，如果子类不是抽象类，则必须实现抽象方法。

接口当中可以包含变量和方法，变量会被隐式地指定为public static final，方法可以只有声明没有实现，也可以使用 default 关键字修饰的方法，可以有默认实现

两者抽象的层次不一样，抽象类是对类的抽象，包括数据和行为，接口是对行为的抽象，抽象类经常用来实现模板方法设计模式，比如Spring当中的 ```AbstractApplicationContext```，Java当中的 ```AbstractQueuedSynchronizer```



> Java为什么采用单继承而不是C++的多继承❓

多重继承有二义性问题，下面这个图，假设B和C都重写了A当中的一个方法，而D继承了B和C，当D调用这个重写的方法时，究竟调用的是B的方法还是C的方法呢，这个无法自动判断，只能通过指定具体类名的方式或者避免这种写法，这就是多重继承的二义性问题

![在这里插入图片描述](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM1NjgzNzM=,size_16,color_FFFFFF,t_70.png)





## Java数据类型

程序设计语言使我们和计算机进行沟通的桥梁和工具，我们把我们的思考逻辑变成代码，然后代码变成计算机指令。通过计算机程序这个中介，我们得以让计算机按照我们的思想进行计算，并得出我们想要的结果。

数据类型是什么呢，无非就是把我们人类世界用到的数据分分类，人类世界用的不叫数据，叫信息，信息在计算机世界里翻译过来就是数据，信息有种类，数据也就有了种类，每种类型的数据都有其特点，程序设计语言给我们定义了数据类型的概念用来方便我们选择我们想要用哪种类型的数据

信息和数据之间可以相互转换，但有一个前提是必需的，那就是数据一定要有规则。没有规则的数据不能转换成信息，也就不包含任何价值。计算机上存储的都叫数据，但这些数据是否都有意义，则取决于我们有没有“翻译”这些数据的方法。举个🌰，一段密文，在没有解密方法之前，密文只是数据，它没法转换成人类可以理解的，所以它不是信息，但是一旦有了解密的方法，看似乱七八糟的密文立马就能变成人们可以理解的信息。所以重点在于：数据的解析方法，也可以当成数据的“翻译”方法。

以这个角度来看数据类型，程序设计语言定义的种种类型，实际上也就是定义了一种种解析数据的规则，int类型，在把数值写入到内存时，会把10进制的值先“翻译”二进制的01串再存储进去，在读取的时候，会把内存当中的01串数据当成一个10进制数的二进制表示形式，所以会按照2进制到10进制的转换方式把01串“翻译”成我们能理解的数字。char类型，会把2个字节16位的01串当成Unicode编码当中的数字坐标，然后通过查找Unicode数字坐标和字符的对应关系，给我们呈现可以认识的字符，而不是01二进制串。

很容易理解的是，这种转换规则是我们可以自己定义的，所以才有了像protoBuffer，它定义了信息如何转换成二进制01串，也同样定义了01串如何转换成特定类型的信息。数据类型也没有什么神秘的，它就是语言设计者定义的一些规则而已。

有了数据类型这种理解就好说明什么是强类型语言，什么是弱类型语言了，强类型语言意味着你在程序当中一个变量必须声明为一种特定的数据类型，之后这个变量的值就只能存储这一种类型的值了，如果你对这个变量做的操作超过了变量所属数据类型允许的操作范围之外，那么程序在编译期间就会报错，这个专业点叫类型安全，这样你就不可能在本来存储int整型数据的内存空间上存储float浮点数，因为变量已经定义成int类型，那么对内存当中的01串的理解方式都是按照整型的规则来的，而浮点数在内存当中01串的表示规则和整型是完全不一样的，强行把它解释成整型只能得到完全没有意义的值，所以强类型语言是绝对不允许这样做的；而弱类型语言不一样，弱类型语言基本指的是脚本语言，它没有数据类型这么一说，你想定义什么数据就定义什么数据，一个var关键字就搞定，var a = 1; var a = {key:"value"};var a = 1.5...，这都可以，你可以对一个变量赋予不同数据类型的值，比如var a=1; a=3.5; 这并不会引起错误，它的规则是什么样的呢，就是变量它只代表一片内存空间的地址，他不管你要往这片内存空间里面塞什么，加入你塞进去的是整型int，那么这片内存区域存储的就是数字的二进制01串，如果你往里面塞的是一个浮点数float，那么这片内存区域存储的就变成了浮点数在内存当中表示的01串。当你要用这个变量的时候，这片内存区域当中的01串该按照什么规则去理解呢？是不是问题就来了，既然动态语言不管你把变量赋值成什么类型，那么我怎么知道我究竟赋值了什么类型呢？如果不知道赋值了什么数据类型，那岂不是自己把自己给整懵了嘛，哎，这个问题可以用javascript来举🌰，<span style="color:blue">js 在底层存储变量的时候，会在变量的机器码的低位1-3位存储其类型信息👉</span>

- 000：对象
- 010：浮点数
- 100：字符串
- 110：布尔
- 1：整数

这就是动态语言知道当前变量所对应的内存空间里究竟存储着什么类型数据的秘密。



参考：

https://juejin.im/post/5b0b9b9051882515773ae714

https://lenciel.com/2016/09/types-in-programming-languages/



### 原始数据类型

原始数据类型指的是单纯代表不同字节数的数据类型

```原始数据类型``` => ```数据```

```封装数据类型``` => ```数据``` + ```算法``` => ```特性```



8种原始数据类型：

```byte```（1字节，-128~127）

```short```（2字节，-2^15 ~ 2^15 -1）

```int```（4字节，-2^31~2^31-1）

```long```（8字节）

```float```（4字节）

```double```（8字节）

```boolean```（1字节/4字节）：单独的boolean类型变量使用4字节存储，因为操作单个boolean类型变量使用的是ini相关的字节码指令，但<span style="color:red">boolean类型的数组使用byte相关指令来操作</span>，这个时候boolean类型变量占据1个字节存储空间，但这并不是确定的，JVM规范当中并没有确定boolean究竟占据几个字节，和具体的虚拟机实现有关系，Oracle的hotspot虚拟机是按照上面👆的方式来存储的，有可能为1字节，也有可能为4个字节

```char```（2字节）UTF-16编码

数字型都使用补码来存储（负数 => 对应正数反码 + 1）



java当中的char固定占据两个字节，无法存储所有Unicode字符，如果要使用非基本集的Unicode字符，不能使用char，而需要使用String，jdk1.8及之前版本，String底层使用char[]实现，非基本集的Unicode字符，使用两个char来存储，也就是占据4个字节，这两个char是根据特定的计算公式计算出来的（其实是将一个20bits的字符拆分成两部分，两部分都是10bits，每部分都使用两个字节的char存储，而这10bit使用固定的区间 ```[U+D800, U+DBFF]``` 和 ```[U+DC00, U+DFFF]```）这两个区间都属于unicode当中的代理字符，正常字符是不会使用这两个区间来表示的，java本身在处理的时候，读取两个字节，如果其范围是在代理字符的范围内，就知道这个字符是使用2个char，也就是4个字节表示的，然后继续读后面2个字节，最后合并这四个字节，解析为4字节的unicode字符。

两个字节可以表示的unicode字符的范围是 [U+0000 ~ U+D7FF] 和 [U+E000 ~ U+FFFF]，可以看到是不包含代理字符范围的。



数组（不是集合当中的List）又分为存储原始数据类型的数组和存储引用类型的数组，不管是原始数据类型的数组还是引用类型的数组，都是在堆当中分配内存空间的，因为数组在java当中是一个对象，其Class是“[X”，其中X是具体的数组类型，如果是int，则是“[I”

```java
// 在堆中分配内存空间
int[] intArray = new int[5];
```





#### 数组工具类

java.util.Arrays工具类提供了很多和数组相关的静态方法，







### 基本封装数据类型

基础数据类型是对原始数据类型的封装（封装成了class），主要封装了对原始数据的相关操作方法，比如 Integer.parse(String)就是把字符转成整数，而原始类型int只代表了数据，而不会有相关功能。



基础数据类型有：

* ```Byte```

  底层实现：private final byte value，hashcode = value，新建一个Byte对象jdk1.9推荐使用Byte.valueOf(byte b)，而不是new Byte(byte b)，因为Byte能表示的 -128 ~ 127范围所有Byte对象都已经被缓存了

* ```Short```

  底层实现：private final short value，hashcode = value，-128 ~ 127范围内的所有Short对象都被缓存了，通过Short.valueOf(short s)来获取缓存的Short对象

* ```Integer```

  底层实现：private final int value，hashcode = value，默认 -128 ~ 127范围内的所有Integer对象都被缓存起来了，通过Integer.valueOf(int i)来获取缓存的Integer对象，其中缓存的Integer范围上限是可以配置的（java.lang.Integer.IntegerCache.high）

* ```Character```

  底层实现：private final char value，hashcode = (int)value，<=127的Character对象都被缓存了，也是通过Character.valueOf(char c)来获取缓存的Character对象

* ```String```

  底层实现：private final char value[]，hashcode = s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]，jdk1.9之后底层实现为：private final byte[] value，然后多了一个重要的成员变量 private final byte coder; coder只有两种取值，LATIN1 = 0，UTF16  = 1，当coder是LATIN1的时候（默认就是LATIN1），一个字符使用一个字节来存储，当coder是UTF16的时候，使用两个字节来存储一个字符，当然，如果字符超过了65535的范围，就会使用4个字节来存储

* ```Long```

  底层实现：private final long value，hashcode = (int)(value ^ (value >>> 32)); -128 ~ 127范围内的所有Long对象都被缓存起来了，通过Long.valueOf(int i)来获取缓存的Long对象

* ```Boolean```

  底层实现：private final boolean value，hashcode = value ? 1231 : 1237; 

* ```Double```

  底层实现：private final double value，hashcode = ①先把double转换成long，② (int)(bits ^ (bits >>> 32));





### 集合框架

集合是用来容纳多个相同类型元素的容器，Java的集合主要包括几类 **列表**（普通列表）、**集合**（元素不重复）、**队列**（FIFO）和 **散列表**（key-value 映射），比较常用的有这么一些具体的集合类，包括 ```ArrayList```、```LinkedList```、```HashMap```、```HashSet```，还有一些不常用的比如说 LinkedHashMap、LinkedHashSet，在一些特殊场景下会用到

![2243690-9cd9c896e0d512ed](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/2243690-9cd9c896e0d512ed.jpg)



##### 列表

```ArrayList```

非线程安全，底层数据结构：

```Object[] elementData```;

数组刚new的时候 elementData为{}，是个空数组，当添加第一个元素的时候，数组容量不足，需要扩容，扩容策略为：

```java
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
  	// 新的数组容量为原先的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
  	// 分为 newCapacity < minCapacity 和 newCapacity == minCapacity两种情况
    if (newCapacity - minCapacity <= 0) {
      	// 数组刚创建时容量为0，即使扩展为1.5倍仍然为0
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
          	// 这个时候数组扩展到默认容量 DEFAULT_CAPACITY = 10
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return minCapacity;
    }
    return (newCapacity - MAX_ARRAY_SIZE <= 0)
        ? newCapacity
        : hugeCapacity(minCapacity);
}
```



扩容时，需要创建一个新的数组，容量为扩展后的长度，然后把原先数组中的元素都拷贝到新的数组中：

```java
private Object[] grow(int minCapacity) {
    return elementData = Arrays.copyOf(elementData,
                                       newCapacity(minCapacity));
}
```



⚠️ 注意事项：使用ArrayList的时候最好能提前预估一下它的容量，ArrayList不适合频繁动态扩容，特别是数组元素过多时，会消耗不少CPU和内存





##### 链表

```LinkedList```

底层数据结构 - 双向链表，非线程安全：

```Node<E> first```，```Node<E> last```，```int size = 0```

Node<E>数据结构：

```E item```，```Node<E> prev```，```Node<E> next```



LinkedList既可以当做链表来用，又可以当做队列来用，LinkedList实现了双向队列 DeQueue 接口

```java
// 添加到链表最前端
public void push(E e);
// 删除并返回链表第一个元素
public E pop();
// 获取链表第一个元素
public E peek();

// 链表最前端添加
public boolean offerFirst(E e);
// 链表最尾端添加
public boolean offerLast(E e);

// 获取不移除
public E peekFirst();
public E peekLast();

// 获取并移除
public E pollFirst();
public E pollLast();
```





##### Vector

和ArrayList功能基本相同，但是所有操作都被sychronized修饰，是线程安全的ArrayList

底层数据结构：

```Object[] elementData```



##### Stack

```Stack<E> extends Vector<E>```

Stack继承了Vector，只不过多提供了几个关于栈的方法 pop, push等等

⚠️ 这个类是一个极其差的实现类，它继承了Vector，等于继承了所有Vector中数组相关的方法，对外暴露了许多本不应该是栈这个数据结构应有的方法，是继承的典型反面教材



##### Queue

队列，没有队列的实现，只有双向队列的实现，双向队列的两种实现：

ArrayDeque 和 LinkedList



```ArrayDeque```

基于数组的队列实现，非线程安全，队列长度可以自动增长，自动增长策略和ArrayList有区别，如果数组长度在64以内，直接扩容为原来容量的2倍，否则扩容为原先的1.5倍

底层数据结构：

```Object[] elements```，```int head```，```int tail```



##### Set

不包含重复元素的集合，两种实现：

HashSet 和 TreeSet



###### HashSet

基于HashMap的非重复元素集合实现

底层数据结构：

```HashMap<E,Object> map```

HashSet把集合中的元素作为Key存储在HashMap中，不同的Key使用相同的Value值（Object PRESENT = new Object()）



```LinkedHashSet```

并没有像官网文档所说的，LinkedHashSet保存了HashSet元素的插入顺序



###### TreeSet









##### Map



###### HashMap

jdk1.7, 1.8对HashMap的改动较大，散列表的两个关键点，1. 数组扩容；2. 位置冲突解决办法；



扩容

jdk1.7时候扩容，会遍历整个hash数组的每一条链，采用头插法的方式，先计算出原先key的位置，然后头插到新位置上，在多线程环境下有可能出现环形链表问题导致CPU 100%，jdk1.8时候，采用尾插法的方式，而且不需要重新计算key的hashcode % arrayLength了，直接判断key的hashcode是否大于原先的数组长度，如果不大于，则位于新数组相同位置，否则位于 当前位置+数组长度 这个新位置，因为hashmap扩容，每次都扩容为原来的两倍，而数组长度一定是 2^n，所以可以这样计算



HashMap的capacity只能是2^n，这就需要我们对HashMap的容量做一个预估，而且HashMap存放的元素越多，越需要提前预估一下存储的元素数量，因为 2^10和2^11可是差了1024个元素呢

通过HashMap的构造函数可以设置它的capacity和load factor，



给出一个数n，如何快速计算出不小于n并且是2的幂次方的最小整数❓

这就是HashMap指定初始化容量值中遇到的一个问题，因为HashMap的capacity必须是2^x，而用户输入的初始化容量可能并不是2^x，所以需要进行一个计算来计算出不小于用户输入的容量的最小的2^x，计算方式如下



```java
static final int tableSizeFor(int cap) {
  	// 减一是为了防止cap就是2^x这种情况，如果cap = 2^x，按照这里的计算方式，最终结果就是2^x
    int n = cap - 1;
  	// n无符号右移一位和原先的n进行 OR 运算，其实就是把最高位的 1 复制到最高位的后面一位
  	// 这里执行过之后，高位一定变成 11xxxxxx
    n |= n >>> 1;
  	// 右移两位然后和旧的n值进行OR运算，其实和上面思路一样，就是把最高的2个1复制到第3和第4位
  	// 这里执行过之后，高位一定变成 1111xxxx
    n |= n >>> 2;
  	// 这里移动4位，是因为通过上面两步操作，高4位一定都是1，这里移动4位就相当于把高4位右边的4位也都
  	// 变成1
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
  	// 最后 n+1 是关键，前面已经把cap变成了 11111111 这种形式，只要 +1 就是 2^x
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```





###### TreeMap

















##### 迭代器&Fail Fast

迭代器是Java当中的一个接口，Java的集合框架支持迭代器模式，因为本身Collection接口就继承了Iterable接口

```java
public interface Collection<E> extends Iterable<E>{...}
```

实现了Iterable接口的类，可以被用在java的foreach循环当中，

```java
for(Object item : list) {
  // 循环逻辑
}

// 上面的代码在编译之后会转换成下面这样👇
Iterator i = list.iterator();
while(i.hasNext()) {
  Object item = i.next();
  // 循环逻辑
}
```



⚠️注意事项：Java集合在迭代过程当中，注意不要使用集合的方法改变集合中元素的个数，因为Java当中的集合迭代器都会在每一步 next() 方法中检查集合是否发生了变化，如果发生了变化 则会抛出 ConcurrentModificationException，正确的用法应该是调用迭代器的 remove() 方法删除元素，而不是用集合自己的 remove() 方法删除元素



```java
// 可迭代对象接口
public interface Iterable<T> {
  // 返回迭代器对象
  Iterator<T> iterator();
  
  default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
	}
  
  default Spliterator<T> spliterator() {
    return Spliterators.spliteratorUnknownSize(iterator(), 0);
	}
}

// 迭代器接口

```





##### 集合工具类



###### Collections

位于 java.util 包下面，提供了集合相关的一些常用方法

```java
// 把非线程安全集合转换成线程安全集合
// 返回套上壳儿的相应集合，代理模式，调用原方法之前都使用 synchronized(mutex)
public static <T> Collection<T> synchronizedCollection(Collection<T> c){...}

// 把原先集合变成只能读不能修改的集合
// 调用修改集合的方法会抛出 UnsupportedOperationException
public static <T> Collection<T> unmodifiableCollection(Collection<? extends T> c){...}
```









### 并发安全的集合框架





#### ConcurrentHashMap



```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh; K fk; V fv;
      	// Node数组还未初始化，在这里初始化
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
          	// 如果插入元素的位置桶为空，则CAS直接插入
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                break;                   // no lock when adding to empty bin
        }
      	// 如果Node数组该位置第一个元素的hash是MOVED(-1)，则说明现在正在扩容，多线程扩容
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else if (onlyIfAbsent // check first node without acquiring lock
                 && fh == hash
                 && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                 && (fv = f.val) != null)
            return fv;
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key, value);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                    else if (f instanceof ReservationNode)
                        throw new IllegalStateException("Recursive update");
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```















## Java多线程



### Java中的线程

Thread类，



#### 生命周期







#### 创建一个线程

创建一个线程，也就是创建一个Thread对象，Thread的构造方法有几种

```java
public Thread();
public Thread(Runnable target);
public Thread(ThreadGroup group, Runnable target);
public Thread(String name);
public Thread(ThreadGroup group, String name);
public Thread(Runnable target, String name);
public Thread(ThreadGroup group, Runnable target, String name);
public Thread(ThreadGroup group, Runnable target, String name,long stackSize);


// 上面👆的方法实际上都是调用了init方法
private void init(ThreadGroup g, Runnable target, String name,long stackSize);
```

可以看到线程有几个关键的参数：

* ```ThreadGroup group```：当前创建的线程所属的线程组，线程组是逻辑上组织线程的一种方式，但是这种组织方式所能提供的线程批量管理功能又是有限的，用处不大，可以用来帮助定位问题（假设创建了很多很多线程，如果对线程进行分组，那就可以方便找到出问题的线程是干什么的，比如说 Tomcat threads、XXX任务线程），综合来说，不建议使用 ThreadGroup相关方法（https://wiki.sei.cmu.edu/confluence/display/java/THI01-J.+Do+not+invoke+ThreadGroup+methods#space-menu-link-content）
* ```Runnable target```：当前线程所要运行的任务
* ```String name```：线程的名字
* ```long stackSize```：线程栈的空间大小





##### 线程组

一个线程可以属于一个线程组，线程可以访问包含它的线程组的相关信息，

![img](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/693275-20190102162714238-518028301.jpg)





#### 启动线程



```java
public synchronized void start() {
    /**
     * This method is not invoked for the main method thread or "system"
     * group threads created/set up by the VM. Any new functionality added
     * to this method in the future may have to also be added to the VM.
     *
     * A zero status value corresponds to state "NEW".
     */
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    /* Notify the group that this thread is about to be started
     * so that it can be added to the group's list of threads
     * and the group's unstarted count can be decremented. */
    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
              it will be passed up the call stack */
        }
    }
}
```











## Java底层原理



### 字节码



依靠 ```字节码中间语言``` + ```针对各个平台开发的JVM```，java程序员可以实现 “一次编写，到处运行”的效果

跨平台的关键就是字节码文件和JVM，因为执行Java程序，分为两步：

1. 通过javac命令将java源代码编译成.class字节码文件
2. 通过java命令启动JVM，加载并运行字节码文件



javac命令，是java语言的编译器，负责将java源程序编译成符合jvm规范的字节码文件（.java -> .class）

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/70-20200722141629316.png" alt="这里写图片描述" style="zoom:60%;" />



### JVM



#### 启动

JVM是通过java命令来启动的，java命令实际上就是个启动器，启动器（Launcher）的功能为加载平台特定的JVM动态链接库，并通过JNI启动JVM：

1. ```选择/指定```要使用哪个```JVM动态链接库```（jvm.dll / libjvm.so / libjvm.dylib）；
2. ```加载```动态链接库（LoadLibrary() / dlopen()）；
3. 创建新线程，执行JavaMain函数，在JavaMain函数中，会```传递参数```（可以指定虚拟机运行时参数）并通过JNI函数JNI_CreateJavaVM() 创建JVM实例，然后加载主类，查找并执行main方法；



