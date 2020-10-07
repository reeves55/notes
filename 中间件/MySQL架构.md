# MySQL架构





## SQL Interface

用户提交的执行语句   ```->```   SQL Interface处理  ```->```  SQL语句



### 视图







## 查询优化器



根据SQL，找到最佳的执行算法







## SQL语句执行过程





### 执行步骤

```From``` -> ```Where``` -> ```Group by``` -> ```Having``` -> ```Select```   -> ```Order by```   -> ```Limit``` 



![26e9bcc32256ee60e1ae3ec1a8d0e171](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/26e9bcc32256ee60e1ae3ec1a8d0e171-20200615142107152.jpeg)



#### 第1步. FROM

单表、多表连接 join











#####3种join算法

（**MySQL只支持 nested loop join算法**）



> **```hash join```**







> **```merged join```**







> **```nested loop join```**（嵌套循环连接）









> **```block nested loop join```**（块嵌套循环连接）









#### 第2步. WHERE

筛选行





#### 第3步. GROUP BY

结果分组



group by的语义是按照指定列分组并默认按照指定的group by列排序







##### 3种分组方法



> **1.利用 ```松散索引``` 实现分组**



只要扫描索引就能得到分组结果，而且不需要扫描所有索引项即可获取分组信息。



![img](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/20160529233451.jpg)



explain关键信息：

```mysql
Extra: Using index for group-by
```





> **2.利用 ```紧凑索引``` 实现分组**



必须扫描范围内所有索引才能获取到分组结果，比如 select count(*) 这种，需要扫描每个索引项，记录一个group 当中的索引项数量。



![img](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/20160529233452.jpg)



explain关键信息：

```mysql
Extra: Using index
```





> **3.利用 ```临时表``` 实现分组**



临时表的生成：创建一个以group by列为唯一主键的临时表，把根据where条件得到范围内的记录插入到这个表当中，最后对这个表以group by列排序，最后输出到客户端。



![img](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/20160529233453.jpg)



explain关键信息：

```mysql
Extra: Using Using filesort
```





⚠️ MySQL当中可以select不在group by中的列，可以称作多值列，但多值列究竟取哪一个值，是随机的，如果多值列每个值都相同，那select出的多值列就没啥问题，但是如果值不相同，由于MySQL选择的随机性，select出的结果可能并没有意义。

![image-20200615131310758](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200615131310758.png)

参考：https://dev.mysql.com/doc/refman/5.6/en/group-by-handling.html



⚠️MySQL还允许在group by当中使用select中列的别名，在标准SQL当中，这是不允许的





####第4步. HAVING

筛选分组







#### 第5步. ORDER BY

排序









####第6步. SELECT

选择需要的列







### 高级查询



####子查询



![image-20200615144614785](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200615144614785.png)





####关联子查询

内部子查询条件引用了外层主查询的结果，关联子查询的执行顺序是，先执行外部主查询，然后把关联条件值传给子查询，子查询得到结果之后，再判断主查询查出来的结果是否满足where条件需求

![image-20200615144705798](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200615144705798.png)







## InnoDB存储引擎



两个核心：```数据存储``` + ```事务```







### 数据在磁盘如何存储







### 索引在磁盘如何存储









### 内存 - 磁盘 两级存储结构







#### 内存中缓存了什么数据



![image-20200202104154326](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200202104154326.png)







### 事务



#### 对用户只暴露事务概念

把需要执行的SQL包装在事务当中，一个事务当中可能存在一个SQL，也可能存在多个SQL



#### 事务的几个关键点



##### 原子性



##### 一致性



##### 持久性



##### 并发事务隔离性（用户可选）











### 事务并发

高性能意味着支持更高的事务并发





#### 没有事务隔离时的问题





> **```丢失修改```**



两个事务并发执行数据的 Update，一个事务覆盖另外一个事务的执行结果，导致一个事务提交时，数据并非那个事务更新的数据值，而是另外一个事务Update的值。







> **```读脏数据```**











> **```不可重复读```**









> **```幻读```**







参考：[https://github.com/CyC2018/CS-Notes/blob/master/notes/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%B3%BB%E7%BB%9F%E5%8E%9F%E7%90%86.md#%E4%BA%8C%E5%B9%B6%E5%8F%91%E4%B8%80%E8%87%B4%E6%80%A7%E9%97%AE%E9%A2%98](https://github.com/CyC2018/CS-Notes/blob/master/notes/数据库系统原理.md#二并发一致性问题)





####事务隔离具体实现（依赖锁）



##### 未提交读

事务隔离性：低

事务读取最新数据，不管其他事务对当前事务有任何影响，不需要使用任何锁，并发性能最高，但是



##### 读已提交

事务隔离性：中

使用MVCC（Multi Version Concurrency Control，多版本并发控制）机制，一个事务执行select时，读取的是最新的快照信息（从undo中读取），这个快照信息只包括已提交的事务对数据库的更改，所以不会出现脏读的情况，但是由于此时select的是最新的快照（最近的undo），所以同样的查询，可能事务前后结果不同，出现这种情况也就意味着在这个事务两次select中间，有其他事务提交了对查询数据的update，



##### 可重复读

使用MVCC机制，和读已提交不同，此时的快照版本是事务开始时候的 ”定格“ 快照版本（从undo中读取），并且在整个事务执行过程中，这个快照不会发生变化，即使两个同样的select中间有其他事务提交了对select相关数据的修改，当前事务也感知不到。



##### 事务串行





#### 锁的粒度

多种锁都可以实现需求，但是不同







#### 一种特殊的“锁” - MVCC

读永远不加锁，

