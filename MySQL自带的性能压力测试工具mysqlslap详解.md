# MySQL自带的性能压力测试工具mysqlslap详解

## 简单介绍

mysqlslap是从5.1.4版开始的一个MySQL官方提供的压力测试工具。通过模拟多个并发客户端访问MySQL来执行压力测试，同时详细的提供了“高负荷攻击MySQL”的数据性能报告。并且能很好的对比多个存储引擎在相同环境下的并发压力性能差别。通过mysqlslap –help可以获得可用的选项，这里列一些主要的参数，更详细的说明参考[官方手册](http://dev.mysql.com/doc/refman/5.5/en/mysqlslap.html)。如果是系统自带或者使用rpm包安装的mysql，安装了MySQL-client端的包就有mysqlslap这个工具。

## mysqlslap运行阶段三步骤

- 创建 schema，table 和任何用来测试的已经存储了的程序和数据。这个阶段使用单客户端连接；
- 进行负载测试。这个阶段使用多客户端连接；
- 清除（断开连接，删除指定表）。这个阶段使用单客户端连接。

## 常见参数说明

```bash
--auto-generate-sql, -a 自动生成测试表和数据，表示用mysqlslap工具自己生成的SQL脚本来测试并发压力。
--auto-generate-sql-load-type=type 测试语句的类型。代表要测试的环境是读操作还是写操作还是两者混合的。取值包括：read，key，write，update和mixed(默认)。
--auto-generate-sql-add-auto-increment 代表对生成的表自动添加auto_increment列，从5.1.18版本开始支持。
--number-char-cols=N, -x N 自动生成的测试表中包含多少个字符类型的列，默认1
--number-int-cols=N, -y N 自动生成的测试表中包含多少个数字类型的列，默认1
--number-of-queries=N 代表总共要运行多少次查询。每个客户运行的查询数量可以用查询总数/并发数来计算。
--query=name,-q 使用自定义脚本执行测试，例如可以调用自定义的一个存储过程或者sql语句来执行测试。
--create-schema 代表自定义的测试库名称，测试的schema，MySQL中schema也就是database。
--commint=N 多少条DML后提交一次。
--compress, -C 如果服务器和客户端支持都压缩，则压缩信息传递。
--concurrency=N, -c N 表示并发量，也就是模拟多少个客户端同时执行select。可指定多个值，以逗号或者--delimiter参数指定的值做为分隔符。例如：--concurrency=100,200,500。
--engine=engine_name, -e engine_name 代表要测试的引擎，可以有多个，用分隔符隔开。例如：--engines=myisam,innodb。
--iterations=N, -i N 测试执行的迭代次数，代表要在不同并发环境下，各自运行测试多少次。
--only-print 只打印测试语句而不实际执行。
--detach=N 执行N条语句后断开重连。
--debug-info, -T 打印内存和CPU的相关信息。
  --create=name       File or string to use create tables.
```



> 说明：
>
> 测试的过程需要生成测试表，插入测试数据，这个mysqlslap可以自动生成，默认生成一个mysqlslap的schema，如果已经存在则先删除。可以用--only-print来打印实际的测试过程，整个测试完成后不会在数据库中留下痕迹。

## 各种测试参数实例

```bash
单线程测试。测试做了什么。
# mysqlslap -a -uroot -p123456
多线程测试。使用–concurrency来模拟并发连接。
# mysqlslap -a -c 100 -uroot -p123456
迭代测试。用于需要多次执行测试得到平均值。
# mysqlslap -a -i 10 -uroot -p123456

# mysqlslap ---auto-generate-sql-add-autoincrement -a -uroot -p123456
# mysqlslap -a --auto-generate-sql-load-type=read -uroot -p123456
# mysqlslap -a --auto-generate-secondary-indexes=3 -uroot -p123456
# mysqlslap -a --auto-generate-sql-write-number=1000 -uroot -p123456
# mysqlslap --create-schema world -q "select count(*) from City" -uroot -p123456
# mysqlslap -a -e innodb -uroot -p123456
# mysqlslap -a --number-of-queries=10 -uroot -p123456

测试同时不同的存储引擎的性能进行对比：
# mysqlslap -a --concurrency=50,100 --number-of-queries 1000 --iterations=5 --engine=myisam,innodb --debug-info -uroot -p123456

执行一次测试，分别50和100个并发，执行1000次总查询：
# mysqlslap -a --concurrency=50,100 --number-of-queries 1000 --debug-info -uroot -p123456

50和100个并发分别得到一次测试结果(Benchmark)，并发数越多，执行完所有查询的时间越长。为了准确起见，可以多迭代测试几次:
# mysqlslap -a --concurrency=50,100 --number-of-queries 1000 --iterations=5 --debug-info -uroot -p123456
```

## 完整测试例子

```bash
50,100,200 个并发分别得到一次测试结果
[root@shj-db01 ~]# /home/mysql/bin/mysqlslap --defaults-file=/home/mysql/my.cnf --concurrency=50,100,200 --iterations=1 --number-int-cols=4 --number-char-cols=35 --auto-generate-sql --auto-generate-sql-add-autoincrement --auto-generate-sql-load-type=mixed --engine=myisam,innodb --number-of-queries=200 --debug-info -uroot -p'123456'  --socket=/home/mysql/run/mysql.sock
Warning: Using a password on the command line interface can be insecure.
Benchmark
	Running for engine myisam
	Average number of seconds to run all queries: 0.015 seconds
	Minimum number of seconds to run all queries: 0.015 seconds
	Maximum number of seconds to run all queries: 0.015 seconds
	Number of clients running queries: 50
	Average number of queries per client: 4

Benchmark
	Running for engine myisam
	Average number of seconds to run all queries: 0.014 seconds
	Minimum number of seconds to run all queries: 0.014 seconds
	Maximum number of seconds to run all queries: 0.014 seconds
	Number of clients running queries: 100
	Average number of queries per client: 2

Benchmark
	Running for engine myisam
	Average number of seconds to run all queries: 0.024 seconds
	Minimum number of seconds to run all queries: 0.024 seconds
	Maximum number of seconds to run all queries: 0.024 seconds
	Number of clients running queries: 200
	Average number of queries per client: 1

Benchmark
	Running for engine innodb
	Average number of seconds to run all queries: 0.070 seconds
	Minimum number of seconds to run all queries: 0.070 seconds
	Maximum number of seconds to run all queries: 0.070 seconds
	Number of clients running queries: 50
	Average number of queries per client: 4

Benchmark
	Running for engine innodb
	Average number of seconds to run all queries: 0.018 seconds
	Minimum number of seconds to run all queries: 0.018 seconds
	Maximum number of seconds to run all queries: 0.018 seconds
	Number of clients running queries: 100
	Average number of queries per client: 2

Benchmark
	Running for engine innodb
	Average number of seconds to run all queries: 0.045 seconds
	Minimum number of seconds to run all queries: 0.045 seconds
	Maximum number of seconds to run all queries: 0.045 seconds
	Number of clients running queries: 200
	Average number of queries per client: 1


User time 0.02, System time 0.29
Maximum resident set size 65020, Integral resident set size 0
Non-physical pagefaults 5649, Physical pagefaults 0, Swaps 0
Blocks in 0 out 0, Messages in 0 out 0, Signals 0
Voluntary context switches 9585, Involuntary context switches 18
[root@shj-db01 ~]# 
```

解释:

对于INNODB引擎，200个客户端同时运行这些SQL语句平均要花0.045秒。相应的MYISAM为0.024秒。



## 测试自己定义的SQL脚本



测试时mysql的连接进程数：

```bash
[root@shj-db02 ~]# /home/mysql/bin/mysqlslap --defaults-file=/home/mysql/my.cnf --concurrency=50,100,200 --iterations=1000  --number-of-queries=10 --debug-info -uroot -p'shjdb2016'  --socket=/home/mysql/run/mysql.sock  --create-schema='test' --query='SELECT *,max(status_dic_id),count(*) as NUM FROM ykee_biz.cst_status_record where status_dic_id=274 GROUP BY track_id HAVING NUM>1 ORDER by NUM DESC;'
Warning: Using a password on the command line interface can be insecure.
Benchmark
	Average number of seconds to run all queries: 0.003 seconds
	Minimum number of seconds to run all queries: 0.001 seconds
	Maximum number of seconds to run all queries: 2.255 seconds
	Number of clients running queries: 50
	Average number of queries per client: 0

Benchmark
	Average number of seconds to run all queries: 0.003 seconds
	Minimum number of seconds to run all queries: 0.003 seconds
	Maximum number of seconds to run all queries: 0.007 seconds
	Number of clients running queries: 100
	Average number of queries per client: 0

Benchmark
	Average number of seconds to run all queries: 0.007 seconds
	Minimum number of seconds to run all queries: 0.006 seconds
	Maximum number of seconds to run all queries: 0.012 seconds
	Number of clients running queries: 200
	Average number of queries per client: 0


User time 34.47, System time 149.54
Maximum resident set size 89792, Integral resident set size 0
Non-physical pagefaults 1211882, Physical pagefaults 0, Swaps 0
Blocks in 0 out 2640, Messages in 0 out 0, Signals 0
Voluntary context switches 3402841, Involuntary context switches 7982
[root@shj-db02 ~]# 

```

