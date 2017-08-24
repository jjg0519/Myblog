#MySQl-optimizer_switch 实例

mysql 5.1中开始引入optimizer_switch, 控制mysql优化器行为。他有一些结果集，通过on和off控制开启和关闭优化器行为。使用有效期全局和会话两个级别，在5.5中optimizer_swtich 可取结果如下，不同mysql版本可取结果不同。[5.1](http://dev.mysql.com/doc/refman/5.1/en/switchable-optimizations.html)和[5.6](http://dev.mysql.com/doc/refman/5.6/en/switchable-optimizations.html)参考官方文档。

```mysql
mysql> select @@optimizer_switch;
+------------------------------------------------------------------------------------------------------------------------+
| @@optimizer_switch                                                                                                     |
+------------------------------------------------------------------------------------------------------------------------+
| index_merge=on,index_merge_union=on,index_merge_sort_union=on,index_merge_intersection=on,engine_condition_pushdown=on | 
+------------------------------------------------------------------------------------------------------------------------+
```



在mysql优化语句过程中，可通过设置optimizer_switch控制优化行为。

 在我们生产环境上，某天mysql服务器压力特别大，load一度达到了100，查询发现数据库中有大量的sql语句state 状态result sorting ，result sorting这种排序特别消耗cpu和内存资源。抽取其中的一条sql查看执行计划

```mysql
 mysql> explain select product_id,main_product_id,is_main_product,external_product_id from product where shop_id = '5939' AND status in  ('0') AND main_product_id in ('0') AND product_type in ('0','1') order by operator_datedesc limit 800,100;

+----+-------------+----------------+-------------+---------------------------------------------+--------------------------------+---------+------+------+------------------------------------------------------------------------------+

| id | select_type |table          |type        |possible_keys                              |key                           | key_len | ref  | rows |Extra                                                                       |

+----+-------------+----------------+-------------+---------------------------------------------+--------------------------------+---------+------+------+------------------------------------------------------------------------------+

|  1 |SIMPLE      | product_common |index_merge | main_product_id,shop_id,status,product_type |shop_id,main_product_id,status | 5,5,5   | NULL |  810 | Usingintersect(shop_id,main_product_id,status); Using where; Using filesort
```



type类型是 index_merge（索引合并排序）。业务需要或滥建索引，数据表上会建很多索引。针对这个查询，可以通过修改optimizer_switch来优化优化器行为：

set global optimizer_seitch="index_merge=off"    

这个方式也行会有一定风险，影响其他sql，在服务器端操作要谨慎  

另外一种方式：如果建立的索引不合理，可以建立复合索引，然后删掉这些单列索引，这样sql效率也会提高。本身index_merge提高查询效率，由于本身的特点，可能会引起性能问题，那么，应该建立复合索引还是单列索引，这篇文章[《multi-column-indexes-vs-index-merge》](http://www.mysqlperformanceblog.com/2009/09/19/multi-column-indexes-vs-index-merge/)写的很好。