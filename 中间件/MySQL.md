# MySQL



## 整体架构



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/70.png" alt="img" style="zoom:67%;float:left" />





## 常用操作

### 连接MySQL Server







### 数据库操作命令

#### 1. 创建数据库

```mysql
# 使用默认编码latin1创建数据库
mysql> create database demodb;
# 使用自定义编码和校对规则创建数据库  ⬅️ 推荐使用这个
# utf8_general_ci校对规则对字符大小写不敏感
mysql> create database demodb default character set utf8 collate utf8_general_ci;
```

#### 2. 选择数据库

```mysql
mysql> use demodb;
```

#### 3. 查看数据库字符编码

```mysql
mysql> show create database db_name;
```

#### 4. 删除数据库

```mysql
mysql> drop database db_name;
```



### 表操作命令

#### 1. 创建表

创建表的语法：

```mysql
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    (create_definition,...)
    [table_options]
    [partition_options]

create_definition:
    col_name column_definition
  | {INDEX|KEY} [index_name] [index_type] (key_part,...)
      [index_option] ...
  | {FULLTEXT|SPATIAL} [INDEX|KEY] [index_name] (key_part,...)
      [index_option] ...
  | [CONSTRAINT [symbol]] PRIMARY KEY
      [index_type] (key_part,...)
      [index_option] ...
  | [CONSTRAINT [symbol]] UNIQUE [INDEX|KEY]
      [index_name] [index_type] (key_part,...)
      [index_option] ...
  | [CONSTRAINT [symbol]] FOREIGN KEY
      [index_name] (col_name,...)
      reference_definition
  | CHECK (expr)

column_definition:
    data_type [NOT NULL | NULL] [DEFAULT default_value]
      [AUTO_INCREMENT] [UNIQUE [KEY]] [[PRIMARY] KEY]
      [COMMENT 'string']
      [COLLATE collation_name]
      [COLUMN_FORMAT {FIXED|DYNAMIC|DEFAULT}]
      [STORAGE {DISK|MEMORY}]
      [reference_definition]
  | data_type
      [COLLATE collation_name]
      [GENERATED ALWAYS] AS (expr)
      [VIRTUAL | STORED] [NOT NULL | NULL]
      [UNIQUE [KEY]] [[PRIMARY] KEY]
      [COMMENT 'string']
      [reference_definition]
 
 table_options:
    table_option [[,] table_option] ...

table_option:
    AUTO_INCREMENT [=] value
  | AVG_ROW_LENGTH [=] value
  | [DEFAULT] CHARACTER SET [=] charset_name
  | CHECKSUM [=] {0 | 1}
  | [DEFAULT] COLLATE [=] collation_name
  | COMMENT [=] 'string'
  | COMPRESSION [=] {'ZLIB'|'LZ4'|'NONE'}
  | CONNECTION [=] 'connect_string'
  | {DATA|INDEX} DIRECTORY [=] 'absolute path to directory'
  | DELAY_KEY_WRITE [=] {0 | 1}
  | ENCRYPTION [=] {'Y' | 'N'}
  | ENGINE [=] engine_name
  | INSERT_METHOD [=] { NO | FIRST | LAST }
  | KEY_BLOCK_SIZE [=] value
  | MAX_ROWS [=] value
  | MIN_ROWS [=] value
  | PACK_KEYS [=] {0 | 1 | DEFAULT}
  | PASSWORD [=] 'string'
  | ROW_FORMAT [=] {DEFAULT|DYNAMIC|FIXED|COMPRESSED|REDUNDANT|COMPACT}
  | STATS_AUTO_RECALC [=] {DEFAULT|0|1}
  | STATS_PERSISTENT [=] {DEFAULT|0|1}
  | STATS_SAMPLE_PAGES [=] value
  | TABLESPACE tablespace_name [STORAGE {DISK|MEMORY}]
  | UNION [=] (tbl_name[,tbl_name]...)

partition_options:
    PARTITION BY
        { [LINEAR] HASH(expr)
        | [LINEAR] KEY [ALGORITHM={1|2}] (column_list)
        | RANGE{(expr) | COLUMNS(column_list)}
        | LIST{(expr) | COLUMNS(column_list)} }
    [PARTITIONS num]
    [SUBPARTITION BY
        { [LINEAR] HASH(expr)
        | [LINEAR] KEY [ALGORITHM={1|2}] (column_list) }
      [SUBPARTITIONS num]
    ]
    [(partition_definition [, partition_definition] ...)]

partition_definition:
    PARTITION partition_name
        [VALUES
            {LESS THAN {(expr | value_list) | MAXVALUE}
            |
            IN (value_list)}]
        [[STORAGE] ENGINE [=] engine_name]
        [COMMENT [=] 'string' ]
        [DATA DIRECTORY [=] 'data_dir']
        [INDEX DIRECTORY [=] 'index_dir']
        [MAX_ROWS [=] max_number_of_rows]
        [MIN_ROWS [=] min_number_of_rows]
        [TABLESPACE [=] tablespace_name]
        [(subpartition_definition [, subpartition_definition] ...)]
```





```mysql
mysql> 
create table table_name (
	colum_name colum_type [NOT NULL] [DEFAULT default_value] [comment 'xxxx'],
);
```









###杂记



* MySQL的字符校对规则collation

  字符有编码方式，不同的编码方式下，需要使用校对规则collation来决定两个字符串是否相等，如何排序等，比如说不区分大小写的校对规则，当你在查询like的时候，可能把大小写的都查出来，如果你不期望这种情况，就需要大小写敏感的字符校对规则。





* redo log -> ib_logfile0 & ib_logfile1

  redo log默认使用ib_logfile0和ib_logfile1两个文件存储内容，两个文件默认大小都是48MB，这是文件新建的时候就设置好它的值的，为什么一开始就创建一个这么大的文件呢，我猜想，创建一个大文件能使得操作系统一下子在磁盘上分配一大块连续的存储空间给应用程序，之后应用程序向这个文件写入数据的时候，就是顺序IO，否则，如果每次向文件写入信息都是追加，操作系统可能在磁盘不同位置分配空间给这个文件，然后文件在IO的时候就很多时候不是顺序IO，IO的性能就会有影响。但使用这种方式，应用程序必须自定义这个大文件的内容格式，让应用程序知道文件当中哪部分数据是有用的，因为创建这个文件以后，这个文件的内容都是0。