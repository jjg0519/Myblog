# MySQL 的日志文件

日志文件是MySQL数据库的重要组成部分。MySQL有几种不同的日志文件，通常包括错误日志文件，二进制日志，通用日志，慢查询日志，等等。这些日志可以帮助我们定位mysqld内部发生的事件，数据库性能故障，记录数据的变更历史，用户恢复数据库等等。

## MySQL日志文件系统的组成
缺省情况下，所有日志创建于mysqld数据目录中。可以通过刷新日志，来强制mysqld来关闭和重新打开日志文件（或者在某些情况下切换到一个新的日志）。当你执行一个FLUSH LOGS语句或执行mysqladmin flush-logs或mysqladmin refresh时，则日志被老化
### 错误日志

记录启动、运行或停止mysqld时出现的问题。

```mysql
mysql>  show variables like 'log_error';
+---------------+-----------------------------+
| Variable_name | Value                       |
+---------------+-----------------------------+
| log_error     | /home/mysql/logs/mysqld.log |
+---------------+-----------------------------+
1 row in set (0.01 sec)

mysql>
```



###通用日志

记录建立的客户端连接和执行的语句。

###更新日志

记录更改数据的语句。该日志在MySQL 5.1中已不再使用。

###二进制日志

记录所有更改数据的语句。还用于复制。



```bash
##查询mysql-bin.000003 log 里面关于bid_budget的操作记录
mysqlbinlog --start-datetime='2017-03-31 00:00:00' --stop-datetime='2017-03-31 09:00:00' mysql-bin.000003 |grep 'bid_budget' -B6 -A6 

mysqlbinlog --start-datetime='2017-03-31 00:00:00' --stop-datetime='2017-03-31 09:00:00' mysql-bin.000003 |grep 'bid_budget' -B6 -A6 >>/home/bid_budget_mater_addition.sql


#--base64-output=DECODE-ROWS -V  解决mix 日志乱码问题
/home/mysql/bin/mysqlbinlog   --base64-output=DECODE-ROWS -v /home/mysql/data/mysql-bin.000005 --database=ykee_workflow --start-datetime='2017-03-29 00:00:00' --stop-datetime='2017-03-31 09:00:00'  >/home/mysql-bin-00005-29.sql


grep -A1 keyword filename
找出filename中带有keyword的行，输出中除显示该行外，还显示之后的一行(After 1)
grep -B1 keyword filename
找出filename中带有keyword的行，输出中除显示该行外，还显示之前的一行(Before 1)
```



###慢查询日志

记录所有执行时间超过long_query_time秒的所有查询或不使用索引的查询。
慢查询日志是将mysql服务器中影响数据库性能的相关SQL语句记录到日志文件，通过对这些特殊的SQL语句分析，改进以达到提高数据库性能的目的。

* 通过使用--slow_query_log[={0|1}]选项来启用慢查询日志。所有执行时间超过long_query_time秒的SQL语句都会被记录到慢查询日志。
* 缺省情况下hostname-slow.log为慢查询日志文件安名，存放到数据目录，同时缺省情况下未开启慢查询日志。
* 缺省情况下数据库相关管理型SQL(比如OPTIMIZE TABLE、ANALYZE TABLE和ALTER TABLE)不会被记录到日志。
* 对于管理型SQL可以通过--log-slow-admin-statements开启记录管理型慢SQL。
* mysqld在语句执行完并且所有锁释放后记入慢查询日志。**记录顺序可以与执行顺序不相同**。获得初使表锁定的时间不算作执行时间。

###Innodb日志

innodb redo log

###接替日志(从复制服务器)
