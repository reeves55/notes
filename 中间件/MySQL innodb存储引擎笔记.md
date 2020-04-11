





# MySQL innodb存储引擎笔记

MySQL是一个软件，一个MySQL数据库实例就是一个进程，MySQL实例是一种单进程多线程的程序。





## MySQL架构

主要组成部分：

* 连接池
* 管理服务和工具
* SQL接口
* 查询分析器
* 优化器
* 缓冲（Cache）
* 插件式存储引擎
* 物理文件



![image-20200202101441269](/Users/reeves/Library/Application Support/typora-user-images/image-20200202101441269.png)



客户端连接MySQL服务程序的方式：

1. TCP/IP
2. 命名管道和共享内存
3. UNIX域套接字



## innodb架构

![image-20200202103032348](/Users/reeves/Library/Application Support/typora-user-images/image-20200202103032348.png)



### 后台线程



####1. Master Thread

主要负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性。

包括脏页的刷新、合并插入缓冲（INSERT BUFFER）、UNDO页的回收等。

innodb引擎中，数据的读写都是基于内存缓冲的，为了保证内存数据和磁盘数据的一致性，就需要一个”同步“操作，这个”同步操作“就是Master Thread来做的



#### 2. IO Thread

包括write、read、insert buffer、log IO Thread，负责AIO请求的回调处理。

#### 3. Purge Thread

事务提交之后，由Purge Thread来回收已经使用并分配的undo页。



### 内存



#### 1. 缓冲池

数据的读和写，直接操作的是内存中的缓存，然后才会刷新到磁盘（Checkpoint机制），以此来提高MySQL的性能。

![image-20200202104154326](/Users/reeves/Library/Application Support/typora-user-images/image-20200202104154326.png)

#### 2. LRU List、Free List、Flush List

缓冲池是一大块内存区域，大小有限，如何管理这片内存区域，就是这几个List：

* Free List是缓冲池内存区域中还没有被使用的页列表，MySQL刚启动时，LRU List是空的，Free List是满的，只要Free List不为空，则从磁盘加载页到缓冲中时，从Free List中查找空闲页，然后从Free list中删除该页并添加到LRU List中；

* 最频繁使用的页（16KB）在LRU List的前端，最少使用的放在尾端，当Free List为空，也就是缓冲池不能存放新读取到的页时，将首先释放LRU列表中尾端的页；
* LRU List中的页的数据如果被修改了，就会出现内存数据和磁盘数据的不一致性，也就是脏页，这个页就会被添加到Flush List中，注意此时LRU List和Flush List都有这个页，数据库会通过Checkpoint机制将Flush List中的页刷新到磁盘，保持内存和磁盘数据的一致性。

#### 3. 重做日志缓冲

innodb存储引擎首先将重做日志信息放入到这个缓冲区，然后按照一定频率将其刷新到磁盘上的重做日志文件，重做日志缓冲的大小应该根据每秒重做日志的数量来确定，重做日志越多，缓冲应该越大。

重做日志缓冲的刷新点有：

* Master Thread每1秒将缓冲刷新到磁盘；
* 每个事务提交时，会把重做日志缓冲刷新到磁盘；
* 当重做日志缓冲剩余空间小于1/2时，刷新到磁盘。

#### 4. 额外的内存池

不清楚

####5. Checkpoint刷新缓冲到磁盘机制

MySQL对数据的直接操作都基于内存缓冲的，但是数据最终必须刷新到磁盘，实现数据库的持久性目标。

缓冲内存空间有限，无法缓存全部数据库数据，发生更改的数据页在换出缓冲区时，如果不刷新到磁盘，将会丢失更新，发生数据的不一致。

重做日志大小有限，因为无限大的重做日志用来恢复数据库数据量过于庞大，这导致了重做日志的内容需要进行删除，以容纳最新的重做日志。

这些机制都是Checkpoint来实现的，Checkpoint功能如下：

* 缩短数据库的恢复时间；
* 缓冲池不够用时，将脏页刷新到磁盘；
* 重做日志不可用时，刷新脏页。



缓冲区的Checkpoint刷新机制是在缓冲大小有限条件下保证数据一致性的手段，重做日志的刷新是在重做日志大小有限条件下保证宕机，数据库能够快速恢复的手段



Checkpoint种类有：

* Master Thread Checkpoint
* FLUSH_LRU_LIST Checkpoint
* Async/Sync Flush Checkpoint
* Dirty Page too much Checkpoint



#### 6. Master Thread工作方式

具有最高的线程优先级...

#### 7. innodb关键特性

包括：

* 插入缓冲
* 两次写
* 自适应哈希索引
* 异步io
* 刷新邻近页

##### 插入缓冲

提升性能

##### 两次写

保证数据页的可靠性

写失效问题：缓冲中的脏页在写入到磁盘过程中宕机，写入未完成，导致磁盘上的数据部分被更改，innodb的重做日志记录的是对物理页的操作，我们不知道重做日志中哪些操作是缓冲刷新到磁盘已经执行了的，哪些还没有执行，就无法直接应用重做日志到磁盘数据文件上，所以，需要一个刷新缓冲之前的页的副本数据，才能通过重做日志来恢复磁盘数据页。

两次写机制，如下图：

doublewrite buffer会将多个脏页聚集在一起，这些脏页可能对应不同物理位置的数据页，然后先将doublewrite buffer数据顺序写入到共享表空间文件中，顺序IO，共享表空间文件在磁盘上，然后再把doublewrite buffer中的脏页写离散地写入到磁盘的不同位置上。如果离散write地时候出现写失效，那就通过共享表空间来进行数据的恢复。<span style="color:red">有个问题，从共享表空间拿到脏页数据进行恢复时，为什么还要应用重做日志</span>

![img](https://images2017.cnblogs.com/blog/1113510/201707/1113510-20170726195345906-321682602.png)



##### 自适应哈希索引

innodb引擎自动帮我们做了，这是一种针对等值查询，也就是 ```where col="xxx"```形式查询时的优化手段，innodb开始使用哈希索引时，就不需要通过B+树来查询数据了，直接使用hash值来查询，时间复杂度为O(1)，B+树查询时间复杂度和树的高度有关，但是innodb引擎创建哈希索引是有条件的。

##### 异步IO

要访问多个页时候，可以连续发出多个IO请求，而无需等待前一个IO完成之后再发出新的IO请求

##### 刷新邻接页



## 文件

**MySQL软件**在运行过程中使用的文件包括：

* 参数文件
* 日志文件
  * 错误日志
  * binlog
  * 慢查询日志
  * 查询日志
* 套接字文件
* pid文件（MySQL进程id写在这个文件里）
* 表结构定义文件（.frm文件和使用的存储引擎无关）
* 存储引擎文件（不同存储引擎的文件可能不同）



### innodb存储引擎文件

包括：

* 表空间文件
* 重做日志文件



## innodb表

我们考虑一张表对应一个表空间的情况，也就是启用了 ```innodb_file_per_table``` 参数，innodb当中，一张表就是一个B+树，叶子节点就是数据，非叶子节点就是索引，那么这一个B+树，在一个表空间里分为几部分存储：

* 数据段
* 索引段
* 回滚段



数据段存放叶子节点，索引段存放非叶子节点，一个B+树就拆开来存储了。一个段里面有多个区，一个区的大小是固定的1M，由于一个页的大小默认是64KB，一个区默认有64页，注意，这里的64个页是连续的，因为申请区的时候是一次性申请1M的磁盘空间，连续的页有助于提升性能。



![image-20200203102719557](/Users/reeves/Library/Application Support/typora-user-images/image-20200203102719557.png)



innodb页类型有许多种：

* 数据页（B-tree Node）
* undo页（undo log page）
* 系统页（System Page）
* 事务数据页（Transaction system page）
* 插入缓冲位图页（Insert buffer bitmap）
* 插入缓冲空闲列表页（Insert buffer free list）
* 未压缩的二进制大对象页（Uncompressed BLOB page）
* 压缩的二进制大对象页（compressed BLOB page）



### 页格式



![image-20200203121702997](/Users/reeves/Library/Application Support/typora-user-images/image-20200203121702997.png)



### 行格式

行存储在页当中，innodb读取的基本单位就是页，所以一次性可以读出来很多行，但是一页里面有多少行时不确定的，页的大小有：4K、8K、16K

行格式：

![image-20200203112253044](/Users/reeves/Library/Application Support/typora-user-images/image-20200203112253044.png)



rowid列只有在行没有主键的时候才会自动添加

* 变长字段长度列表记录此行所有可变的列的实际数据长度，单位字节，按照列的逆序来排列，如果长度小于255，就用一个字节来表示，否则使用两个字节表示。

* NULL标志位是每一列用一位来表示是否是NULL，如果是NULL则该列对应的位是1，否则是0，所以NULL标志位的长度和列的个数有关。



一行究竟占用多大的数据量是不确定的，就是因为有这些可变列的存在。

#### varchar类型

可变长度的字符串，varchar理论上来说最多支持65535个字节长度，但是一个数据页最多也就16KB，怎么能容纳64KB长度的列呢，这个时候叫做行溢出，实际上一个数据页如果不能容纳至少两行数据，就是发生了行溢出，行溢出的时候，会把varchar中的数据存放在一个单独的页当中，然后行记录该列的值只是一个20字节的指针。但是如果不发生行溢出，是不会把数据单独存放在另外的页里面的。

