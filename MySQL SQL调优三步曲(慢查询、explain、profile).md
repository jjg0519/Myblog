# MySQL调优三步曲

## 概述

mysql profile explain slow_query_log分析优化查询

在做性能测试中经常会遇到一些sql的问题，其实做性能测试这几年遇到问题最多还是数据库这块，要么就是IO高要么就是cpu高，所以对数据的优化在性能测试过程中占据着很重要的地方，下面我就介绍一些msyql性能调优过程中经常用到的三件利器：

* 慢查询 （分析出现出问题的sql）


* Explain （显示了mysql如何使用索引来处理select语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句）


* Profile（查询到 SQL 会执行多少时间, 并看出 CPU/Memory 使用量, 执行过程中 Systemlock, Table lock 花多少时间等等.）

## mysql的慢查询

1，配置开启

Linux:

在mysql配置文件my.cnf中增加

log-slow-queries=/var/lib/mysql/slowquery.log (指定日志文件存放位置，可以为空，系统会给一个缺省的文件host_name-slow.log)

long_query_time=2 (记录超过的时间，默认为10s)

log-queries-not-using-indexes (log下来没有使用索引的query,可以根据情况决定是否开启)

log-long-format (如果设置了，所有没有使用索引的查询也将被记录)

Windows:

在my.ini的[mysqld]添加如下语句：

log-slow-queries =E:\web\mysql\log\mysqlslowquery.log

long_query_time = 2(其他参数如上)

 

2,查看方式

Linux:

使用mysql自带命令mysqldumpslow查看

常用命令

-s ORDER what to sort by (t, at, l, al, r, aretc), 'at’ is default

-t NUM just show the top n queries

-g PATTERN grep: only consider stmts that includethis string

eg:

s，是order的顺序，说明写的不够详细，俺用下来，包括看了代码，主要有 c,t,l,r和ac,at,al,ar，分别是按照query次数，时间，lock的时间和返回的记录数来排序，前面加了a的时倒序 -t，是top n的意思，即为返回前面多少条的数据 -g，后边可以写一个正则匹配模式，大小写不敏感的

mysqldumpslow -s c -t 20 host-slow.log

mysqldumpslow -s r -t 20 host-slow.log

上述命令可以看出访问次数最多的20个sql语句和返回记录集最多的20个sql。

mysqldumpslow -t 10 -s t -g “left join” host-slow.log这个是按照时间返回前10条里面含有左连接的sql语句。

## explain

执行

```mysql
EXPLAIN SELECT * FROM res_user ORDER BYmodifiedtime LIMIT 0,1000 
```

得到如下结果：

 

显示结果分析：

```mysql
table |  type | possible_keys | key |key_len  | ref | rows | Extra 
```

EXPLAIN列的解释： 

* table 

  显示这一行的数据是关于哪张表的 

* type 

  这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为const、eq_reg、ref、range、indexhe和ALL 

  **不同连接类型的解释（按照效率高低的顺序排序）**

  * system
    这是const联接类型的一个特例。表仅有一行满足条件.如下(t3表上的id是 primary key)

    ```mysql
    mysql> explain select * from (select * from t3 where id=3952602) a ;
    +----+-------------+------------+--------+-------------------+---------+---------+------+------+-------+
    | id | select_type | table      | type   | possible_keys     | key     | key_len | ref  | rows | Extra |
    +----+-------------+------------+--------+-------------------+---------+---------+------+------+-------+
    |  1 | PRIMARY     | <derived2> | system | NULL              | NULL    | NULL    | NULL |    1 |       |
    |  2 | DERIVED     | t3         | const  | PRIMARY,idx_t3_id | PRIMARY | 4       |      |    1 |       |
    +----+-------------+------------+--------+-------------------+---------+---------+------+------+-------+
    ```

  * const
    表最多有一个匹配行，它将在查询开始时被读取。因为仅有一行，在这行的列值可被优化器剩余部分认为是常数。const表很快，因为它们只读取一次！

    const用于用常数值比较PRIMARY KEY或UNIQUE索引的所有部分时。在下面的查询中，tbl_name可以用于const表：

    ```mysql

    SELECT * from tbl_name WHERE primary_key=1；
    SELECT * from tbl_name WHERE primary_key_part1=1和 primary_key_part2=2；

    #例如:
    mysql> explain select * from t3 where id=3952602;
    +----+-------------+-------+-------+-------------------+---------+---------+-------+------+-------+
    | id | select_type | table | type  | possible_keys     | key     | key_len | ref   | rows | Extra |
    +----+-------------+-------+-------+-------------------+---------+---------+-------+------+-------+
    |  1 | SIMPLE      | t3    | const | PRIMARY,idx_t3_id | PRIMARY | 4       | const |    1 |       |
    +----+-------------+-------+-------+-------------------+---------+---------+-------+------+-------+

    ```

  * eq_ref
    对于每个来自于前面的表的行组合，从该表中读取一行。这可能是最好的联接类型，除了const类型。它用在一个索引的所有部分被联接使用并且索引是UNIQUE或PRIMARY KEY。
    在下面的例子中，MySQL可以使用eq_ref联接来处理ref_tables：

    ```mysql

    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column=other_table.column;

    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column_part1=other_table.column
        AND ref_table.key_column_part2=1;

    #例如
    mysql> create unique index  idx_t3_id on t3(id) ;
    Query OK, 1000 rows affected (0.03 sec)
    Records: 1000  Duplicates: 0  Warnings: 0

    mysql> explain select * from t3,t4 where t3.id=t4.accountid;
    +----+-------------+-------+--------+-------------------+-----------+---------+----------------------+------+-------+
    | id | select_type | table | type   | possible_keys     | key       | key_len | ref                  | rows | Extra |
    +----+-------------+-------+--------+-------------------+-----------+---------+----------------------+------+-------+
    |  1 | SIMPLE      | t4    | ALL    | NULL              | NULL      | NULL    | NULL                 | 1000 |       |
    |  1 | SIMPLE      | t3    | eq_ref | PRIMARY,idx_t3_id | idx_t3_id | 4       | dbatest.t4.accountid |    1 |       |
    +----+-------------+-------+--------+-------------------+-----------+---------+----------------------+------+-------+
    ```

  * ref
    对于每个来自于前面的表的行组合，所有有匹配索引值的行将从这张表中读取。如果联接只使用键的最左边的前缀，或如果键不是UNIQUE或PRIMARY KEY（换句话说，如果联接不能基于关键字选择单个行的话），则使用ref。如果使用的键仅仅匹配少量行，该联接类型是不错的。

    ref可以用于使用=或<=>操作符的带索引的列。

    在下面的例子中，MySQL可以使用ref联接来处理ref_tables：

    ```mysql
    SELECT * FROM ref_table WHERE key_column=expr;

    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column=other_table.column;

    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column_part1=other_table.column
        AND ref_table.key_column_part2=1;

    #例如:
    mysql> drop index idx_t3_id on t3;
    Query OK, 1000 rows affected (0.03 sec)
    Records: 1000  Duplicates: 0  Warnings: 0

    mysql> create index idx_t3_id on t3(id) ;
    Query OK, 1000 rows affected (0.04 sec)
    Records: 1000  Duplicates: 0  Warnings: 0

    mysql> explain select * from t3,t4 where t3.id=t4.accountid;
    +----+-------------+-------+------+-------------------+-----------+---------+----------------------+------+-------+
    | id | select_type | table | type | possible_keys     | key       | key_len | ref                  | rows | Extra |
    +----+-------------+-------+------+-------------------+-----------+---------+----------------------+------+-------+
    |  1 | SIMPLE      | t4    | ALL  | NULL              | NULL      | NULL    | NULL                 | 1000 |       |
    |  1 | SIMPLE      | t3    | ref  | PRIMARY,idx_t3_id | idx_t3_id | 4       | dbatest.t4.accountid |    1 |       |
    +----+-------------+-------+------+-------------------+-----------+---------+----------------------+------+-------+
    2 rows in set (0.00 sec)
    ```

    ​

  * ref_or_null
    该联接类型如同ref，但是添加了MySQL可以专门搜索包含NULL值的行。在解决子查询中经常使用该联接类型的优化。

    在下面的例子中，MySQL可以使用ref_or_null联接来处理ref_tables：

    ```mysql
    SELECT * FROM ref_table
    WHERE key_column=expr OR key_column IS NULL;
    ```

  * index_merge
    该联接类型表示使用了索引合并优化方法。在这种情况下，key列包含了使用的索引的清单，key_len包含了使用的索引的最长的关键元素。

    ```mysql
    mysql> explain select * from t4 where id=3952602 or accountid=31754306 ;
    +----+-------------+-------+-------------+----------------------------+----------------------------+---------+------+------+------------------------------------------------------+
    | id | select_type | table | type        | possible_keys              | key                        | key_len | ref  | rows | Extra                                                |
    +----+-------------+-------+-------------+----------------------------+----------------------------+---------+------+------+------------------------------------------------------+
    |  1 | SIMPLE      | t4    | index_merge | idx_t4_id,idx_t4_accountid | idx_t4_id,idx_t4_accountid | 4,4     | NULL |    2 | Using union(idx_t4_id,idx_t4_accountid); Using where |
    +----+-------------+-------+-------------+----------------------------+----------------------------+---------+------+------+------------------------------------------------------+
    1 row in set (0.00 sec)
    ```

  * unique_subquery
    该类型替换了下面形式的IN子查询的ref：

    value IN (SELECT primary_key FROM single_table WHERE some_expr)
    unique_subquery是一个索引查找函数，可以完全替换子查询，效率更高。

  * index_subquery
    该联接类型类似于unique_subquery。可以替换IN子查询，但只适合下列形式的子查询中的非唯一索引：

    value IN (SELECT key_column FROM single_table WHERE some_expr)

  * range
    只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。key_len包含所使用索引的最长关键元素。在该类型中ref列为NULL。

    当使用=、<>、>、>=、<、<=、IS NULL、<=>、BETWEEN或者IN操作符，用常量比较关键字列时，可以使用range

    ```mysql

    mysql> explain select * from t3 where id=3952602 or id=3952603 ;
    +----+-------------+-------+-------+-------------------+-----------+---------+------+------+-------------+
    | id | select_type | table | type  | possible_keys     | key       | key_len | ref  | rows | Extra       |
    +----+-------------+-------+-------+-------------------+-----------+---------+------+------+-------------+
    |  1 | SIMPLE      | t3    | range | PRIMARY,idx_t3_id | idx_t3_id | 4       | NULL |    2 | Using where |
    +----+-------------+-------+-------+-------------------+-----------+---------+------+------+-------------+
    1 row in set (0.02 sec)
    ```

  * index
    该联接类型与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小。

    当查询只使用作为单索引一部分的列时，MySQL可以使用该联接类型。

  * all
    对于每个来自于先前的表的行组合，进行完整的表扫描。如果表是第一个没标记const的表，这通常不好，并且通常在它情况下很差。通常可以增加更多的索引而不要使用ALL，使得行能基于前面的表中的常数值或列值被检索出。

  ​

* possible_keys 

  显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关的域从WHERE语句中选择一个合适的语句 

* key 

  实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足的索引。这种情况下，可以在SELECT语句中使用USE INDEX（indexname）来强制使用一个索引或者用IGNORE INDEX（indexname）来强制MYSQL忽略索引 

* key_len 

  使用的索引的长度。在不损失精确性的情况下，长度越短越好 

* ref 

  显示索引的哪一列被使用了，如果可能的话，是一个常数 

* rows 

  MYSQL认为必须检查的用来返回请求数据的行数 

* Extra 

  关于MYSQL如何解析查询的额外信息。将在表4.3中讨论，但这里可以看到的坏的例子是Using temporary和Using filesort，意思MYSQL根本不能使用索引，结果是检索会很慢;

  **Extra列返回的描述的意义** 

  * Distinct 

    一旦MYSQL找到了与行相联合匹配的行，就不再搜索了 

  * Not exists 

    MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行， 
    就不再搜索了 

  * Range checked for each Record（index map:#） 

    没有找到理想的索引，因此对于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一 

  * Using filesort 

    看到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行 

  * Using index 

    列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候 

  * Using temporary 

    看到这个的时候，查询需要优化了。这里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上 

  * Where used 

    使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题

## profile

我们可以先使用

```mysql
mysql> SELECT @@profiling;
+-------------+
| @@profiling |
+-------------+
|          0 |
+-------------+
1 row in set (0.00 sec)
```

来查看是否已经启用profile，如果profilng值为0，可以通过

```mysql
mysql> SET profiling = 1;
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT @@profiling;
+-------------+
| @@profiling |
+-------------+
|          1 |
+-------------+
1 row in set (0.00 sec)
```

来启用。启用profiling之后，我们执行一条查询语句，比如：

```mysql
SELECT * FROM res_user ORDER BY modifiedtimeLIMIT 0,1000
mysql> show profiles;
+----------+------------+-------------------------------------------------------------+
| Query_ID | Duration   | Query                                                      |
+----------+------------+-------------------------------------------------------------+
|        1| 0.00012200 | SELECT @@profiling                                         |
|        2| 1.54582000 | SELECT res_id FROM res_user ORDER BY modifiedtime LIMIT 0,3 |
+----------+------------+-------------------------------------------------------------+
2 rows in set (0.00 sec)  
#注意：Query_ID表示刚执行的查询语句
mysql> show profile for query 2;
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| starting                       | 0.000013 |
| checking query cache for query | 0.000035 |
| Opening tables                 | 0.000009 |
| System lock                    | 0.000002 |
| Table lock                     | 0.000015 |
| init                           | 0.000011 |
| optimizing                     | 0.000003 |
| statistics                     | 0.000006 |
| preparing                      | 0.000006 |
| executing                      | 0.000001 |
| Sorting result                 | 1.545565 |
| Sending data                   | 0.000038 |
| end                            | 0.000003 |
| query end                      | 0.000003 |
| freeing items                  | 0.000069 |
| storing result in query cache  | 0.000004 |
| logging slow query             | 0.000001 |
| logging slow query             | 0.000033 |
| cleaning up                    | 0.000003 |
+--------------------------------+----------+
19 rows in set (0.00 sec)
```

结论：以看出此条查询语句的执行过程及执行时间，总的时间约为1.545s。这时候我们再执行一次。

```mysql
mysql> SELECT res_id FROM res_user ORDERBY modifiedtime LIMIT 0,3;
+---------+
| res_id |
+---------+
| 1000305 |
| 1000322 |
| 1000323 |
+---------+
3 rows in set (0.00 sec)
mysql> show profiles;
+----------+------------+-------------------------------------------------------------+
| Query_ID | Duration   | Query                                                      |
+----------+------------+-------------------------------------------------------------+
|       1 | 0.00012200 | SELECT @@profiling                                          |
|       2 | 1.54582000 | SELECT res_id FROM res_userORDER BY modifiedtime LIMIT 0,3 |
|        3 | 0.00006500 | SELECT res_id FROMres_user ORDER BY modifiedtime LIMIT 0,3 |
+----------+------------+-------------------------------------------------------------+
3 rows in set (0.00 sec)
mysql> show profile for query 3;
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| starting                       | 0.000013 |
| checking query cache for query | 0.000005|
| checking privileges on cached  | 0.000003 |
| sending cached result to clien | 0.000040|
| logging slow query             | 0.000002 |
| cleaning up                    | 0.000002 |
+--------------------------------+----------+
6 rows in set (0.00 sec) (注意红色标记的地方)
```

结论：
可以看出此次第二次查询因为前一次的查询生成了cache，所以这次无需从数据库文件中再次读取数据而是直接从缓存中读取，结果查询时间比第一次快多了（第一次查询用了1.5秒而本次用了不到5毫秒）。