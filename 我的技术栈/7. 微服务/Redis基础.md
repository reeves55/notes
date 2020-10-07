# Redis基础





###Redis如何 存储和查找 键值对❓

首先，搞清楚Redis在内存当中存储键值对的数据结构是什么样子的，然后再看相应的存储和查找算法。

Redis是key-value内存数据库，核心数据结构就是字典

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316154416371.png" alt="image-20200316154416371" style="zoom:50%;float:left" />

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316154454365.png" alt="image-20200316154454365" style="zoom:50%;float:left" />



和Java当中的 ```HashMap``` 类似，底层基于数组，对key进行hash之后计算出索引值，有hash冲突的值通过链表连接在一起。





### 内存不足，过期策略

















### Redis 过期的键值对 删除策略❓

关键点：```定时删除```，```惰性删除```，```定期删除```

过期键值对的删除，其实就相当于Redis的 “垃圾回收”，针对过期键值对的删除策略有3种：

* 精确定时删除：针对每个设置过期时间的键值对，生成一个定时器，过期时间一到，就把键值对删除了，这样，Redis管理的内存几乎不会存在任何垃圾，但是维护定时器的资源消耗非常高🙅‍♀️，而且当大量键值对到期，连续进行删除时，会大量抢占CPU资源，导致正常的键值对操作受影响；
* 惰性删除：一个键值对，只有再被再次访问时，会被检测到已过期，再进行删除，这样被动式的删除最省CPU资源，但是内存当中的垃圾可能会非常多，垃圾多就意味着，redis需要resize的时候，可能找不到所需的连续内存空间，而且在set key的时候，首先要在过期的键值对当中去查找当前键是否过期，比较key的过期时间和当前系统时间，这个比较操作也是需要耗时的；
* 定期删除：定期扫描整个系统，把过期的键值对删除掉，但是不一定删除所有，可以调整一次删除的时长以及频率；

Redis本身采用 懒惰删除 + 定期删除 的策略。

定期删除具体实现：Redis有一个操作ServerCron函数（默认每100ms执行一次）会被周期性定时调用，这个函数会执行很多操作，其中一个操作就是删除过期键值对，删除过期键值对大体工作流程如下：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316153803328.png" alt="image-20200316153803328" style="zoom:44%;float:left" />



### Redis如何将内存中的数据持久化到磁盘❓

关键点：```RDB```，```AOF```，```触发时机```

Redis有两种方式将Redis的内存数据状态存储到磁盘：

* <span style="color:red">RDB文件</span>，存储redis整个数据库所有的数据状态；
* <span style="color:red">AOF（Append Only File）文件</span>，存储redis服务器执行的写命令，也就是状态转换记录；



#### RDB持久化触发条件

RDB存储可由手动输入命令来触发，也可以通过配置Redis来让redis自动触发RDB生成过程。

1. 手动输入命令  **SAVE**（阻塞Redis服务器）， **BGSAVE**（创建子进程来执行SAVE操作）

2. 通过配置 save选项，如 save 900 1 => 指的是服务器在900秒之内，对数据库做了至少一次修改，则触发RDB序列化过程，具体是通过Redis服务器周期性操作函数 **serverCron**，在执行过程中，其中一项工作就是检查 save选项所设置的保存条件是否已经满足，如果满足的话，就执行 **BGSAVE** 命令。



#### RDB文件格式





#### AOF持久化过程

持久化分为3个过程：

1. **命令追加**（append）           -> 将写命令追加到内存中 aof_buf 缓冲区中
2. **文件写入及同步（sync）**    -> 调用操作系统write函数，将 aof_buf 缓冲区的内容写入到文件，并刷新操作系统文件缓冲到磁盘



**什么时机触发AOF 文件写入及同步 过程**

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316150841413.png" alt="image-20200316150841413" style="zoom:55%;float:left;border:2px solid" />

其中，```flushAppendOnlyFile()``` 函数行为受到配置项 appendfsync 的影响：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316151131821.png" alt="image-20200316151131821" style="zoom:55%;" />





###Redis如何从磁盘恢复数据？

关键点：```AOF重写```

载入操作是自动执行的，如果有相应文件，就自动触发载入过程，载入过程属于Redis启动操作的一部分，载入不完成，Redis相当于没有启动。

####1. 通过RDB文件恢复

根据RDB文件格式解析RDB文件，读取RDB当中存储的key-value，SET进指定数据库即可。



#### 2. 通过AOF文件恢复

这种方式就稍微有些复杂了，AOF是一系列的指令，恢复过程就是执行这些指令的过程，有点类似MySQL的redo log，而Redis指令只能由客户端来执行，因此通过AOF进行恢复的时候，会自动创建一个虚拟的client，由client读取AOF文件当中的指令，然后发送给Redis服务器，来实现数据恢复。

这里有一个问题，由于每一次的写操作都会追加到AOF文件当中，AOF文件可能会非常庞大，恢复过程很漫长，所以就有了<span style="color:red">AOF重写机制</span>，AOF重写机制指的是根据当前Redis的状态，生成AOF文件，注意：AOF重写机制和已有的AOF文件没有任何关系，<span style="color:red">它只是一种生成AOF文件的方式</span>，这种方式根据Redis数据状态直接生成相应的Redis指令，模仿正统AOF内容，但是通过这种方式，多次写操作可以合并成一个批量写操作，从而减小了AOF文件大小，其实这是一个 持久化过程的操作，而不是恢复数据的操作。



### Redis集群



#### 主从复制

新旧版本主从复制方法有些不同

**旧版（Redis version < 2.8）主从复制过程：**

1. 从服务器执行 ```SLAVEOF master_host:port``` 命令，向主服务器发送  ```SYNC``` 命令；
2. 主服务器收到 ```SYNC``` 命令后执行 ```BGSAVE``` 命令，在后台生成一个RDB文件，并使用一个缓冲区记录 ```BGSAVE``` 命令之后主服务器执行的所有的写命令；
3. 当主服务器的 ```BGSAVE``` 命令执行完成，主服务器会将生成的RDB文件发送给从服务器，从服务器接收并载入这个RDB文件，将从服务器的状态同步至主服务器执行 ```BGSAVE``` 命令时的状态；
4. 主服务器将记录在缓冲区内的所有写命令发送给从服务器，从服务器执行这些写命令（这个时候相当于主服务器是从服务器的client），至此，从服务器状态和主服务器同步；
5. 主服务器接收到客户端发送的写命令，执行，并将写命令发送给从服务器；
6. 从服务器执行主服务器发送过来的写命令，如果这个时候发生了网络断开，主从数据变得不一致了，当从服务器重新连接上主服务器时，从服务器向主服务器发送 ```SYNC``` 命令，然后 调到第 3 步。



缺陷：

网络断开到重试期间，主从服务器状态不一致，但是这个不一致的变化量可能并不大，因为之前网络正常的时候，主从是一直处于同步状态的，而 ```SYNC``` 是一个极其消耗资源的操作。

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316165425462.png" alt="image-20200316165425462" style="zoom:45%;" />



**新版（Redis version >= 2.8）主从复制过程**：

1. 从服务器执行 ```SLAVEOF master_host:port``` 命令，向主服务器发送  ```SYNC``` 命令；
2. 主服务器收到 ```SYNC``` 命令后执行 ```BGSAVE``` 命令，在后台生成一个RDB文件，并使用一个缓冲区记录 ```BGSAVE``` 命令之后主服务器执行的所有的写命令；
3. 当主服务器的 ```BGSAVE``` 命令执行完成，主服务器会将生成的RDB文件发送给从服务器，从服务器接收并载入这个RDB文件，将从服务器的状态同步至主服务器执行 ```BGSAVE``` 命令时的状态；
4. 主服务器将记录在缓冲区内的所有写命令发送给从服务器，从服务器执行这些写命令（这个时候相当于主服务器是从服务器的client），至此，从服务器状态和主服务器同步；
5. 主服务器接收到客户端发送的写命令，执行，并将写命令发送给从服务器；
6. 从服务器执行主服务器发送过来的写命令，如果这个时候发生了网络断开，主从数据变得不一致了，当从服务器重新连接上主服务器时，从服务器向主服务器发送 ```PSYNC <runid> <offset>``` 命令，其中，```<runid>``` 是上次主从同步主服务器的ID，```<offset>``` 是从服务器的复制偏移量；
7. 主服务器检测 ```PSYNC``` 命令中的 ```<runid>``` 是不是自己，如果不是自己则返回 ```+FULLRESYNC <runid> <offset>``` 回复，要求从服务器和主服务器进行完全同步，如果是自己，则继续检测 ```<offset>``` 是否在主服务器的复制积压缓冲区当中，如果在的话， 向从服务器返回 ```+CONTINUNE``` 回复，表示执行部分同步，如果不在，则执行完全同步；



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316171658644.png" alt="image-20200316171658644" style="zoom:50%;float:left" />



<span style="color:red">复制积压缓冲区</span> 实际上是一个FIFO队列，默认大小为 1MB<span style="color:red">（实际应用需要根据情况配置）</span>

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316171742879.png" alt="image-20200316171742879" style="zoom:50%;" />



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316171836702.png" alt="image-20200316171836702" style="zoom:50%;float:left" />



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316172101977.png" alt="image-20200316172101977" style="zoom:50%;" />



#### 复制算法的问题

凡是采用 <span style="color:red">异步复制 </span>方式的机制都有一个共同的问题，如果在主从数据不一致，正在同步的过程中发生主节点下线的情况，从节点变成主节点后，实际上是有 <span style="color:red">数据丢失</span> 的。

这个问题在redis有，在kafka也有（kafka主从broker也是异步复制的）

解决这个问题有一个方案，就是客户端降级，指的是，如果客户端发现现在的主服务器故障，先将数据写入操作缓存在本地或者缓存在消息队列里面，每隔一段时间重试主服务器节点。



#### 哨兵（Sentinel）机制

哨兵机制的引入是为了增加redis的可用性，<span style="color:red">这里可用性指的是，一个redis主从备份系统能在主节点故障下线的情况下，重新在从节点中选出一个作为主节点，继续服务</span>。同时哨兵机制中，哨兵系统也增加了可用性，即多个哨兵节点组成了分布式的哨兵系统，这些哨兵节点监控所有的redis主节点和从节点。

⚠️ 这里需要注意的是，无论sentinel系统监控了多少redis主节点，这些主节点之前是没有什么联系的，针对一个主从系统，数据的写入永远都只是主节点来处理，和其他主从系统中的主节点没有关系。



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316173803729.png" alt="image-20200316173803729" style="zoom:40%;float:left" /> <img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316173844521.png" alt="image-20200316173844521" style="zoom:40%;float:left" />















<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200316174034024.png" alt="image-20200316174034024" style="zoom:40%;" />



#####1. 启动sentinel节点

一个sentinel节点其实是也是一个redis服务器，但是只有哨兵相关功能，它启动时，会根据配置文件找到它初识时的主服务节点列表，然后设置近sentinelState数据结构中去（左边是配置文件，右边是内存中sentinel的状态信息）：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200316232032386.png" alt="image-20200316232032386" style="zoom:40%;float:left" /> <img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200316231919761.png" alt="image-20200316231919761" style="zoom:40%;float:right" />

















#####2. 连接主从服务器并获取信息

先连接主服务器 =》获取主服务器对应的从服务器信息 =》连接从服务器 

当新的从服务器添加进来时，被sentinel检测到以后，sentinel就会建立到这个新的从服务器的网络连接。



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317000700620.png" alt="image-20200317000700620" style="zoom:50%;float:left" />  <img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317000906322.png" alt="image-20200317000906322" style="zoom:50%;float:right" />

















<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317001036155.png" alt="image-20200317001036155" style="zoom:40%;float:left" />



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317001328861.png" alt="image-20200317001328861" style="zoom:50%;float:left" />



如果有多个sentinel，sentinel之前还会建立网络连接：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317001733190.png" alt="image-20200317001733190" style="zoom:50%;float:left" />



##### 3. 心跳检测

默认情况下，sentinel会以每秒一次的频率向所有与它创建了连接的实例（包括主服务器、从服务器、其他sentinel在内）发送 ```PING``` 命令，并通过实例返回的 ```PING```命令的回复信息来判断实例是否在线。

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317003118815.png" alt="image-20200317003118815" style="zoom:45%;float:left" />

```PING```命令的回复有两种，一种是有效回复（```+PONG```，```-LOADING```，```-MASTERDOWN```），另一种是无效回复，不在有效回复范围内的都叫无效回复。



##### 4. 主服务器下线

如果某个主服务器连续一段时间（可配置，```down-after-milliseconds```）都向sentinel返回无效回复时，sentinel就认为该主服务器 主观下线，为了确认这个主服务器真的下线了，它会向同样监视这一主服务器的其他sentinel发送 ```SENTINEL is-master-down-by-addr <ip> <port> <current_epoch> <runid>``` 命令来进行询问，看它们是否也认为该主服务器已经进入到了下线状态（可以是主观下线或者客观下线）。当sentinel从其他sentinel那里接收到足够数量的已下线判断之后，sentinel就会将主服务器判定为客观下线，并对主服务器进行故障转移操作。



##### 5. 选举领头sentinel对下线服务器做故障转移操作

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317013115442.png" alt="image-20200317013115442" style="zoom:50%;float:left" />



##### 6. 故障转移

**三步走：**

1. 在已下线主服务器的所有从服务器里面，挑选出一个从服务器，并将其转换成主服务器；
2. 让已下线的主服务器属下的所有其他从服务器改为复制新的主服务器；
3. 将已下线的主服务器设置为新的主服务器的从服务器，当这个旧的主服务器重新上线时，它就会成为新的主服务器的从服务器。

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317013928079.png" alt="image-20200317013928079" style="zoom:50%;float:left" /><img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317013959651.png" alt="image-20200317013959651" style="zoom:50%;float:left" />

















<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317013534966.png" alt="image-20200317013534966" style="zoom:50%;float:left" />



#### 集群（分片）

关键点：```槽分配```，```重新分片```，```转向```，```故障转移```

集群要解决的是<span style="color:red">为redis提供分布式存储的功能</span>，即数据分别存储在集群当中的不同节点上，一方面降低节点的存储压力，另一方面，提高整体性能。

##### 1. 以集群模式启动节点

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317014927220.png" alt="image-20200317014927220" style="zoom:50%;float:left" />



##### 2. 通过 CLUSTER MEET  命令组建集群

客户端向集群中任何一个主服务器节点发送命令，都可以组建集群

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317015150020.png" alt="image-20200317015150020" style="zoom:50%;float:left" />



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317014727902.png" alt="image-20200317014727902" style="zoom:50%;float:left" />



当在一个主服务器A上添加另一个集群的主服务器节点B，两者完成握手之后，<span style="color:red">节点A会将节点B的信息通过Gossip协议传播给集群中的其他节点，让其他节点也和B进行握手</span>，最终，经过一段时间，集群内所有节点都完成了和节点B的握手，每个主服务器都会保存组建完成的集群中节点信息：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200317015104853.png" alt="image-20200317015104853" style="zoom:50%;float:left" />



##### 3. 使用命令 CLUSTER ADDSLOTS 进行 槽分配

redis定义了共16384个槽，当数据库中的16384个槽都有节点在处理（槽都分配完成了），集群才会处于上线状态，否则，集群处于下线状态。<span style="color:red">槽的分配需要我们手动在各个主服务器上输入 ```CLUSTER ADDSLOTS``` 命令</span>。

每个节点都会使用一个二进制数组记录下所有槽的分配情况，初始时只是设置了自己被分配的槽

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317091226178.png" alt="image-20200317091226178" style="zoom:45%;float:left" />



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317092643662.png" alt="image-20200317092643662" style="zoom:47%;float:left" />

在每个节点上，维护的槽分配信息如下，既有自己的，也有集群全部节点的

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317093151297.png" alt="image-20200317093151297" style="zoom:45%;float:left" />



##### 4. 传播槽分配信息

每个主服务器上设置的槽分配信息需要相互交换，这样每个主服务器才知道哪个槽归哪个主服务器管，交换过程其实就是直接把自己的 ```slots``` 数组发送给其他主服务器节点。

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317091629679.png" alt="image-20200317091629679" style="zoom:40%;float:left" /><img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317091702024.png" alt="image-20200317091702024" style="zoom:35%;float:left" /><img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317091745253.png" alt="image-20200317091745253" style="zoom:35%;float:left" />













等到主服务器之间槽分配信息交换完毕，每个主服务器上都有了现在集群中每个槽的分配信息：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317092500896.png" alt="image-20200317092500896" style="zoom:45%;" />



##### 5. 集群开始提供服务

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317093953861.png" alt="image-20200317093953861" style="zoom:50%;float:left" />



这里有两个问题，

第一个是如何判断指定的key属于哪个槽❓

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317094207974.png" alt="image-20200317094207974" style="zoom:45%;float:left" />

第二个是 ```MOVED``` 错误是什么❓

MOVED命令格式为 ```MOVED <slot> <ip>:<port>```，slot是请求的key对应的槽，ip和port指的是处理该槽的节点的地址，这样客户端就可以根据MOVED错误信息去找到真正处理该key的节点。

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317094538921.png" alt="image-20200317094538921" style="zoom:50%;float:left" />

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317094612181.png" alt="image-20200317094612181" style="zoom:50%;float:left" />



<span style="color:red">这里有一个问题：</span>在集群中有一个主服务器发生故障转移时，分配给该主服务器的槽无法被正常处理，集群应该停止对外服务，否则一个指向这个故障的主服务器的MOVED指令并不能让客户端正确执行一个命令。



##### 6. 重新分片

重新分片操作可以将任意数量已经指派给了某个节点（源节点）的槽改为指派给另一个节点（目标节点），并且和槽相关的key-value对也会从源节点移动到目标节点。重新分片操作无需集群下线，可以线上进行，而且源节点和目标节点都可以继续执行命令。

重新分片操作需要使用到redis集群专用的一个软件 ```redis-trib```，<span style="color:red">槽的重新分配需要一个槽一个槽地处理</span>，单个槽的重新分配过程如下：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317100504954.png" alt="image-20200317100504954" style="zoom:42%;float:left" /> 



















<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317095628804.png" alt="image-20200317095628804" style="zoom:45%;float:left" />



##### 7. ACK错误

重新分片过程中，因为一个槽的数据可能存在在两台不同的主服务器上，客户端请求时，可能会出现ACK错误

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317100739150.png" alt="image-20200317100739150" style="zoom:50%;float:left" />

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200317101058700.png" alt="image-20200317101058700" style="zoom:45%;float:left" />



##### 8. 客户端访问分片

由于redis集群是通过槽分派的方式来进行分片的，槽分配信息确定，key确定了，就能唯一定位一个处理这个key的主服务器，客户端无论连接到哪一个主服务器，主服务器都可以通过 MOVED 命令来让客户端知道这个key应该交由哪个主服务器处理，但是，如果客户端记住了集群的槽分配信息，就可以在客户端进行 CRC16 计算出key应该由哪个主服务器处理，直接给指定的主服务器发送命令即可。

关于客户端分片（<span style="color:red">和redis集群无关</span>）

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200318141841389.png" alt="image-20200318141841389" style="zoom:40%;float:left" />



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200318141950428.png" alt="image-20200318141950428" style="zoom:45%;float:left" />



##### 9. 缓存穿透、缓存并发和缓存雪崩

1. **缓存穿透**：客户端发送大量缓存无法命中的请求，导致请求全部直接落到数据库，给数据库造成巨大压力，压垮数据库；

   > 解决方案：正常情况下，缓存当中缓存的都是热点数据，但是如果使用系统不存在的key来请求数据的话，数据肯定直接打到数据库了，这样的请求多了，就会对数据库造成压力，针对这种情况，如果能够在缓存层判断数据是否在数据库中，就能在缓存层拦截掉不在数据库中的数据，这样，就不会把没用的流量打到数据库上了，可以使用布隆过滤器（Redis高级主题有说明），redis当中使用bitmap实现，在数据库插入新的数据时，计算出redis当中对应存储的业务key，对key使用多个hash函数计算出多个hash值，每个hash值对应的bitmap位上的值都设置成1，这样，就可以用这几个比特值为1的位置来”代表“这个key，相当于给数据库做了一个索引，新来一个key访问请求时，计算这个key是否可能存在数据库就转换成把这个key进行hash计算，看看bitmap对应位置值是否都是1就可以了，如果有一个比特位置不是1，那么这个key肯定不存在于数据库中，就可以直接返回空，而无需请求数据库。
   >
   > 
   >
   > 其实这个问题，能在网关把用户请求做一次过滤应该是必需的，在网关对请求做鉴权，参数做校验，对IP请求频率做限制，把一些恶意请求直接挡掉 

2. **缓存并发 / 缓存击穿**：热点数据过期，大量请求发现之后从数据读取并写入到缓存，导致数据库压力过大，而且这些操作都是重复的，其实只需要一个客户端从数据库读取最新值然后写入到缓存就行了；

   > 解决方案：可以使用redis的分布式锁，只让一个客户端去数据库查询数据，查询完毕，再更新key的值，最后删除分布式锁，这样，更新热点数据的操作就不会大量客户端一起做，也就不会大量流量到数据库当中去

3. **缓存雪崩**：缓存服务器重启或者同时大量key过期失效，导致流量全部打到后端数据库，造成数据库压力过大，甚至压垮数据库；

   > 解决方案：可以将key的过期时间设置为 expire_time = time_set + random_time，也就是在我们设置的过期时间基础上加一个随机时间值，这样可以尽量避免缓存集中失效





#### Redis高级主题



##### HyperLogLog



参考：

[http://www.rainybowe.com/blog/2017/07/13/%E7%A5%9E%E5%A5%87%E7%9A%84HyperLogLog%E7%AE%97%E6%B3%95/index.html](http://www.rainybowe.com/blog/2017/07/13/神奇的HyperLogLog算法/index.html)



##### 布隆过滤器

用几个比特位来”代表“一个数据的存在与否，使用多个hash函数来计算这几个”特征“比特位，这种算法的优势是可以快速判断一个元素是否不在一个集合当中，但是判断这个元素在集合当中，实际上也不一定就在，因为一个数据可能包含另外一个数据的”特征“，但是误差率比较小，实际应用中，用来计算”特征“的hash函数越多，”特征“就越具有代表性和特殊性，准确率也就越高，但是复杂度会提升。



##### GeoHash



参考：

https://www.cnblogs.com/LBSer/p/3310455.html

















