# MySQL执行SQL语句过程

MySQL执行SQL语句过程，简单图示如下：

![1652e56415e9a6f4](/Users/xiao/Downloads/1652e56415e9a6f4.jpg)

## 1. 查询缓存

***缓存命中返回结果***：返回一个正常查询返回的完整结果，所以，这个阶段如果缓存命中，数据库根本就不会再继续解析SQL语句，当然也不会去调用SQL引擎执行数据查询。

***缓存失效***：查询缓存会跟踪查询相关的表，如果相关表发生变化，那么查询缓存将会失效。

***缓存命中条件***：查询缓存的操作直接根据客户端发送给Server的SQL语句、查询数据库、客户端协议等等和一些其他信息生成Hash值，然后到缓存当中查询是否有对应Hash的缓存结果。所以即使客户端两次查询是相同的查询语义，但是SQL语句相差了一个空格，那么也是无法命中缓存的。

所以总的来说，想要命中MySQL的查询缓存，条件还是蛮苛刻的。

## 2. 解析SQL语句，生成执行计划

如果查询缓存没有命中，那就走正常的SQL语句执行过程，简单点来说就是：

语法和语义解析 -> 优化SQL语句，制定执行计划 -> 执行并返回结果

1. 语法和语义解析的对象是客户端传来的SQL语句，分析语法是否正确，检查表是否存在，检查列是否存在等等，然后生成解析树；
2. 优化SQL语句是MySQL尽可能优化，但是优化效果甚微。最重要的是生成执行计划，explain命令可以查看select语句的执行计划，展现的方式就是执行计划的等效SQL语句列表，执行一个执行计划，就相当于执行计划中列出的SQL语句。一个执行计划详细信息如下：

![这里写图片描述](https://img-blog.csdn.net/20170326085246494?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjQxMDczMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* id：标识符,表示执行顺序，id相同->顺序执行，id不同，先把大的执行完，再顺序执行小的，相同id顺序执行；
* select_type：查询类型，。
* table：查询的表。
* partitions：使用的哪个分区,需要结合表分区才能看到(since 5.7)
* type：查询类型,例如index索引
* possible_key：可能用到的索引,如果是多个索引以逗号隔开
* key：实际用到的索引,保存的是索引的名称,如果是多个索引以逗号隔开
* key_len：使用到索引的长度
* ref：引用索引对应的表中哪些行
* rows：显示mysql认为执行查询时必须要返回的行数
* filtered：通过过滤条件之后对比总数的百分比
* Extra：额外的信息,using file sort,using where



## 3. 根据执行计划，调用存储引擎



存储引擎接口：

https://github.com/mysql/mysql-server/blob/7d10c82196c8e45554f27c00681474a9fb86d137/sql/handler.h#L3614





