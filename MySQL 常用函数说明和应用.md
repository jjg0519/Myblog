# MySQL 常用函数说明和应用

## concat()

MySQL的concat函数可以连接一个或者多个字符串,如

```mysql
mysql> select concat('10');
+--------------+
| concat('10') |
+--------------+
| 10           |
+--------------+
1 row in set (0.00 sec)
mysql> select concat('11','22','33');
+------------------------+
| concat('11','22','33') |
+------------------------+
| 112233                 |
+------------------------+
1 row in set (0.00 sec)
```



而[Oracle](http://www.2cto.com/database/Oracle/)的concat函数只能连接两个字符串

```sql
SQL> select concat('11','22') from dual;
```

MySQL的concat函数在连接字符串的时候，只要其中一个是NULL,那么将返回NULL

```mysql
mysql> select concat('11','22',null);
+------------------------+
| concat('11','22',null) |
+------------------------+
| NULL                   |
+------------------------+
1 row in set (0.00 sec)
```



而Oracle的concat函数连接的时候，只要有一个字符串不是NULL,就不会返回NULL

```sql
SQL> select concat('11',NULL) from dual;
```

CONCAT
--
11

2、concat_ws()函数, 表示concat with separator,即有分隔符的字符串连接
如连接后以逗号分隔
mysql> select concat_ws(',','11','22','33');

+-------------------------------+
| concat_ws(',','11','22','33') |
+-------------------------------+
| 11,22,33                      |
+-------------------------------+
1 row in set (0.00 sec)

和concat不同的是, concat_ws函数在执行的时候,不会因为NULL值而返回NULL
mysql> select concat_ws(',','11','22',NULL);
+-------------------------------+
| concat_ws(',','11','22',NULL) |
+-------------------------------+
| 11,22                         |
+-------------------------------+
1 row in set (0.00 sec)

3、group_concat()可用来行转列, Oracle没有这样的函数

完整的语法如下
group_concat([DISTINCT] 要连接的字段[Order BY ASC/DESC 排序字段] [Separator '分隔符'])
如下例子
mysql> select * from aa;

+------+------+
| id   | name |
+------+------+
|    1 | 10   |
|    1 | 20   |
|    1 | 20   |
|    2 | 20   |
|    3 | 200  |
|    3 | 500  |
+------+------+
6 rows in set (0.00 sec)
3.1 以id分组，把name字段的值打印在一行，逗号分隔(默认)
mysql> select id,group_concat(name) from aa group by id;
+------+--------------------+
| id   | group_concat(name) |
+------+--------------------+
|    1 | 10,20,20           |
|    2 | 20                 |
|    3 | 200,500            |
+------+--------------------+

3 rows in set (0.00 sec)

3.2 以id分组，把name字段的值打印在一行，分号分隔
mysql> select id,group_concat(name separator ';') from aa group by id;
+------+----------------------------------+
| id   | group_concat(name separator ';') |
+------+----------------------------------+
|    1 | 10;20;20                         |
|    2 | 20                               |
|    3 | 200;500                          |
+------+----------------------------------+

3 rows in set (0.00 sec)

3.3 以id分组，把去冗余的name字段的值打印在一行，逗号分隔

mysql> select id,group_concat(distinct name) from aa group by id;

+------+-----------------------------+
| id   | group_concat(distinct name) |
+------+-----------------------------+
|    1 | 10,20                       |
|    2 | 20                          |
|    3 | 200,500                     |
+------+-----------------------------+

3 rows in set (0.00 sec)

3.4 以id分组，把name字段的值打印在一行，逗号分隔,以name排倒序

mysql> select id,group_concat(name order by name desc) from aa group by id;

+------+---------------------------------------+
| id   | group_concat(name order by name desc) |
+------+---------------------------------------+
|    1 | 20,20,10                              |
|    2 | 20                                    |
|    3 | 500,200                               |
+------+---------------------------------------+

3 rows in set (0.00 sec)

4、repeat()函数，用来复制字符串,如下'ab'表示要复制的字符串，2表示复制的份数

mysql> select repeat('ab',2);

+----------------+
| repeat('ab',2) |
+----------------+
| abab           |
+----------------+

1 row in set (0.00 sec)

又如
mysql> select repeat('a',2);

+---------------+
| repeat('a',2) |
+---------------+
| aa            |
+---------------+
1 row in set (0.00 sec)