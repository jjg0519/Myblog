# 如何选择MySQL存储引擎

## 查看mysql已提供的存储引擎

```mysql
#看你的mysql现在已提供什么存储引擎:
mysql> show engines;
#看你的mysql当前默认的存储引擎:
mysql> show variables like '%storage_engine%';
+----------------------------+--------+
| Variable_name              | Value  |
+----------------------------+--------+
| default_storage_engine     | InnoDB |
| default_tmp_storage_engine | InnoDB |
| storage_engine             | InnoDB |
+----------------------------+--------+
3 rows in set (0.00 sec)
mysql> 
```

## 查看看某个表用了什么引擎

```bash
#你要看某个表用了什么引擎(在显示结果里参数engine后面的就表示该表当前用的存储引擎):
mysql> show create table 表名;
#查看表的引擎
mysql> SHOW TABLE STATUS FROM ykee_biz WHERE NAME='cst_customer'\G
*************************** 1. row ***************************
           Name: cst_customer
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 220483
 Avg_row_length: 188
    Data_length: 41517056
Max_data_length: 0
   Index_length: 77824000
      Data_free: 6291456
 Auto_increment: 946351
    Create_time: 2017-04-07 11:07:10
    Update_time: NULL
     Check_time: NULL
      Collation: utf8mb4_unicode_ci
       Checksum: NULL
 Create_options: 
        Comment: 顾客表
1 row in set (0.00 sec)

```

## 修改表的引擎

```mysql
#更改表的引擎
ALTER TABLE table_name ENGINE=innodb;  
ALTER TABLE  table_name ENGINE=myisam;  
```



## MySQL的存储引擎

完整的引擎说明还是看官方文档：[http://dev.mysql.com/doc/refman/5.6/en/storage-engines.html](http://dev.mysql.com/doc/refman/5.6/en/storage-engines.html)

这里介绍一些主要的引擎

### InnoDB存储引擎
InnoDB是MySQL的默认事务型引擎，它被设计用来处理大量的短期(short-lived)事务。除非有非常特别的原因需要使用其他的存储引擎，否则应该优先考虑InnoDB引擎。
建议使用MySQL5.5及以后的版本，因为这个版本及以后的版本的InnoDB引擎性能更好。
MySQL4.1以后的版本中，InnoDB可以将每个表的数据和索引存放在单独的文件中。这样在复制备份崩溃恢复等操作中有明显优势。可以通过在my.cnf中增加innodb_file_per_table来开启这个功能。如下：

Cnf代码

1. [mysqld]  
2. innodb_file_per_table  

InnoDB采用MVCC来支持高并发，并且实现了四个标准的隔离级别。其默认级别是REPEATABLE READ(可重复读)，并且通过间隙锁(next-key locking)策略防止幻读的出现。(事务和事务隔离级别是另一个大题目，各自网补吧)。
InnoDB是基于聚簇索引建立的，聚簇索引对主键查询有很高的性能。不过它的二级索引(secondary index，非主键索引)中必须包含主键列，所以如果主键列很大的话，其他的所有索引都会很大。因此表上的索引较多的话，主键应当尽可能的小。
InnoDB的存储格式是平台独立的，可以将数据和索引文件从Intel平台复制到Sun SPARC平台或其他平台。
InnoDB通过一些机制和工具支持真正的热备份，MySQL的其他存储引擎不支持热备份。



### MyISAM存储引擎

MyISAM提供了大量的特性，包括全文索引、压缩、空间函数(GIS)等，但MyISAM不支持事务和行级锁，有一个毫无疑问的缺陷就是崩溃后无法安全恢复。
MyISAM会将表存储在两个文件在中：数据文件和索引文件，分别是.MYD和.MYI为扩展名。
在MySQL5.0以前，只能处理4G的数据，5.0中可以处理256T的数据。
在数据不再进行修改操作时，可以对MyISAM表进行压缩，压缩后可以提高读能力，原因是减少了磁盘I/O。

### Archive引擎
Archive存储引擎只支持INSERT和SELECT操作，在MySQL5.1之前不支持索引。
Archive表适合日志和数据采集类应用。
Archive引擎支持行级锁和专用的缓存区，所以可以实现高并发的插入，但它不是一个事物型的引擎，而是一个针对高速插入和压缩做了优化的简单引擎。

### Blackhole引擎
Blackhole引擎没有实现任何存储机制，它会丢弃所有插入的数据，不做任何保存。但服务器会记录Blackhole表的日志，所以可以用于复制数据到备库，或者简单地记录到日志。但这种应用方式会碰到很多问题，因此并不推荐。

### CSV引擎

CSV引擎可以将普通的SCV文件作为MySQL的表来处理，但不支持索引。

CSV引擎可以作为一种数据交换的机制，非常有用。

### Feerated引擎
Federated引擎是访问其他MySQL服务器的一个代理，尽管该引擎看起来提供了一种很好的跨服务器的灵活性，但也经常带来问题，因此默认是禁用的。

### Memory引擎
如果需要快速地访问数据，并且这些数据不会被修改，重启以后丢失也没有关系，那么使用Memory表是非常有用。Memory表至少比MyISAM表要快一个数量级。
Memory表是表级锁，因此并发写入的性能较低。它不支持BLOB或TEXT类型的列，并且每行的长度是固定的，这可能呆滞部分内存的浪费。
临时表和Memory表不是一回事。临时表是指使用CREATE TEMPORARY TABLE语句创建的表，它可以使用任何存储引擎，只在单个连接中可见，当连接断开时，临时表也将不复存在。



### NDB集群引擎

MySQL服务器、NDB集群存储引擎，以及分布式的、share-nothing的、容灾的、高可用的NDB数据库的组合，被称为MySQL集群(MySQL Cluster)。

 

### 其他第三方或社区引擎
XtraDB：是InnoDB的一个改进版本，可以作为InnoDB的一个完美的替代产品。
TokuDB：使用了一种新的叫做分形树(Fractal Trees)的索引数据结构。
Infobright：是最有名的面向列的存储引擎。
Groonga：是一款全文索引引擎。
OQGraph：该引擎由Open Query研发，支持图操作(比如查找两点之间的最短路径)。
Q4M：该引擎在MySQL内部实现了队列操作。
SphinxSE：该引擎为Sphinx全文索引搜索服务器提供了SQL接口。

## 选择合适的引擎

大部分情况下，InnoDB都是正确的选择，可以简单地归纳为一句话“除非需要用到某些InnoDB不具备的特性，并且没有其他办法可以替代，否则都应该优先选择InnoDB引擎”。

除非万不得已，否则建议不要混合使用多种存储引擎，否则可能带来一系列负责的问题，以及一些潜在的bug和边界问题。

如果应用需要不同的存储引擎，请先考虑以下几个因素：

事务：

* 如果应用需要事务支持，那么InnoDB(或者XtraDB)是目前最稳定并且经过验证的选择。

备份：

* 如果可以定期地关闭服务器来执行备份，那么备份的因素可以忽略。反之，如果需要在线热备份，那么选择InnoDB就是基本的要求。

崩溃恢复

*  MyISAM崩溃后发生损坏的概率比InnoDB要高很多，而且恢复速度也要慢。

特有的特性

* 如果一个存储引擎拥有一些关键的特性，同时却又缺乏一些必要的特性，那么有时候不得不做折中的考虑，或者在架构设计上做一些取舍。

有些查询SQL在不同的引擎上表现不同。比较典型的是：
SELECT COUNT(*) FROM table;
对于MyISAM确实会很快，但其他的可能都不行。

 

## 应用举例
### 日志型应用
MyISAM或者Archive存储引擎对这类应用比较合适，因为他们开销低，而且插入速度非常快。
如果需要对记录的日志做分析报表，生成报表的SQL很可能会导致插入效率明显降低，这时候该怎么办？
一种解决方法，是利用MySQL内置的复制方案将数据复制一份到备库，然后在备库上执行比较消耗时间和CPU的查询。当然也可以在系统负载较低的时候执行报表查询操作，但应用在不断变化，如果依赖这个策略可能以后会导致问题。
另一种方法，在日志记录表的名字中包含年和月的信息，这样可以在已经没有插入操作的历史表上做频繁的查询操作，而不会干扰到最新的当前表上的插入操作。
###只读或者大部分情况下只读的表

有些表的数据用于编制类目或者分列清单（如工作岗位），这种应用场景是典型的读多写少的业务。如果不介意MyISAM的崩溃恢复问题，选用MyISAM引擎是合适的。（MyISAM只将数据写到内存中，然后等待操作系统定期将数据刷出到磁盘上）

###订单处理
涉及订单处理，支持事务是必要的，InnoDB是订单处理类应用的最佳选择。

###大数据量
如果数据增长到10TB以上的级别，可能需要建立数据仓库。Infobright是MySQL数据仓库最成功的方案。也有一些大数据库不适合Infobright，却可能适合TokuDB。