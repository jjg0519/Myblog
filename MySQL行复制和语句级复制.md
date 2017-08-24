# MySQL行复制和语句级复制

**MYSQL复制的几种模式**

Mysql中的复制可以是基于语句（Statement Level）的和基于行的（RowLevael）。
从 MySQL 5.1.12 开始，可以用以下三种模式来实现：
-- 基于SQL语句的复制(statement-basedreplication, SBR)，
-- 基于行的复制(row-based replication, RBR)，
-- 混合模式复制(mixed-based replication, MBR)。
相应地，binlog的格式也有三种：STATEMENT，ROW，MIXED。

在运行时可以动态低改变binlog的格式，除了以下几种情况：
. 存储过程或者触发器中间
. 启用了NDB(Mysql cluster 主要采取的存储引擎)
. 当前会话试用 RBR 模式，并且已打开了临时表

如果binlog采用了 MIXED 模式，那么在以下几种情况下会自动将binlog的模式由 SBR 模式改成RBR 模式。
. 当DML语句更新一个NDB(Mysql cluster 主要采取的存储引擎)表时
. 当函数中包含 UUID() 时
. 2个及以上包含 AUTO_INCREMENT 字段的表被更新时
. 行任何 INSERT DELAYED 语句时
. 用 UDF (User Definition Function)时
. 视图中必须要求使用 RBR 时，例如创建视图是使用了UUID() 函数
设定主从复制模式的方法非常简单，只要在以前设定复制配置的基础上，再加一个参数：

```
binlog_format="STATEMENT" 
#binlog_format="ROW"
#binlog_format="MIXED"
```

当然了，也可以在运行时动态修改binlog的格式。例如

```mysql
mysql> SET SESSION binlog_format = 'STATEMENT';
mysql> SET SESSION binlog_format = 'ROW';
mysql> SET SESSION binlog_format = 'MIXED';
mysql> SET GLOBAL binlog_format = 'STATEMENT';
mysql> SET GLOBAL binlog_format = 'ROW';
mysql> SET GLOBAL binlog_format = 'MIXED';
```





### 3.1、基于语句的复制(Statement-Based Replication)

​     MySQL 5.0及之前的版本仅支持基于语句的复制（也叫做逻辑复制，logical replication），这在数据库并不常见。master记录下改变数据的查询，然后，slave从中继日志中读取事件，并执行它，这些SQL语句与master执行的语句一样。
这种方式的优点就是实现简单。此外，基于语句的复制的二进制日志可以很好的进行压缩，而且日志的数据量也较小，占用带宽少——例如，一个更新GB的数据的查询仅需要几十个字节的二进制日志。而mysqlbinlog对于基于语句的日志处理十分方便。
​      但是，基于语句的复制并不是像它看起来那么简单，因为一些查询语句依赖于master的特定条件，例如，master与slave可能有不同的时间。所以，MySQL的二进制日志的格式不仅仅是查询语句，还包括一些元数据信息，例如，当前的时间戳。即使如此，还是有一些语句，比如，CURRENT USER函数，不能正确的进行复制。此外，存储过程和触发器也是一个问题。
​     另外一个问题就是基于语句的复制必须是串行化的。这要求大量特殊的代码，配置，例如InnoDB的next-key锁等。并不是所有的存储引擎都支持基于语句的复制。

### 3.2、基于记录的复制(Row-Based Replication)

​      MySQL增加基于记录的复制，在二进制日志中记录下实际数据的改变，这与其它一些DBMS的实现方式类似。这种方式有优点，也有缺点。优点就是可以对任何语句都能正确工作，一些语句的效率更高。主要的缺点就是二进制日志可能会很大，而且不直观，所以，你不能使用mysqlbinlog来查看二进制日志。
对于一些语句，基于记录的复制能够更有效的工作，如：
mysql> INSERT INTO summary_table(col1, col2, sum_col3)
​    -> SELECT col1, col2, sum(col3)
​    -> FROM enormous_table
​    -> GROUP BY col1, col2;
​     假设，只有三种唯一的col1和col2的组合，但是，该查询会扫描原表的许多行，却仅返回三条记录。此时，基于记录的复制效率更高。
​    另一方面，下面的语句，基于语句的复制更有效：
 mysql> UPDATE enormous_table SET col1 = 0;
此时使用基于记录的复制代价会非常高。由于两种方式不能对所有情况都能很好的处理，所以，MySQL 5.1支持在基于语句的复制和基于记录的复制之前动态交换。你可以通过设置session变量binlog_format来进行控制。

两种模式各自的优缺点**：

**SBR **的优点：

1. 技术比较成熟


2. binlog文件较小，log文件可读
3. binlog中包含了所有数据库更改信息，可以据此来审核数据库的安全等情况
4. binlog可以用于实时的还原，而不仅仅用于复制
5. 主从版本可以不一样，从服务器版本可以比主服务器版本高

**SBR **的缺点

1. 不是所有的UPDATE语句都能被复制，尤其是包含不确定操作的时候。

2. 调用具有不确定因素的 UDF 时复制也可能出问题
使用以下函数的语句也无法被复制：
\* LOAD_FILE()
\* UUID()
\* USER()
\* FOUND_ROWS()
\* SYSDATE() (除非启动时启用了 --sysdate-is-now 选项)

3. 对于有 AUTO_INCREMENT 字段的 InnoDB表而言，INSERT 语句会阻塞其他 INSERT 语句，对于一些复杂的语句，在从服务器上的耗资源情况会更严重，而 RBR 模式下，只会对那个发生变化的记录产生影响。以为需要记录上下文信息所以存储函数(不是存储过程，mysql内部存储的函数)在被调用的同时也会执行一次 NOW() 函数。

4. 确定了的 UDF 也需要在从服务器上执行
数据表必须几乎和主服务器保持一致才行，否则可能会导致复制出错
执行复杂语句如果出错的话，会消耗更多资源

**RBR **的优点：

1. 任何情况都可以被复制，这对复制来说是最安全可靠的


2. 如果从服务器上的表如果有主键的话，复制就会快了很多\


3. 从服务器上采用多线程来执行复制成为可能

4.  不会出现某些特定情况下的存储过程，或function，以及trigger 的调用和触发无法被正确复制的问题。

**RBR** 的缺点：

1. 相比SBR的binlog 大了很多，还原的时候可能比较慢
2. 复杂的回滚时 binlog 中会包含大量的数据
3. 因为RBR是基于行级的，所以如果有alter table 之类的DDL操作产生的数据量非常大。(GRANT，REVOKE，SETPASSWORD等语句建议使用SBR模式记录).
4. 当在非事务表上执行一段堆积的SQL语句时，最好采用 SBR 模式，否则很容易导致主从服务器的数据不一致情况发生

5.  采取RBR模式记录的话，log是不可读的。（经过加密）

>注：采用 RBR 模式后，能解决很多原先出现的主键重复问题。