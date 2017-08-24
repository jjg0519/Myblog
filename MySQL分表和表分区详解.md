
# MySQL分表和表分区详解



## 为什么要分表和分区？

日常开发中我们经常会遇到大表的情况，所谓的大表是指存储了百万级乃至千万级条记录的表。这样的表过于庞大，导致数据库在查询和插入的时候耗时太长，性能低下，如果涉及联合查询的情况，性能会更加糟糕。分表和表分区的目的就是减少数据库的负担，提高数据库的效率，通常点来讲就是提高表的增删改查效率。

## 什么是分表？

分表是将一个大表按照一定的规则分解成多张具有独立存储空间的实体表，我们可以称为子表，每个表都对应三个文件，MYD数据文件，.MYI索引文件，.frm表结构文件。这些子表可以分布在同一块磁盘上，也可以在不同的机器上。app读写的时候根据事先定义好的规则得到对应的子表名，然后去操作它。

## 什么是分区？

分区和分表相似，都是按照规则分解表。不同在于分表将大表分解为若干个独立的实体表，而分区是将数据分段划分在多个位置存放，可以是同一块磁盘也可以在不同的机器。分区后，表面上还是一张表，但数据散列到多个位置了。app读写的时候操作的还是大表名字，db自动去组织分区的数据。

## MySQL分表和分区有什么联系呢？

1.都能提高mysql的性高，在高并发状态下都有一个良好的表现。
2.分表和分区不矛盾，可以相互配合的，对于那些大访问量，并且表数据比较多的表，我们可以采取分表和分区结合的方式（如果merge这种分表方式，不能和分区配合的话，可以用其他的分表试），访问量不大，但是表数据很多的表，我们可以采取分区的方式等。
3.分表技术是比较麻烦的，需要手动去创建子表，app服务端读写时候需要计算子表名。采用merge好一些，但也要创建子表和配置子表间的union关系。
4.表分区相对于分表，操作方便，不需要创建子表。

## 分表的几种方式：

### MySQL集群

它并不是分表，但起到了和分表相同的作用。集群可分担数据库的操作次数，将任务分担到多台数据库上。集群可以读写分离，减少读写压力。从而提升数据库性能。

### 自定义规则分表

大表可以按照业务的规则来分解为多个子表。通常为以下几种类型，也可自己定义规则。

```bash
Range（范围）–这种模式允许将数据划分不同范围。例如可以将一个表通过年份划分成若干个分区。
Hash（哈希）–这中模式允许通过对表的一个或多个列的Hash Key进行计算，最后通过这个Hash码不同数值对应的数据区域进行分区。例如可以建立一个对表主键进行分区的表。
Key（键值）-上面Hash模式的一种延伸，这里的Hash Key是MySQL系统产生的。
List（预定义列表）–这种模式允许系统通过预定义的列表的值来对数据进行分割。
Composite（复合模式） –以上模式的组合使用　
```

分表规则与分区规则一样，在分区模块详细介绍。
下面以Range简单介绍下如何分表（按照年份表）。

假设表结构有4个字段：自增id，姓名，存款金额，存款日期

把存款日期作为规则分表，分别创建几个表

2011年：account_2011

2012年：account_2012

……

2015年：account_2015

app在读写的时候根据日期来查找对应的表名，需要手动来判定。

```javascript
var getTableName = function() {
var data = {
    name: 'tom',
    money: 2800.00,
    date: '201410013059'
};
var tablename = 'account_';
var year = parseInt(data.date.substring(0, 4));
if (year < 2012) {
    tablename += 2011; // account_2011
} else if (year < 2013) {
    tablename += 2012; // account_2012
} else if (year < 2014) {
    tablename += 2013; // account_2013
} else if (year < 2015) {
    tablename += 2014; // account_2014
} else {
    tablename += 2015; // account_2015
}
return tablename;
}
```
### 利用merge存储引擎来实现分表

merge分表，分为主表和子表，主表类似于一个壳子，逻辑上封装了子表，实际上数据都是存储在子表中的。

我们可以通过主表插入和查询数据，如果清楚分表规律，也可以直接操作子表。

```mysql
CREATE TABLE t1 (
	a INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
	message CHAR (20)
) ENGINE = myisam;

CREATE TABLE t2 (
	a INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
	message CHAR (20)
) ENGINE = myisam;

CREATE TABLE total (
	a INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
	message CHAR (20)
) ENGINE = MERGE
UNION
	= (t1, t2) INSERT_METHOD = LAST;
```



子表2011年

```mysql
CREATE TABLE account_2011 (
id  int(11) NOT NULL AUTO_INCREMENT ,
name  varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL ,
money  float NOT NULL ,
tradeDate  datetime NOT NULL
PRIMARY KEY (id)
)
ENGINE=MyISAM
DEFAULT CHARACTER SET=utf8 COLLATE=utf8_general_ci
AUTO_INCREMENT=2
CHECKSUM=0
ROW_FORMAT=DYNAMIC
DELAY_KEY_WRITE=0
;

```

子表2012年

```mysql
CREATE TABLE account_2012 (
id  int(11) NOT NULL AUTO_INCREMENT ,
name  varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL ,
money  float NOT NULL ,
tradeDate  datetime NOT NULL
PRIMARY KEY (id)
)
ENGINE=MyISAM
DEFAULT CHARACTER SET=utf8 COLLATE=utf8_general_ci
AUTO_INCREMENT=2
CHECKSUM=0
ROW_FORMAT=DYNAMIC
DELAY_KEY_WRITE=0
;
```

主表，所有年

```mysql
CREATE TABLE account_all (
id  int(11) NOT NULL AUTO_INCREMENT ,
name  varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULLDEFAULT NULL ,
money  float NOT NULL ,
tradeDate  datetime NOT NULL
PRIMARY KEY (id)
)
ENGINE=MRG_MYISAM
DEFAULT CHARACTER SET=utf8 COLLATE=utf8_general_ci
UNION=(account_2011,account_2012)
INSERT_METHOD=LAST
ROW_FORMAT=DYNAMIC
;
```

创建主表的时候有个INSERT_METHOD，指明插入方式，取值可以是：0 不允许插入；FIRST 插入到UNION中的第一个表； LAST 插入到UNION中的最后一个表。

通过主表查询的时候，相当于将所有子表合在一起查询。这样并不能体现分表的优势，建议还是查询子表。

## 分区的几种方式

### Range：

```mysql
create table range( 
　　id int(11), 
　　money int(11) unsigned not null, 
　　date datetime 
　　)partition by range(year(date))( 
　　partition p2007 values less than (2008), 
　　partition p2008 values less than (2009), 
　　partition p2009 values less than (2010) 
　　partition p2010 values less than maxvalue 
);
```

### List：

```mysql
create table list( 
　　a int(11), 
　　b int(11) 
　　)(partition by list (b) 
　　partition p0 values in (1,3,5,7,9), 
　　partition p1 values in (2,4,6,8,0) 
　);
```

### Hash：

```mysql
create table hash( 
　　a int(11), 
　　b datetime 
　　)partition by hash (YEAR(b) 
　　partitions 4;
```

### Key：

```mysql
create table t_key( 
　　a int(11), 
　　b datetime) 
　　partition by key (b) 
　　partitions 4;
```

## 分区管理

新增分区

````mysql
ALTER TABLE sale_data
ADD PARTITION (PARTITION p201010 VALUES LESS THAN (201011));
````

删除分区

```mysql
--当删除了一个分区，也同时删除了该分区中所有的数据。
ALTER TABLE sale_data DROP PARTITION p201010;
```

分区的合并
下面的SQL，将p201001 - p201009 合并为3个分区p2010Q1 - p2010Q3````

````mysql
ALTER TABLE sale_data
REORGANIZE PARTITION p201001,p201002,p201003,
p201004,p201005,p201006,
p201007,p201008,p201009 INTO
(
PARTITION p2010Q1 VALUES LESS THAN (201004),
PARTITION p2010Q2 VALUES LESS THAN (201007),
PARTITION p2010Q3 VALUES LESS THAN (201010)
);

````



