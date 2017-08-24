# MySQL5.6-PERFORMANCE_SCHEMA 说明

[TOC]



## 背景

 MySQL 5.5开始新增一个数据库：PERFORMANCE_SCHEMA，主要用于收集数据库服务器性能参数。并且库里表的存储引擎均为PERFORMANCE_SCHEMA，而用户是不能创建存储引擎为PERFORMANCE_SCHEMA的表。MySQL5.5默认是关闭的，需要手动开启，在配置文件里添加：

```mysql
[mysqld]
performance_schema=ON
```

查看是否开启：

```mysql
mysql> show variables like 'performance_schema';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| performance_schema | ON    |
+--------------------+-------+
1 row in set (0.01 sec)
mysql> 
```

从MySQL5.6开始，默认打开，本文就从MySQL5.6来说明，在数据库使用当中PERFORMANCE_SCHEMA的一些比较常用的功能。具体的信息可以查看[官方文档](http://dev.mysql.com/doc/refman/5.6/en/performance-schema.html)。

## 相关表信息

### 1 配置（setup）表

```mysql
atabase changed
mysql> show tables like '%setup%';
+----------------------------------------+
| Tables_in_performance_schema (%setup%) |
+----------------------------------------+
| setup_actors                           |
| setup_consumers                        |
| setup_instruments                      |
| setup_objects                          |
| setup_timers                           |
+----------------------------------------+
5 rows in set (0.00 sec)
```

#### setup_actors:配置用户纬度的监控，默认监控所有用户

```mysql
mysql> select * from setup_actors;
+------+------+------+
| HOST | USER | ROLE |
+------+------+------+
| %    | %    | %    |
+------+------+------+
1 row in set (0.00 sec)

mysql> 
```

#### setup_consumers:配置events的消费者类型，即收集的events写入到哪些统计表中

```mysql
mysql> select * from setup_consumers;
+--------------------------------+---------+
| NAME                           | ENABLED |
+--------------------------------+---------+
| events_stages_current          | NO      |
| events_stages_history          | NO      |
| events_stages_history_long     | NO      |
| events_statements_current      | YES     |
| events_statements_history      | NO      |
| events_statements_history_long | NO      |
| events_waits_current           | NO      |
| events_waits_history           | NO      |
| events_waits_history_long      | NO      |
| global_instrumentation         | YES     |
| thread_instrumentation         | YES     |
| statements_digest              | YES     |
+--------------------------------+---------+
12 rows in set (0.00 sec)

mysql> 
```

这里需要说明的是需要查看哪个就更新其ENABLED列为YES。如：

````mysql
mysql> update setup_consumers set ENABLED='YES' where NAME in ('events_stages_current','events_waits_current');
Query OK, 2 rows affected (0.00 sec)
````

更新完后立即生效，但是服务器重启之后又会变回默认值，要永久生效需要在配置文件里添加：

```mysql
[mysqld]
#performance_schema
performance_schema_consumer_events_waits_current=on
performance_schema_consumer_events_stages_current=on
performance_schema_consumer_events_statements_current=on
performance_schema_consumer_events_waits_history=on
performance_schema_consumer_events_stages_history=on
performance_schema_consumer_events_statements_history=on
```

即在这些表的前面加上：performance_schema_consumer_xxx。表setup_consumers里面的值有个层级关系：

```
global_instrumentation > thread_instrumentation = statements_digest > events_stages_current = events_statements_current = events_waits_current > events_stages_history = events_statements_history = events_waits_history > events_stages_history_long = events_statements_history_long = events_waits_history_long
```

只有上一层次的为YES，才会继续检查该本层为YES or NO。global_instrumentation是最高级别consumer，如果它设置为NO，则所有的consumer都会忽略。其中history和history_long存的是current表的历史记录条数，history表记录了每个线程最近等待的10个事件，而history_long表则记录了最近所有线程产生的10000个事件，这里的10和10000都是可以配置的。这三个表表结构相同，history和history_long表数据都来源于current表。长度通过控制参数：

```mysql
mysql> show variables like 'performance_schema%history%size';
+--------------------------------------------------------+-------+
| Variable_name                                          | Value |
+--------------------------------------------------------+-------+
| performance_schema_events_stages_history_long_size     | 10000 |
| performance_schema_events_stages_history_size          | 10    |
| performance_schema_events_statements_history_long_size | 10000 |
| performance_schema_events_statements_history_size      | 10    |
| performance_schema_events_waits_history_long_size      | 10000 |
| performance_schema_events_waits_history_size           | 10    |
+--------------------------------------------------------+-------+
6 rows in set (0.00 sec)

mysql> 
```

#### setup_instruments:配置具体的instrument

主要包含4大类：idle、stage/xxx、statement/xxx、wait/xxx

```mysql
mysql> select name,count(*) from setup_instruments group by LEFT(name,5);
+---------------------------------+----------+
| name                            | count(*) |
+---------------------------------+----------+
| idle                            |        1 |
| stage/sql/After create          |      108 |
| statement/sql/select            |      168 |
| wait/synch/mutex/sql/PAGE::lock |      279 |
+---------------------------------+----------+
4 rows in set (0.00 sec)

mysql> 
```

idle表示socket空闲的时间，stage类表示语句的每个执行阶段的统计，statement类统计语句维度的信息，wait类统计各种等待事件，比如IO，mutux，spin_lock,condition等。

#### setup_objects:配置监控对象

默认对mysql，performance_schema和information_schema中的表都不监控，而其它DB的所有表都监控

```mysql
mysql> select * from setup_objects;
+-------------+--------------------+-------------+---------+-------+
| OBJECT_TYPE | OBJECT_SCHEMA      | OBJECT_NAME | ENABLED | TIMED |
+-------------+--------------------+-------------+---------+-------+
| TABLE       | mysql              | %           | NO      | NO    |
| TABLE       | performance_schema | %           | NO      | NO    |
| TABLE       | information_schema | %           | NO      | NO    |
| TABLE       | %                  | %           | YES     | YES   |
+-------------+--------------------+-------------+---------+-------+
4 rows in set (0.00 sec)

mysql> 
```

#### setup_timers:配置每种类型指令的统计时间单位。

MICROSECOND表示统计单位是微妙，CYCLE表示统计单位是时钟周期，时间度量与CPU的主频有关，NANOSECOND表示统计单位是纳秒。但无论采用哪种度量单位，最终统计表中统计的时间都会装换到皮秒。（1秒＝1000000000000皮秒）

```mysql
mysql> select * from setup_timers;
+-----------+-------------+
| NAME      | TIMER_NAME  |
+-----------+-------------+
| idle      | MICROSECOND |
| wait      | CYCLE       |
| stage     | NANOSECOND  |
| statement | NANOSECOND  |
+-----------+-------------+
4 rows in set (0.00 sec)

mysql>
```

### 2 instance表

#### **cond_instances**：条件等待对象实例

表中记录了系统中使用的条件变量的对象，**OBJECT_INSTANCE_BEGIN**为对象的内存地址。

#### **file_instances**：文件实例

表中记录了系统中打开了文件的对象，包括ibdata文件，redo文件，binlog文件，用户的表文件等，**open_count**显示当前文件打开的数目，如果重来没有打开过，不会出现在表中。

```mysql
mysql> select * from file_instances limit 2,5;
+-----------------------------------+--------------------------------------+------------+
| FILE_NAME                         | EVENT_NAME                           | OPEN_COUNT |
+-----------------------------------+--------------------------------------+------------+
| /home/mysql/data/mysql/plugin.frm | wait/io/file/sql/FRM                 |          0 |
| /home/mysql/data/mysql/plugin.MYI | wait/io/file/myisam/kfile            |          0 |
| /home/mysql/data/mysql/plugin.MYD | wait/io/file/myisam/dfile            |          0 |
| /home/mysql/data/ibdata1          | wait/io/file/innodb/innodb_data_file |          2 |
| /home/mysql/data/ib_logfile0      | wait/io/file/innodb/innodb_log_file  |          2 |
+-----------------------------------+--------------------------------------+------------+
5 rows in set (0.00 sec)

mysql> 
```

#### **mutex_instances：**互斥同步对象实例

表中记录了系统中使用互斥量对象的所有记录，其中name为：wait/synch/mutex/*。**LOCKED_BY_THREAD_ID**显示哪个线程正持有mutex，若没有线程持有，则为NULL。

#### **rwlock_instances：** 读写锁同步对象实例

表中记录了系统中使用读写锁对象的所有记录，其中name为 wait/synch/rwlock/*。**WRITE_LOCKED_BY_THREAD_ID**为正在持有该对象的thread_id，若没有线程持有，则为NULL。**READ_LOCKED_BY_COUNT**为记录了同时有多少个读者持有读锁。（通过 events_waits_current 表可以知道，哪个线程在等待锁；通过rwlock_instances知道哪个线程持有锁。rwlock_instances的缺陷是，只能记录持有写锁的线程，对于读锁则无能为力）。

#### socket_instances：活跃会话对象实例

表中记录了thread_id,socket_id,ip和port，其它表可以通过thread_id与socket_instance进行关联，获取IP-PORT信息，能够与应用对接起来。
event_name主要包含3类：
wait/io/socket/sql/server_unix_socket，服务端unix监听socket
wait/io/socket/sql/server_tcpip_socket，服务端tcp监听socket
wait/io/socket/sql/client_connection，客户端socket

### 3 Wait表

#### **events_waits_current**：记录了当前线程等待的事件

#### **events_waits_history**：记录了每个线程最近等待的10个事件

#### **events_waits_history_long**：记录了最近所有线程产生的10000个事件

表结构定义如下：

````mysql
CREATE TABLE `events_waits_current` (
  `THREAD_ID` bigint(20) unsigned NOT NULL COMMENT '线程ID',
  `EVENT_ID` bigint(20) unsigned NOT NULL COMMENT '当前线程的事件ID，和THREAD_ID确定唯一',
  `END_EVENT_ID` bigint(20) unsigned DEFAULT NULL COMMENT '当事件开始时，这一列被设置为NULL。当事件结束时，再更新为当前的事件ID',
  `EVENT_NAME` varchar(128) NOT NULL COMMENT '事件名称',
  `SOURCE` varchar(64) DEFAULT NULL COMMENT '该事件产生时的源码文件',
  `TIMER_START` bigint(20) unsigned DEFAULT NULL COMMENT '事件开始时间（皮秒）',
  `TIMER_END` bigint(20) unsigned DEFAULT NULL COMMENT '事件结束结束时间（皮秒）',
  `TIMER_WAIT` bigint(20) unsigned DEFAULT NULL COMMENT '事件等待时间（皮秒）',
  `SPINS` int(10) unsigned DEFAULT NULL COMMENT '',
  `OBJECT_SCHEMA` varchar(64) DEFAULT NULL COMMENT '库名',
  `OBJECT_NAME` varchar(512) DEFAULT NULL COMMENT '文件名、表名、IP:SOCK值',
  `OBJECT_TYPE` varchar(64) DEFAULT NULL COMMENT 'FILE、TABLE、TEMPORARY TABLE',
  `INDEX_NAME` varchar(64) DEFAULT NULL COMMENT '索引名',
  `OBJECT_INSTANCE_BEGIN` bigint(20) unsigned NOT NULL COMMENT '内存地址',
  `NESTING_EVENT_ID` bigint(20) unsigned DEFAULT NULL COMMENT '该事件对应的父事件ID',
  `NESTING_EVENT_TYPE` enum('STATEMENT','STAGE','WAIT') DEFAULT NULL COMMENT '父事件类型(STATEMENT, STAGE, WAIT)',
  `OPERATION` varchar(32) NOT NULL COMMENT '操作类型（lock, read, write）',
  `NUMBER_OF_BYTES` bigint(20) DEFAULT NULL COMMENT '',
  `FLAGS` int(10) unsigned DEFAULT NULL COMMENT '标记'
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8
````

### 4 Stage 表 

#### **events_stages_current**：记录了当前线程所处的执行阶段

#### **events_stages_history**：记录了当前线程所处的执行阶段10条历史记录

#### **events_stages_history_long**：记录了当前线程所处的执行阶段10000条历史记录

表结构定义如下：

```mysql
CREATE TABLE `events_stages_current` (
  `THREAD_ID` bigint(20) unsigned NOT NULL COMMENT '线程ID',
  `EVENT_ID` bigint(20) unsigned NOT NULL COMMENT '事件ID',
  `END_EVENT_ID` bigint(20) unsigned DEFAULT NULL COMMENT '结束事件ID',
  `EVENT_NAME` varchar(128) NOT NULL COMMENT '事件名称',
  `SOURCE` varchar(64) DEFAULT NULL COMMENT '源码位置',
  `TIMER_START` bigint(20) unsigned DEFAULT NULL COMMENT '事件开始时间（皮秒）',
  `TIMER_END` bigint(20) unsigned DEFAULT NULL COMMENT '事件结束结束时间（皮秒）',
  `TIMER_WAIT` bigint(20) unsigned DEFAULT NULL COMMENT '事件等待时间（皮秒）',
  `NESTING_EVENT_ID` bigint(20) unsigned DEFAULT NULL COMMENT '该事件对应的父事件ID',
  `NESTING_EVENT_TYPE` enum('STATEMENT','STAGE','WAIT') DEFAULT NULL COMMENT '父事件类型(STATEMENT, STAGE, WAIT)'
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8
```

### 5 Statement 表

#### events_statements_current:通过 thread_id+event_id可以唯一确定一条记录。

Statments表只记录最顶层的请求，SQL语句或是COMMAND，每条语句一行。event_name形式为statement/sql/*，或statement/com/*

#### events_statements_history

#### events_statements_history_long

表结构定义如下：

```mysql
CREATE TABLE `events_statements_current` (
  `THREAD_ID` bigint(20) unsigned NOT NULL COMMENT '线程ID',
  `EVENT_ID` bigint(20) unsigned NOT NULL COMMENT '事件ID',
  `END_EVENT_ID` bigint(20) unsigned DEFAULT NULL COMMENT '结束事件ID',
  `EVENT_NAME` varchar(128) NOT NULL COMMENT '事件名称',
  `SOURCE` varchar(64) DEFAULT NULL COMMENT '源码位置',
  `TIMER_START` bigint(20) unsigned DEFAULT NULL COMMENT '事件开始时间（皮秒）',
  `TIMER_END` bigint(20) unsigned DEFAULT NULL COMMENT '事件结束结束时间（皮秒）',
  `TIMER_WAIT` bigint(20) unsigned DEFAULT NULL COMMENT '事件等待时间（皮秒）',
  `LOCK_TIME` bigint(20) unsigned NOT NULL COMMENT '锁时间',
  `SQL_TEXT` longtext COMMENT '记录SQL语句',
  `DIGEST` varchar(32) DEFAULT NULL COMMENT '对SQL_TEXT做MD5产生的32位字符串',
  `DIGEST_TEXT` longtext COMMENT '将语句中值部分用问号代替，用于SQL语句归类',
  `CURRENT_SCHEMA` varchar(64) DEFAULT NULL COMMENT '默认的数据库名',
  `OBJECT_TYPE` varchar(64) DEFAULT NULL COMMENT '保留字段',
  `OBJECT_SCHEMA` varchar(64) DEFAULT NULL COMMENT '保留字段',
  `OBJECT_NAME` varchar(64) DEFAULT NULL COMMENT '保留字段',
  `OBJECT_INSTANCE_BEGIN` bigint(20) unsigned DEFAULT NULL COMMENT '内存地址',
  `MYSQL_ERRNO` int(11) DEFAULT NULL COMMENT '',
  `RETURNED_SQLSTATE` varchar(5) DEFAULT NULL COMMENT '',
  `MESSAGE_TEXT` varchar(128) DEFAULT NULL COMMENT '信息',
  `ERRORS` bigint(20) unsigned NOT NULL COMMENT '错误数目',
  `WARNINGS` bigint(20) unsigned NOT NULL COMMENT '警告数目',
  `ROWS_AFFECTED` bigint(20) unsigned NOT NULL COMMENT '影响的数目',
  `ROWS_SENT` bigint(20) unsigned NOT NULL COMMENT '返回的记录数',
  `ROWS_EXAMINED` bigint(20) unsigned NOT NULL COMMENT '读取扫描的记录数目',
  `CREATED_TMP_DISK_TABLES` bigint(20) unsigned NOT NULL COMMENT '创建磁盘临时表数目',
  `CREATED_TMP_TABLES` bigint(20) unsigned NOT NULL COMMENT '创建临时表数目',
  `SELECT_FULL_JOIN` bigint(20) unsigned NOT NULL COMMENT 'join时，第一个表为全表扫描的数目',
  `SELECT_FULL_RANGE_JOIN` bigint(20) unsigned NOT NULL COMMENT '引用表采用range方式扫描的数目',
  `SELECT_RANGE` bigint(20) unsigned NOT NULL COMMENT 'join时，第一个表采用range方式扫描的数目',
  `SELECT_RANGE_CHECK` bigint(20) unsigned NOT NULL COMMENT '',
  `SELECT_SCAN` bigint(20) unsigned NOT NULL COMMENT 'join时，第一个表位全表扫描的数目',
  `SORT_MERGE_PASSES` bigint(20) unsigned NOT NULL COMMENT '',
  `SORT_RANGE` bigint(20) unsigned NOT NULL COMMENT '范围排序数目',
  `SORT_ROWS` bigint(20) unsigned NOT NULL COMMENT '排序的记录数目',
  `SORT_SCAN` bigint(20) unsigned NOT NULL COMMENT '全表排序数目',
  `NO_INDEX_USED` bigint(20) unsigned NOT NULL COMMENT '没有使用索引数目',
  `NO_GOOD_INDEX_USED` bigint(20) unsigned NOT NULL COMMENT '',
  `NESTING_EVENT_ID` bigint(20) unsigned DEFAULT NULL COMMENT '该事件对应的父事件ID',
  `NESTING_EVENT_TYPE` enum('STATEMENT','STAGE','WAIT') DEFAULT NULL COMMENT '父事件类型(STATEMENT, STAGE, WAIT)'
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8
```

### 6 Connection 表

#### **users**：记录用户连接数信息

```mysql
mysql> select * from users;
+--------------+---------------------+-------------------+
| USER         | CURRENT_CONNECTIONS | TOTAL_CONNECTIONS |
+--------------+---------------------+-------------------+
| root         |                   7 |             23811 |
| ykee_quartz  |                   4 |               531 |
| ykee_weixin  |                   0 |                10 |
| NULL         |                  18 |                26 |
| ykee_xsimple |                 597 |           2524073 |
| ykee_bid     |                   7 |              2509 |
| ykee_biz     |                  13 |              1169 |
| ykee_base    |                   2 |             25805 |
| common_read  |                   8 |             43528 |
| shj_order    |                   1 |              1084 |
| rep          |                   1 |                 2 |
| ykee_sys     |                   6 |               157 |
| ykee_pm      |                  35 |             10718 |
| ykee_im      |                  11 |               869 |
+--------------+---------------------+-------------------+
14 rows in set (0.00 sec)

mysql> 
```

#### hosts：记录了主机连接数信息

```mysql
mysql> select * from hosts;
+----------------+---------------------+-------------------+
| HOST           | CURRENT_CONNECTIONS | TOTAL_CONNECTIONS |
+----------------+---------------------+-------------------+
| NULL           |                  18 |                26 |
| 192.168.20.224 |                   2 |               787 |
| 192.168.20.143 |                   5 |               878 |
| localhost      |                   1 |             22700 |
| 192.168.1.128  |                 574 |           2498656 |
| 192.168.1.23   |                   5 |             21368 |
| 192.168.40.66  |                   0 |                 4 |
| 192.168.20.106 |                   5 |               460 |
| 192.168.20.210 |                   6 |              1140 |
| 192.168.20.93  |                   0 |               149 |
| 192.168.20.87  |                   0 |             20323 |
| 192.168.1.231  |                   1 |             26061 |
| 192.168.1.124  |                   1 |              1881 |
| 192.168.20.69  |                   0 |               205 |
| 192.168.1.126  |                  32 |             15657 |
| 192.168.20.62  |                   0 |               124 |
| 192.168.20.79  |                   1 |               142 |
| 192.168.20.127 |                   0 |               351 |
| 192.168.20.74  |                   0 |               165 |
| 192.168.1.125  |                  38 |             15459 |
| 192.168.20.217 |                  16 |               643 |
| 192.168.20.40  |                   0 |                45 |
| 192.168.1.122  |                   1 |                 2 |
| 192.168.20.251 |                   0 |                92 |
| 192.168.40.226 |                   0 |                 8 |
| 192.168.20.118 |                   0 |              4988 |
| 192.168.1.123  |                   2 |              1559 |
| 192.168.1.42   |                   2 |               133 |
| 192.168.20.105 |                   0 |               310 |
+----------------+---------------------+-------------------+
29 rows in set (0.00 sec)

mysql>
```



#### **accounts**：记录了用户主机连接数信息

```mysql
mysql> select * from accounts;
+--------------+----------------+---------------------+-------------------+
| USER         | HOST           | CURRENT_CONNECTIONS | TOTAL_CONNECTIONS |
+--------------+----------------+---------------------+-------------------+
| common_read  | 192.168.20.143 |                   1 |               433 |
| ykee_biz     | 192.168.20.210 |                   0 |                24 |
| ykee_bid     | 192.168.1.126  |                   4 |               364 |
| shj_order    | 192.168.1.123  |                   1 |              1084 |
| rep          | 192.168.1.122  |                   1 |                 2 |
| ykee_pm      | 192.168.1.123  |                   0 |                26 |
| ykee_xsimple | 192.168.20.74  |                   0 |                 6 |
| ykee_bid     | 192.168.20.210 |                   0 |                 5 |
| ykee_xsimple | 192.168.20.224 |                   0 |                 5 |
| ykee_base    | 192.168.1.125  |                   1 |               265 |
| ykee_biz     | 192.168.1.231  |                   0 |                98 |
| common_read  | 192.168.1.123  |                   0 |               247 |
| ykee_pm      | 192.168.20.127 |                   0 |                 1 |
| ykee_xsimple | 192.168.20.79  |                   0 |                29 |
| ykee_pm      | 192.168.20.105 |                   0 |               249 |
| ykee_bid     | 192.168.20.224 |                   0 |               703 |
| ykee_bid     | 192.168.20.105 |                   0 |                37 |
```

### 7 Summary 表

 Summary表聚集了各个维度的统计信息包括表维度，索引维度，会话维度，语句维度和锁维度的统计信息

#### **events_waits_summary_global_by_event_name**：按等待事件类型聚合，每个事件一条记录

```mysql
CREATE TABLE `events_waits_summary_global_by_event_name` (
  `EVENT_NAME` varchar(128) NOT NULL COMMENT '事件名称',
  `COUNT_STAR` bigint(20) unsigned NOT NULL COMMENT '事件计数',
  `SUM_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '总的等待时间',
  `MIN_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最小等待时间',
  `AVG_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '平均等待时间',
  `MAX_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最大等待时间'
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8
```

#### **events_waits_summary_by_instance**：按等待事件对象聚合

同一种等待事件，可能有多个实例，每个实例有不同的内存地址，因此event_name+object_instance_begin唯一确定一条记录。

```mysql
CREATE TABLE `events_waits_summary_by_instance` (
  `EVENT_NAME` varchar(128) NOT NULL COMMENT '事件名称',
  `OBJECT_INSTANCE_BEGIN` bigint(20) unsigned NOT NULL COMMENT '内存地址',
  `COUNT_STAR` bigint(20) unsigned NOT NULL COMMENT '事件计数',
  `SUM_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '总的等待时间',
  `MIN_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最小等待时间',
  `AVG_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '平均等待时间',
  `MAX_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最大等待时间'
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8
```

#### **events_waits_summary_by_thread_by_event_name**：按每个线程和事件来统计，thread_id+event_name唯一确定一条记录。

```mysql
CREATE TABLE `events_waits_summary_by_thread_by_event_name` (
  `THREAD_ID` bigint(20) unsigned NOT NULL COMMENT '线程ID',
  `EVENT_NAME` varchar(128) NOT NULL COMMENT '事件名称',
  `COUNT_STAR` bigint(20) unsigned NOT NULL COMMENT '事件计数',
  `SUM_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '总的等待时间',
  `MIN_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最小等待时间',
  `AVG_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '平均等待时间',
  `MAX_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最大等待时间'
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8
```

#### **events_stages_summary_global_by_event_name**：按事件阶段类型聚合，每个事件一条记录，表结构同上。

#### **events_stages_summary_by_thread_by_event_name**：按每个线程和事件来阶段统计，表结构同上。

#### **events_statements_summary_by_digest**：按照事件的语句进行聚合。

```mysql
CREATE TABLE `events_statements_summary_by_digest` (
  `SCHEMA_NAME` varchar(64) DEFAULT NULL COMMENT '库名',
  `DIGEST` varchar(32) DEFAULT NULL COMMENT '对SQL_TEXT做MD5产生的32位字符串。如果为consumer表中没有打开statement_digest选项，则为NULL',
  `DIGEST_TEXT` longtext COMMENT '将语句中值部分用问号代替，用于SQL语句归类。如果为consumer表中没有打开statement_digest选项，则为NULL。',
  `COUNT_STAR` bigint(20) unsigned NOT NULL COMMENT '事件计数',
  `SUM_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '总的等待时间',
  `MIN_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最小等待时间',
  `AVG_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '平均等待时间',
  `MAX_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最大等待时间',
  `SUM_LOCK_TIME` bigint(20) unsigned NOT NULL COMMENT '锁时间总时长',
  `SUM_ERRORS` bigint(20) unsigned NOT NULL COMMENT '错误数的总',
  `SUM_WARNINGS` bigint(20) unsigned NOT NULL COMMENT '警告的总数',
  `SUM_ROWS_AFFECTED` bigint(20) unsigned NOT NULL COMMENT '影响的总数目',
  `SUM_ROWS_SENT` bigint(20) unsigned NOT NULL COMMENT '返回总数目',
  `SUM_ROWS_EXAMINED` bigint(20) unsigned NOT NULL COMMENT '总的扫描的数目',
  `SUM_CREATED_TMP_DISK_TABLES` bigint(20) unsigned NOT NULL COMMENT '创建磁盘临时表的总数目',
  `SUM_CREATED_TMP_TABLES` bigint(20) unsigned NOT NULL COMMENT '创建临时表的总数目',
  `SUM_SELECT_FULL_JOIN` bigint(20) unsigned NOT NULL COMMENT '第一个表全表扫描的总数目',
  `SUM_SELECT_FULL_RANGE_JOIN` bigint(20) unsigned NOT NULL COMMENT '总的采用range方式扫描的数目',
  `SUM_SELECT_RANGE` bigint(20) unsigned NOT NULL COMMENT '第一个表采用range方式扫描的总数目',
  `SUM_SELECT_RANGE_CHECK` bigint(20) unsigned NOT NULL COMMENT '',
  `SUM_SELECT_SCAN` bigint(20) unsigned NOT NULL COMMENT '第一个表位全表扫描的总数目',
  `SUM_SORT_MERGE_PASSES` bigint(20) unsigned NOT NULL COMMENT '',
  `SUM_SORT_RANGE` bigint(20) unsigned NOT NULL COMMENT '范围排序总数',
  `SUM_SORT_ROWS` bigint(20) unsigned NOT NULL COMMENT '排序的记录总数目',
  `SUM_SORT_SCAN` bigint(20) unsigned NOT NULL COMMENT '第一个表排序扫描总数目',
  `SUM_NO_INDEX_USED` bigint(20) unsigned NOT NULL COMMENT '没有使用索引总数',
  `SUM_NO_GOOD_INDEX_USED` bigint(20) unsigned NOT NULL COMMENT '',
  `FIRST_SEEN` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '第一次执行时间',
  `LAST_SEEN` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '最后一次执行时间'
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8
```

#### **events_statements_summary_global_by_event_name**：按照事件的语句进行聚合。表结构同上。

#### **events_statements_summary_by_thread_by_event_name**：按照线程和事件的语句进行聚合，表结构同上。

#### **file_summary_by_instance**：按事件类型统计（**物理IO维度**）

#### **file_summary_by_event_name**：具体文件统计（**物理IO维度**）

9和10一起说明：

统计IO操作：COUNT_STAR，SUM_TIMER_WAIT,MIN_TIMER_WAIT,AVG_TIMER_WAIT,MAX_TIMER_WAIT

统计读      ：COUNT_READ,SUM_TIMER_READ,MIN_TIMER_READ,AVG_TIMER_READ,MAX_TIMER_READ, SUM_NUMBER_OF_BYTES_READ

统计写      ：COUNT_WRITE,SUM_TIMER_WRITE,MIN_TIMER_WRITE,AVG_TIMER_WRITE,MAX_TIMER_WRITE, SUM_NUMBER_OF_BYTES_WRITE

统计其他IO事件，比如create，delete，open，close等：COUNT_MISC,SUM_TIMER_MISC,MIN_TIMER_MISC,AVG_TIMER_MISC,MAX_TIMER_MISC

#### **table_io_waits_summary_by_table**：根据wait/io/table/sql/handler，聚合每个表的I/O操作（**逻辑IO纬度**）

统计IO操作：COUNT_STAR,SUM_TIMER_WAIT,MIN_TIMER_WAIT,AVG_TIMER_WAIT,MAX_TIMER_WAIT 

统计读      ：COUNT_READ,SUM_TIMER_READ,MIN_TIMER_READ,AVG_TIMER_READ,MAX_TIMER_READ

​              ：COUNT_FETCH,SUM_TIMER_FETCH,MIN_TIMER_FETCH,AVG_TIMER_FETCH, MAX_TIMER_FETCH

统计写      ：COUNT_WRITE,SUM_TIMER_WRITE,MIN_TIMER_WRITE,AVG_TIMER_WRITE,MAX_TIMER_WRITE

INSERT统计，相应的还有DELETE和UPDATE统计：COUNT_INSERT,SUM_TIMER_INSERT,MIN_TIMER_INSERT,AVG_TIMER_INSERT,MAX_TIMER_INSERT

#### **table_io_waits_summary_by_index_usage：**与table_io_waits_summary_by_table类似，按索引维度统计

#### **table_lock_waits_summary_by_table**：聚合了表锁等待事件，包括internal lock 和 external lock

internal lock通过SQL层函数thr_lock调用，OPERATION值为：read normal、read with shared locks、read high priority、read no insert、write allow write、write concurrent insert、write delayed、write low priority、write normal
external lock则通过接口函数handler::external_lock调用存储引擎层，OPERATION列的值为：read external、write external

#### **Connection Summaries表**：account、user、host

events_waits_summary_by_account_by_event_name
events_waits_summary_by_user_by_event_name
events_waits_summary_by_host_by_event_name 
events_stages_summary_by_account_by_event_name
events_stages_summary_by_user_by_event_name
events_stages_summary_by_host_by_event_name 
events_statements_summary_by_account_by_event_name
events_statements_summary_by_user_by_event_name
events_statements_summary_by_host_by_event_name

#### **socket_summary_by_instance、socket_summary_by_event_name**：socket聚合统计表。

### 8 其他相关表

#### **performance_timers**：系统支持的统计时间单位

#### **threads**：监视服务端的当前运行的线程

## 统计应用

**关于SQL维度的统计信息主要集中在events_statements_summary_by_digest表中，通过将SQL语句抽象出digest，可以统计某类SQL语句在各个维度的统计信息**

### 哪个SQL执行最多

```mysql
mysql>SELECT SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,SUM_ROWS_SENT,SUM_ROWS_EXAMINED,FIRST_SEEN,LAST_SEEN FROM events_statements_summary_by_digest ORDER BY COUNT_STAR desc LIMIT 1\G
*************************** 1. row ***************************
      SCHEMA_NAME: ykee_pm
      DIGEST_TEXT: SELECT `id` , `work_cost_id` , `work_type` , `work_cost_detail_code` , `work_person` , `contract_amount` , `increase_decrease_amount` , `already_payment_amount` , `reward_amount` , `settlement_amount` , `payment_amount` , `review_amount` , `ebs_flag` , `created_by` , `created_date` , `last_updated_by` , `last_updated_date` , `remove_flag` , `version` FROM `pm_project_work_cost_detail` WHERE `work_person` = ? AND `work_cost_id` = ? AND `remove_flag` = ? AND LEFT ( `work_cost_detail_code` , ? ) = ? 
       COUNT_STAR: 392339841
    SUM_ROWS_SENT: 56959
SUM_ROWS_EXAMINED: 921271652
       FIRST_SEEN: 2017-06-12 10:41:00
        LAST_SEEN: 2017-07-05 10:51:28
1 row in set (0.04 sec)

mysql> 
```

各个字段的注释可以看上面的表结构说明：从6月12号到7月5号该SQL执行了392339841次。

### 哪个SQL平均响应时间最多

```mysql

mysql> SELECT SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,AVG_TIMER_WAIT,SUM_ROWS_SENT,SUM_ROWS_EXAMINED,FIRST_SEEN,LAST_SEEN FROM events_statements_summary_by_digest ORDER BY AVG_TIMER_WAIT desc LIMIT 1\G
*************************** 1. row ***************************
      SCHEMA_NAME: ykee_pm
      DIGEST_TEXT: SELECT `id` , `workflow_key` , `project_id` , `run_id` , `account` , `is_end` , `version` FROM `pm_workflow` WHERE `workflow_key` = ? OR `workflow_key` = ? 
       COUNT_STAR: 41
   AVG_TIMER_WAIT: 173125491008000
    SUM_ROWS_SENT: 3150557
SUM_ROWS_EXAMINED: 15200733
       FIRST_SEEN: 2017-06-13 10:19:03
        LAST_SEEN: 2017-07-04 14:21:47
1 row in set (0.03 sec)
```

各个字段的注释可以看上面的表结构说明：从6月13号到7月4号该SQL平均响应时间173 1254 9100 8000皮秒（1 0000 0000 0000皮秒=1秒）

### 哪个SQL扫描的行数最多

SUM_ROWS_EXAMINED

### 哪个SQL使用的临时表最多

SUM_CREATED_TMP_DISK_TABLES、SUM_CREATED_TMP_TABLES

### 哪个SQL返回的结果集最多

SUM_ROWS_SENT

### 哪个SQL排序数最多

SUM_SORT_ROWS

通过上述指标我们可以间接获得某类SQL的逻辑IO(SUM_ROWS_EXAMINED)，CPU消耗(SUM_SORT_ROWS)，网络带宽(SUM_ROWS_SENT)的对比。

通过file_summary_by_instance表，可以获得系统运行到现在，哪个文件(表)物理IO最多，这可能意味着这个表经常需要访问磁盘IO。

### 哪个表、文件逻辑IO最多（热数据）

````mysql
mysql> SELECT FILE_NAME,EVENT_NAME,COUNT_READ,SUM_NUMBER_OF_BYTES_READ,COUNT_WRITE,SUM_NUMBER_OF_BYTES_WRITE FROM file_summary_by_instance ORDER BY SUM_NUMBER_OF_BYTES_READ+SUM_NUMBER_OF_BYTES_WRITE DESC LIMIT 2\G
*************************** 1. row ***************************
                FILE_NAME: /home/mysql/data/ibdata1
               EVENT_NAME: wait/io/file/innodb/innodb_data_file
               COUNT_READ: 651
 SUM_NUMBER_OF_BYTES_READ: 12730368
              COUNT_WRITE: 24974857
SUM_NUMBER_OF_BYTES_WRITE: 843161878528
*************************** 2. row ***************************
                FILE_NAME: /home/mysql/data/ykee_xsimple/kn_device_info.ibd
               EVENT_NAME: wait/io/file/innodb/innodb_data_file
               COUNT_READ: 1698
 SUM_NUMBER_OF_BYTES_READ: 27820032
              COUNT_WRITE: 5432986
SUM_NUMBER_OF_BYTES_WRITE: 89018171392
2 rows in set (0.00 sec)

mysql> 
````

### 哪个索引使用最多

```mysql
mysql> SELECT OBJECT_NAME, INDEX_NAME, COUNT_FETCH, COUNT_INSERT, COUNT_UPDATE, COUNT_DELETE FROM table_io_waits_summary_by_index_usage ORDER BY SUM_TIMER_WAIT DESC limit 1;
+-----------------+------------------------+--------------+--------------+--------------+--------------+
| OBJECT_NAME     | INDEX_NAME             | COUNT_FETCH  | COUNT_INSERT | COUNT_UPDATE | COUNT_DELETE |
+-----------------+------------------------+--------------+--------------+--------------+--------------+
| rl_group_record | idx_rl_group_record_n2 | 328858127697 |            0 |            0 |            0 |
+-----------------+------------------------+--------------+--------------+--------------+--------------+
1 row in set (0.00 sec)

mysql> 
```

通过**table_io_waits_summary_by_index_usage**表，可以获得系统运行到现在，哪个表的具体哪个索引(包括主键索引，二级索引)使用最多。

### 哪个索引没有使用过

```mysql
mysql> SELECT OBJECT_SCHEMA, OBJECT_NAME, INDEX_NAME FROM table_io_waits_summary_by_index_usage WHERE INDEX_NAME IS NOT NULL AND COUNT_STAR = 0 AND OBJECT_SCHEMA <> 'mysql' ORDER BY OBJECT_SCHEMA,OBJECT_NAME LIMIT 10;
+---------------+---------------------------+------------------------------+
| OBJECT_SCHEMA | OBJECT_NAME               | INDEX_NAME                   |
+---------------+---------------------------+------------------------------+
| accesslog     | accesslog                 | PRIMARY                      |
| shjweixindb   | knd_attached_documents    | PRIMARY                      |
| shjweixindb   | knd_regional_org_relation | PRIMARY                      |
| shjweixindb   | knd_sale_regional         | PRIMARY                      |
| shjweixindb   | knd_usage_details         | PRIMARY                      |
| shjweixindb   | knd_user_login_count      | PRIMARY                      |
| shjweixindb   | knd_user_use              | PRIMARY                      |
| shjweixindb   | kn_application_info       | PRIMARY                      |
| shjweixindb   | kn_application_setup_info | PRIMARY                      |
| shjweixindb   | kn_application_setup_info | FK_o0600djvu5ij2hkgtceeq95cg |
+---------------+---------------------------+------------------------------+
10 rows in set (0.01 sec)

mysql> 
```

### 哪个等待事件消耗的时间最多

```mysql
mysql> SELECT EVENT_NAME, COUNT_STAR, SUM_TIMER_WAIT, AVG_TIMER_WAIT FROM events_waits_summary_global_by_event_name WHERE event_name != 'idle' ORDER BY SUM_TIMER_WAIT DESC LIMIT 1;
+---------------------------+---------------+---------------------+----------------+
| EVENT_NAME                | COUNT_STAR    | SUM_TIMER_WAIT      | AVG_TIMER_WAIT |
+---------------------------+---------------+---------------------+----------------+
| wait/io/table/sql/handler | 2532911193763 | 2471098341675747992 |         975422 |
+---------------------------+---------------+---------------------+----------------+
1 row in set (0.01 sec)

mysql> 
```

### 类似profiling功能

分析具体某条SQL，该SQL在执行各个阶段的时间消耗，通过events_statements_xxx表和events_stages_xxx表，就可以达到目的。两个表通过event_id与nesting_event_id关联，stages表的nesting_event_id为对应statements表的event_id；针对每个stage可能出现的锁等待，一个stage会对应一个或多个wait，通过stage_xxx表的event_id字段与waits_xxx表的nesting_event_id进行关联。如：

```mysql
比如分析包含count(*)的某条SQL语句，具体如下：

SELECT
EVENT_ID,
sql_text
FROM events_statements_history
WHERE sql_text LIKE '%count(*)%';
+----------+--------------------------------------+
| EVENT_ID | sql_text |
+----------+--------------------------------------+
| 1690 | select count(*) from chuck.test_slow |
+----------+--------------------------------------+
首先得到了语句的event_id为1690，通过查找events_stages_xxx中nesting_event_id为1690的记录，可以达到目的。

a.查看每个阶段的时间消耗：
SELECT
event_id,
EVENT_NAME,
SOURCE,
TIMER_END - TIMER_START
FROM events_stages_history_long
WHERE NESTING_EVENT_ID = 1690;
+----------+--------------------------------+----------------------+-----------------------+
| event_id | EVENT_NAME | SOURCE | TIMER_END-TIMER_START |
+----------+--------------------------------+----------------------+-----------------------+
| 1691 | stage/sql/init | mysqld.cc:990 | 316945000 |
| 1693 | stage/sql/checking permissions | sql_parse.cc:5776 | 26774000 |
| 1695 | stage/sql/Opening tables | sql_base.cc:4970 | 41436934000 |
| 2638 | stage/sql/init | sql_select.cc:1050 | 85757000 |
| 2639 | stage/sql/System lock | lock.cc:303 | 40017000 |
| 2643 | stage/sql/optimizing | sql_optimizer.cc:138 | 38562000 |
| 2644 | stage/sql/statistics | sql_optimizer.cc:362 | 52845000 |
| 2645 | stage/sql/preparing | sql_optimizer.cc:485 | 53196000 |
| 2646 | stage/sql/executing | sql_executor.cc:112 | 3153000 |
| 2647 | stage/sql/Sending data | sql_executor.cc:192 | 7369072089000 |
| 4304138 | stage/sql/end | sql_select.cc:1105 | 19920000 |
| 4304139 | stage/sql/query end | sql_parse.cc:5463 | 44721000 |
| 4304145 | stage/sql/closing tables | sql_parse.cc:5524 | 61723000 |
| 4304152 | stage/sql/freeing items | sql_parse.cc:6838 | 455678000 |
| 4304155 | stage/sql/logging slow query | sql_parse.cc:2258 | 83348000 |
| 4304159 | stage/sql/cleaning up | sql_parse.cc:2163 | 4433000 |
+----------+--------------------------------+----------------------+-----------------------+
通过间接关联，我们能分析得到SQL语句在每个阶段的时间消耗，时间单位以皮秒表示。这里展示的结果很类似profiling功能，有了performance schema，就不再需要profiling这个功能了。另外需要注意的是，由于默认情况下events_stages_history表中只为每个连接记录了最近10条记录，为了确保获取所有记录，需要访问events_stages_history_long表

b.查看某个阶段的锁等待情况
针对每个stage可能出现的锁等待，一个stage会对应一个或多个wait，events_waits_history_long这个表容易爆满[默认阀值10000]。由于select count(*)需要IO(逻辑IO或者物理IO)，所以在stage/sql/Sending data阶段会有io等待的统计。通过stage_xxx表的event_id字段与waits_xxx表的nesting_event_id进行关联。
SELECT
event_id,
event_name,
source,
timer_wait,
object_name,
index_name,
operation,
nesting_event_id
FROM events_waits_history_long
WHERE nesting_event_id = 2647;
+----------+---------------------------+-----------------+------------+-------------+------------+-----------+------------------+
| event_id | event_name | source | timer_wait | object_name | index_name | operation | nesting_event_id |
+----------+---------------------------+-----------------+------------+-------------+------------+-----------+------------------+
| 190607 | wait/io/table/sql/handler | handler.cc:2842 | 1845888 | test_slow | idx_c1 | fetch | 2647 |
| 190608 | wait/io/table/sql/handler | handler.cc:2842 | 1955328 | test_slow | idx_c1 | fetch | 2647 |
| 190609 | wait/io/table/sql/handler | handler.cc:2842 | 1929792 | test_slow | idx_c1 | fetch | 2647 | 
| 190610 | wait/io/table/sql/handler | handler.cc:2842 | 1869600 | test_slow | idx_c1 | fetch | 2647 |
| 190611 | wait/io/table/sql/handler | handler.cc:2842 | 1922496 | test_slow | idx_c1 | fetch | 2647 |
+----------+---------------------------+-----------------+------------+-------------+------------+-----------+------------------+
通过上面的实验，我们知道了statement,stage,wait的三级结构，通过nesting_event_id进行关联，它表示某个事件的父event_id。

(2).模拟innodb行锁等待的例子
会话A执行语句update test_icp set y=y+1 where x=1(x为primary key)，不commit；会话B执行同样的语句update test_icp set y=y+1 where x=1，会话B堵塞，并最终报错。通过连接连接查询events_statements_history_long和events_stages_history_long，可以看到在updating阶段花了大约60s的时间。这主要因为实例上的innodb_lock_wait_timeout设置为60，等待60s后超时报错了。

SELECT
statement.EVENT_ID,
stages.event_id,
statement.sql_text,
stages.event_name,
stages.timer_wait
FROM events_statements_history_long statement 
join events_stages_history_long stages 
on statement.event_id=stages.nesting_event_id 
WHERE statement.sql_text = 'update test_icp set y=y+1 where x=1';
+----------+----------+-------------------------------------+--------------------------------+----------------+
| EVENT_ID | event_id | sql_text | event_name | timer_wait |
+----------+----------+-------------------------------------+--------------------------------+----------------+
| 5816 | 5817 | update test_icp set y=y+1 where x=1 | stage/sql/init | 195543000 |
| 5816 | 5819 | update test_icp set y=y+1 where x=1 | stage/sql/checking permissions | 22730000 |
| 5816 | 5821 | update test_icp set y=y+1 where x=1 | stage/sql/Opening tables | 66079000 |
| 5816 | 5827 | update test_icp set y=y+1 where x=1 | stage/sql/init | 89116000 |
| 5816 | 5828 | update test_icp set y=y+1 where x=1 | stage/sql/System lock | 218744000 |
| 5816 | 5832 | update test_icp set y=y+1 where x=1 | stage/sql/updating | 6001362045000 |
| 5816 | 5968 | update test_icp set y=y+1 where x=1 | stage/sql/end | 10435000 |
| 5816 | 5969 | update test_icp set y=y+1 where x=1 | stage/sql/query end | 85979000 |
| 5816 | 5983 | update test_icp set y=y+1 where x=1 | stage/sql/closing tables | 56562000 |
| 5816 | 5990 | update test_icp set y=y+1 where x=1 | stage/sql/freeing items | 83563000 |
| 5816 | 5992 | update test_icp set y=y+1 where x=1 | stage/sql/cleaning up | 4589000 |
+----------+----------+-------------------------------------+--------------------------------+----------------+
查看wait事件：
SELECT
event_id,
event_name,
source,
timer_wait,
object_name,
index_name,
operation,
nesting_event_id
FROM events_waits_history_long
WHERE nesting_event_id = 5832;
*************************** 1. row ***************************
event_id: 5832
event_name: wait/io/table/sql/handler
source: handler.cc:2782
timer_wait: 6005946156624
object_name: test_icp
index_name: PRIMARY
operation: fetch
从结果来看，waits表中记录了一个fetch等待事件，但并没有更细的innodb行锁等待事件统计。

(3).模拟MDL锁等待的例子
会话A执行一个大查询select count(*) from test_slow，会话B执行表结构变更alter table test_slow modify c2 varchar(152);通过如下语句可以得到alter语句的执行过程，重点关注“stage/sql/Waiting for table metadata lock”阶段。

SELECT
statement.EVENT_ID,
stages.event_id,
statement.sql_text,
stages.event_name as stage_name,
stages.timer_wait as stage_time
FROM events_statements_history_long statement 
left join events_stages_history_long stages 
on statement.event_id=stages.nesting_event_id
WHERE statement.sql_text = 'alter table test_slow modify c2 varchar(152)';
+-----------+-----------+----------------------------------------------+----------------------------------------------------+---------------+
| EVENT_ID | event_id | sql_text | stage_name | stage_time |
+-----------+-----------+----------------------------------------------+----------------------------------------------------+---------------+
| 326526744 | 326526745 | alter table test_slow modify c2 varchar(152) | stage/sql/init | 216662000 |
| 326526744 | 326526747 | alter table test_slow modify c2 varchar(152) | stage/sql/checking permissions | 18183000 |
| 326526744 | 326526748 | alter table test_slow modify c2 varchar(152) | stage/sql/checking permissions | 10294000 |
| 326526744 | 326526750 | alter table test_slow modify c2 varchar(152) | stage/sql/init | 4783000 |
| 326526744 | 326526751 | alter table test_slow modify c2 varchar(152) | stage/sql/Opening tables | 140172000 |
| 326526744 | 326526760 | alter table test_slow modify c2 varchar(152) | stage/sql/setup | 157643000 |
| 326526744 | 326526769 | alter table test_slow modify c2 varchar(152) | stage/sql/creating table | 8723217000 |
| 326526744 | 326526803 | alter table test_slow modify c2 varchar(152) | stage/sql/After create | 257332000 |
| 326526744 | 326526832 | alter table test_slow modify c2 varchar(152) | stage/sql/Waiting for table metadata lock | 1000181831000 |
| 326526744 | 326526835 | alter table test_slow modify c2 varchar(152) | stage/sql/After create | 33483000 |
| 326526744 | 326526838 | alter table test_slow modify c2 varchar(152) | stage/sql/Waiting for table metadata lock | 1000091810000 |
| 326526744 | 326526841 | alter table test_slow modify c2 varchar(152) | stage/sql/After create | 17187000 |
| 326526744 | 326526844 | alter table test_slow modify c2 varchar(152) | stage/sql/Waiting for table metadata lock | 1000126464000 |
| 326526744 | 326526847 | alter table test_slow modify c2 varchar(152) | stage/sql/After create | 27472000 |
| 326526744 | 326526850 | alter table test_slow modify c2 varchar(152) | stage/sql/Waiting for table metadata lock | 561996133000 |
| 326526744 | 326526853 | alter table test_slow modify c2 varchar(152) | stage/sql/After create | 124876000 |
| 326526744 | 326526877 | alter table test_slow modify c2 varchar(152) | stage/sql/System lock | 30659000 |
| 326526744 | 326526881 | alter table test_slow modify c2 varchar(152) | stage/sql/preparing for alter table | 40246000 |
| 326526744 | 326526889 | alter table test_slow modify c2 varchar(152) | stage/sql/altering table | 36628000 |
| 326526744 | 326528280 | alter table test_slow modify c2 varchar(152) | stage/sql/end | 43824000 |
| 326526744 | 326528281 | alter table test_slow modify c2 varchar(152) | stage/sql/query end | 112557000 |
| 326526744 | 326528299 | alter table test_slow modify c2 varchar(152) | stage/sql/closing tables | 27707000 |
| 326526744 | 326528305 | alter table test_slow modify c2 varchar(152) | stage/sql/freeing items | 201614000 |
| 326526744 | 326528308 | alter table test_slow modify c2 varchar(152) | stage/sql/cleaning up | 3584000 |
+-----------+-----------+----------------------------------------------+----------------------------------------------------+---------------+
从结果可以看到，出现了多次stage/sql/Waiting for table metadata lock阶段，并且间隔1s，说明每隔1s钟会重试判断。找一个该阶段的event_id,通过nesting_event_id关联，确定到底在等待哪个wait事件。
SELECT
event_id,
event_name,
source,
timer_wait,
object_name,
index_name,
operation,
nesting_event_id
FROM events_waits_history_long
WHERE nesting_event_id = 326526850;
+-----------+---------------------------------------------------+------------------+--------------+-------------+------------+------------+------------------+
| event_id | event_name | source | timer_wait | object_name | index_name | operation | nesting_event_id |
+-----------+---------------------------------------------------+------------------+--------------+-------------+------------+------------+------------------+
| 326526851 | wait/synch/cond/sql/MDL_context::COND_wait_status | mdl.cc:1327 | 562417991328 | NULL | NULL | timed_wait | 326526850 |
| 326526852 | wait/synch/mutex/mysys/my_thread_var::mutex | sql_class.h:3481 | 733248 | NULL | NULL | lock | 326526850 |
+-----------+---------------------------------------------------+------------------+--------------+-------------+------------+------------+------------------+
通过结果可以知道，产生阻塞的是条件变量MDL_context::COND_wait_status，并且显示了代码的位置。
```

## 总结

本文通过对Performance Schema数据库的介绍，主要用于收集数据库服务器性能参数：
①提供进程等待的详细信息，包括锁、互斥变量、文件信息；
②保存历史的事件汇总信息，为提供MySQL服务器性能做出详细的判断；
③对于新增和删除监控事件点都非常容易，并可以改变mysql服务器的监控周期，例如（CYCLE、MICROSECOND）。通过该库得到数据库运行的统计信息，更好分析定位问题和完善监控信息。

类似的监控还有：

```mysql
#打开标准的innodb监控：
CREATE TABLE innodb_monitor (a INT) ENGINE=INNODB;
#打开innodb的锁监控：
CREATE TABLE innodb_lock_monitor (a INT) ENGINE=INNODB;
#打开innodb表空间监控：
CREATE TABLE innodb_tablespace_monitor (a INT) ENGINE=INNODB;
#打开innodb表监控：
CREATE TABLE innodb_table_monitor (a INT) ENGINE=INNODB;
```

## 参考文章

[https://dev.mysql.com/doc/refman/5.6/en/performance-schema.html](https://dev.mysql.com/doc/refman/5.6/en/performance-schema.html)

[http://www.cnblogs.com/cchust/p/5022148.html](http://www.cnblogs.com/cchust/p/5022148.html)

[http://www.cnblogs.com/cchust/p/5057498.html](http://www.cnblogs.com/cchust/p/5057498.html)

[http://www.cnblogs.com/cchust/p/5061131.html](http://www.cnblogs.com/cchust/p/5061131.html)

[http://mysqllover.com/?p=522](http://mysqllover.com/?p=522&utm_source=tuicool&utm_medium=referral)

http://www.cnblogs.com/zhoujinyi/p/5236705.html