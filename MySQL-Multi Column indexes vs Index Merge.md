# MySQL-Multi Column indexes vs Index Merge

[Peter Zaitsev](https://www.percona.com/blog/author/admin/)  | September 19, 2009 |  Posted In: [Insight for DBAs](https://www.percona.com/blog/category/dba-insight/), [MySQL](https://www.percona.com/blog/category/mysql/)

The mistake I commonly see among MySQL users is how indexes are created. Quite commonly people just index individual columns as they are referenced in where clause thinking this is the optimal indexing strategy. For example if I would have something like **AGE=18 AND STATE=’CA’** they would create 2 separate indexes on AGE and STATE columns.

The better strategy is often to have combined multi-column index on (AGE,STATE). Lets see why it is the case.

MySQL indexes are (with few exceptions) BTREE indexes – this index type is very good to be able to quickly lookup the data on any its prefix and traversing ranges between values in sorted order. For example when you query AGE=18 with single column BTREE index MySQL will dive into the index to find first matching row and when will continue scanning index in order until it runs into the value of AGE more than 18 when it stops doing so assuming there are no more matching. The RANGES such as AGE BETWEEN 18 AND 20 work similar way – MySQL just stops at different value.

The enumerated ranges such as AGE IN (18,20,30) are more complicated as this is in fact several separate index lookups.

So we spoke about how MySQL uses the index but not exactly what it gets from the index – typically (unless it is covering index) MySQL gets a “row pointer” which can be primary key value (for Innodb tables) physical file offset (for MyISAM tables) or something else. It is important internally storage engine can use that value to lookup the full row data corresponding to the given index entry.

So what choices does MySQL have if you have just 2 separate indexes ? I will either use just one of them to look up the data (and check remaining portion on WHERE clause after data is read) or it can lookup the row pointers from all indexes, intersect them and when lookup the data.

Which method works better depends on selectivity and correlation. If where clause from first column selects 5% of the rows and applying where clause on second column brings it down to 1% using intersection makes sense. If second where clause just brings it down to 4.5% it is most likely better to use single index only and simply filter out rows we do not need after lookup.

Lets look at some examples now:

```mysql
CREATE TABLE `idxtest` (
  `i1` int(10) unsigned NOT NULL,
  `i2` int(10) unsigned NOT NULL,
  `val` varchar(40) DEFAULT NULL,
  KEY `i1` (`i1`),
  KEY `i2` (`i2`),
  KEY `combined` (`i1`,`i2`)
) ENGINE=MyISAM DEFAULT CHARSET=latin1
```

I made columns i1 and i2 independent in this case each selecting about 1% rows from this table which contains about 10M rows.

```mysql
mysql [localhost] {msandbox} (test) > explain select avg(length(val)) from idxtest  where i1=50 and i2=50;
+----+-------------+---------+------+----------------+----------+---------+-------------+------+-------+
| id | select_type | table   | type | possible_keys  | key      | key_len | ref         | rows | Extra |
+----+-------------+---------+------+----------------+----------+---------+-------------+------+-------+
|  1 | SIMPLE      | idxtest | ref  | i1,i2,combined | combined | 8       | const,const |  665 |       |
+----+-------------+---------+------+----------------+----------+---------+-------------+------+-------+
1 row in set (0.00 sec)
```



As you can see MySQL picks to use combined index and indeed the query completes in **less than 10ms**

Now what if we pretend we only have single column indexes (by hinting optimizer to ignore combined index)

```mysql
mysql [localhost] {msandbox} (test) > explain select avg(length(val)) from idxtest ignore index (combined) where i1=50 and i2=50;
+----+-------------+---------+-------------+---------------+-------+---------+------+------+-------------------------------------+
| id | select_type | table   | type        | possible_keys | key   | key_len | ref  | rows | Extra                               |
+----+-------------+---------+-------------+---------------+-------+---------+------+------+-------------------------------------+
|  1 | SIMPLE      | idxtest | index_merge | i1,i2         | i1,i2 | 4,4     | NULL | 1032 | Using intersect(i1,i2); Using where |
+----+-------------+---------+-------------+---------------+-------+---------+------+------+-------------------------------------+
1 row in set (0.00 sec)
```



As you can see in this case MySQL does the Intersection for the index values (the process a mentioned above) and the query now runs in about **70 ms** which is quite a difference.

Now lets see if using just single index and post filtering is any better:

```mysql
mysql [localhost] {msandbox} (test) > explain select avg(length(val)) from idxtest ignore index (combined,i2) where i1=50 and i2=50;
+----+-------------+---------+------+---------------+------+---------+-------+--------+-------------+
| id | select_type | table   | type | possible_keys | key  | key_len | ref   | rows   | Extra       |
+----+-------------+---------+------+---------------+------+---------+-------+--------+-------------+
|  1 | SIMPLE      | idxtest | ref  | i1            | i1   | 4       | const | 106222 | Using where |
+----+-------------+---------+------+---------------+------+---------+-------+--------+-------------+
1 row in set (0.00 sec)
```



Now this query scans a lot of rows and completed in about **290ms**
So we can see index merge indeed improves performance compared to single index only but it is by far better to use multi column indexes.

But the problems with Index Merge do not stop there. It is currently rather restricted in which conditions it would do the index merge:

```mysql
mysql [localhost] {msandbox} (test) > explain select avg(length(val)) from idxtest ignore index (combined) where i1=50 and i2 in (49,50);
+----+-------------+---------+------+---------------+------+---------+-------+--------+-------------+
| id | select_type | table   | type | possible_keys | key  | key_len | ref   | rows   | Extra       |
+----+-------------+---------+------+---------------+------+---------+-------+--------+-------------+
|  1 | SIMPLE      | idxtest | ref  | i1,i2         | i1   | 4       | const | 106222 | Using where |
+----+-------------+---------+------+---------------+------+---------+-------+--------+-------------+
1 row in set (0.00 sec)
```



As soon as any of the columns has enumeration instead of equality index merge is not selected any more. Even though in this case it should be good idea as i2 in (49,50) is still rather selective.

Now lets do another test. I’ve changed the table to make columns i1 and i2 highly correlated. In fact they are now the same:

```mysql
mysql [localhost] {msandbox} (test) > update idxtest set i2=i1;
Query OK, 10900996 rows affected (6 min 47.87 sec)
Rows matched: 11010048  Changed: 10900996  Warnings: 0
```



Lets see what happens in this case

```mysql
mysql [localhost] {msandbox} (test) > explain select avg(length(val)) from idxtest where i1=50 and i2=50;
+----+-------------+---------+-------------+----------------+-------+---------+------+------+-------------------------------------+
| id | select_type | table   | type        | possible_keys  | key   | key_len | ref  | rows | Extra                               |
+----+-------------+---------+-------------+----------------+-------+---------+------+------+-------------------------------------+
|  1 | SIMPLE      | idxtest | index_merge | i1,i2,combined | i2,i1 | 4,4     | NULL |  959 | Using intersect(i2,i1); Using where |
+----+-------------+---------+-------------+----------------+-------+---------+------+------+-------------------------------------+
1 row in set (0.00 sec)
```



Hm… Optimizer decides to use index merge in this case which is probably the worse decision it could do. Indeed the query takes **360ms** Note also the estimated values of “rows” is very wrong here.

This happens because Optimizer assumes columns i1 and i2 are independent estimating selectivity for the intersection. This is as good as it can to because there is no correlation statistics available.

```mysql
mysql [localhost] {msandbox} (test) >  explain select avg(length(val)) from idxtest ignore index(i2) where i1=50 and i2=50;
+----+-------------+---------+------+---------------+------+---------+-------+--------+-------------+
| id | select_type | table   | type | possible_keys | key  | key_len | ref   | rows   | Extra       |
+----+-------------+---------+------+---------------+------+---------+-------+--------+-------------+
|  1 | SIMPLE      | idxtest | ref  | i1,combined   | i1   | 4       | const | 106222 | Using where |
+----+-------------+---------+------+---------------+------+---------+-------+--------+-------------+
1 row in set (0.00 sec)
```



Now if we do not allow MySQL optimizer to use second index and hence index merge, what does it turn to ? It is not combined index but single index on another column. This is because MySQL is able to estimate number of rows it will find using both indexes and as they are about the same it picks smaller index. The query **takes 290ms** which is exactly what we’ve seen before.

What if we leave MySQL no choice but only to use combined index:

```mysql
mysql [localhost] {msandbox} (test) > explain select avg(length(val)) from idxtest ignore index(i1,i2) where i1=50 and i2=50;
+----+-------------+---------+------+---------------+----------+---------+-------------+--------+-------+
| id | select_type | table   | type | possible_keys | key      | key_len | ref         | rows   | Extra |
+----+-------------+---------+------+---------------+----------+---------+-------------+--------+-------+
|  1 | SIMPLE      | idxtest | ref  | combined      | combined | 8       | const,const | 121137 |       |
+----+-------------+---------+------+---------------+----------+---------+-------------+--------+-------+
1 row in set (0.00 sec)
```



We can see here MySQL estimates 20% more rows to traverse, which is wrong of course – it can’t be more than if only index prefix is used. MySQL does not know it as it looks at stats from different indexes independently not trying to reconcile them some way.

Because index is longer query execution takes a bit longer – **300ms**

So in this case we see index merge is chosen even though it turns out to be the worst plan. Though technically it is right plan considering the statistics MySQL had available.

It is very easy to disable index merge if you do not want it to run, however I do not know of the hint in MySQL which would allow forcing using index merge when MySQL does not think it should be used. I hope hint would be added at some point.

Finally let me mention the case when Index merge works much better than multiple column indexes. This is in case you’re using **OR** between the columns. In this case the combined index is useless and MySQL has an option of doing full table scan or doing the Union (instead of intersection) on values it gets from the single table.

I have reverted table to have i1 and i2 as independent columns in this case to look at more typical case:

```mysql
mysql [localhost] {msandbox} (test) > explain select avg(length(val)) from idxtest   where i1=50 or i2=50;
+----+-------------+---------+-------------+----------------+-------+---------+------+--------+---------------------------------+
| id | select_type | table   | type        | possible_keys  | key   | key_len | ref  | rows   | Extra                           |
+----+-------------+---------+-------------+----------------+-------+---------+------+--------+---------------------------------+
|  1 | SIMPLE      | idxtest | index_merge | i1,i2,combined | i1,i2 | 4,4     | NULL | 203803 | Using union(i1,i2); Using where |
+----+-------------+---------+-------------+----------------+-------+---------+------+--------+---------------------------------+
1 row in set (0.00 sec)
```



This query takes **660ms** to execute. now if we change it:

If we disable index on the second column we get a full table scan:

```mysql
mysql [localhost] {msandbox} (test) > explain select avg(length(val)) from idxtest ignore index(i2)  where i1=50 or i2=50;
+----+-------------+---------+------+---------------+------+---------+------+----------+-------------+
| id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows     | Extra       |
+----+-------------+---------+------+---------------+------+---------+------+----------+-------------+
|  1 | SIMPLE      | idxtest | ALL  | i1,combined   | NULL | NULL    | NULL | 11010048 | Using where |
+----+-------------+---------+------+---------------+------+---------+------+----------+-------------+
1 row in set (0.00 sec)
```



Note MySQL puts i1 and combined into “possible_key” while it has no way to use them for this query.

The query takes **3370ms** if this plan is used.

Note the query takes about 5 times longer even though in case of full table scan about 50 times more rows are scanned. This reflects very large performance difference between full table scan and access through the index, which seems to be about 10x (in terms of cost access per row) even though it is in memory workload.

For UNION case the optimizer is more advanced and it is able to deal with ranges:

```mysql
mysql [localhost] {msandbox} (test) > explain select avg(length(val)) from idxtest   where i1=50 or i2 in (49,50);
+----+-------------+---------+-------------+----------------+-------+---------+------+--------+--------------------------------------+
| id | select_type | table   | type        | possible_keys  | key   | key_len | ref  | rows   | Extra                                |
+----+-------------+---------+-------------+----------------+-------+---------+------+--------+--------------------------------------+
|  1 | SIMPLE      | idxtest | index_merge | i1,i2,combined | i1,i2 | 4,4     | NULL | 299364 | Using sort_union(i1,i2); Using where |
+----+-------------+---------+-------------+----------------+-------+---------+------+--------+--------------------------------------+
1 row in set (0.00 sec)
```



## 结论

and  多列索引

or   单列索引

As a summary:Use multi column indexes is typically best idea if you use **AND** between such columns in where clause. Index merge does helps performance but it is far from performance of combined index in this case. In case you’re using **OR** between columns – single column indexes are required for index merge to work and combined indexes can’t be used for such queries.

P.S The tests were done in **MySQL 5.4.2**