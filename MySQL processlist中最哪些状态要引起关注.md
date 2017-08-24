# MySQL processlist 的状态说明

## 常见的状态

| 状态                                       | 说明                                       | 常用解决方法                                   |
| :--------------------------------------- | :--------------------------------------- | ---------------------------------------- |
| Checking table                           | 正在检查数据表（这是自动的)                           |                                          |
| **Closing tables**                       | 正在将表中修改的数据刷新到磁盘中，同时正在关闭已经用完的表。这是一个很快的操作，如果不是这样的话，就应该确认磁盘空间是否已经满了或者磁盘是否正处于重负中。 |                                          |
| **Connect Out**                          |                                          |                                          |
| **executing**                            | 该线程已开始执行一条语句。                            |                                          |
|                                          |                                          |                                          |
| **Sending binlog event to slave**        | 已经读出来，正在发送                               |                                          |
| **Finished reading one binlog; switching to next binlog** | The thread has finished reading a binary log file and is opening the next one to send to the slave. |                                          |
| **Master has sent all binlog to slave; waiting for binlog to be updated** | 等着新的binlog产生                             |                                          |
| **Waiting to finalize termination**      | **线程停止时候一个短暂状态**                         |                                          |
| **Copying to tmp table on disk**         |                                          | 通过变量tmp_table_size和max_heap_table_size来控制内存表大小上限，如果超过上限会将数据写到磁盘上，从而会有物理磁盘的读写操作，导致影响性能。 |
| **Creating sort index**                  | 线程正在处理一个SELECT就是使用内部临时表解决。               | 建议： 创建适当的索引                              |



## processlist中哪些状态要引起关注

在processlist中，看到哪些运行状态时要引起关注，主要有下面几个：

| **状态**                                   | **建议**                                   |
| ---------------------------------------- | ---------------------------------------- |
| **copy to tmp table**                    | 执行ALTER TABLE修改表结构时**建议：**放在凌晨执行或者采用类似pt-osc工具 |
| **Copying to tmp table**                 | 拷贝数据到内存中的临时表，常见于GROUP BY操作时**建议：**创建适当的索引 |
| **Copying to tmp table on disk**         | 临时结果集太大，内存中放不下，需要将内存中的临时表拷贝到磁盘上，形成 #sql***.MYD、#sql***.MYI(在5.6及更高的版本，临时表可以改成InnoDB引擎了，可以参考选项**default_tmp_storage_engine**)**建议：**创建适当的索引，并且适当加大**sort_buffer_size/tmp_table_size/max_heap_table_size** |
| **Creating sort index**                  | 当前的SELECT中需要用到临时表在进行ORDER BY排序**建议：**创建适当的索引 |
| **Creating tmp table**                   | 创建基于内存或磁盘的临时表，当从内存转成磁盘的临时表时，状态会变成：Copying to tmp table on disk**建议：**创建适当的索引，或者少用UNION、视图(VIEW)之类的 |
| **Reading from net**                     | 表示server端正通过网络读取客户端发送过来的请求**建议：**减小客户端发送数据包大小，提高网络带宽/质量 |
| **Sending data**                         | 从server端发送数据到客户端，也有可能是接收存储引擎层返回的数据，再发送给客户端，数据量很大时尤其经常能看见备注：Sending Data不是网络发送，是从硬盘读取，发送到网络是Writing to net**建议：**通过索引或加上LIMIT，减少需要扫描并且发送给客户端的数据量 |
| **Sorting result**                       | 正在对结果进行排序，类似Creating sort index，不过是正常表，而不是在内存表中进行排序**建议：**创建适当的索引 |
| **statistics**                           | 进行数据统计以便解析执行计划，如果状态比较经常出现，有可能是磁盘IO性能很差**建议：**查看当前io性能状态，例如iowait |
| **Waiting for global read lock**         | FLUSH TABLES WITH READ LOCK整等待全局读锁**建议：**不要对线上业务数据库加上全局读锁，通常是备份引起，可以放在业务低谷期间执行或者放在slave服务器上执行备份 |
| **Waiting for tables,Waiting for table flush** | FLUSH TABLES, ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE, OPTIMIZE TABLE等需要刷新表结构并重新打开**建议：**不要对线上业务数据库执行这些操作，可以放在业务低谷期间执行 |
| **Waiting for lock_type lock**           | >等待各种类型的锁：• Waiting for event metadata lock• Waiting for global read lock   • Waiting for schema metadata lock   • Waiting for stored function metadata lock   • Waiting for stored procedure metadata lock   • Waiting for table level lock   • Waiting for table metadata lock   • Waiting for trigger metadata lock   **建议：**比较常见的是上面提到的global read lock以及table metadata lock，建议不要对线上业务数据库执行这些操作，可以放在业务低谷期间执行。如果是table level lock，通常是因为还在使用MyISAM引擎表，赶紧转投InnoDB引擎吧，别再老顽固了 |