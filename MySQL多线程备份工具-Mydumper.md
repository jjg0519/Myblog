# MySQL多线程备份工具-Mydumper

Mydumper是一个针对MySQL和Drizzle的高性能多线程备份和恢复工具。开发人员主要来自MySQL,Facebook,SkySQL公司。目前已经在一些线上使用了Mydumper。

**mydumper是支持对一张表多个线程备份的，参数-r。**

## 特性

Mydumper主要特性：

- 轻量级C语言写的
- 执行速度比mysqldump快10倍
- 事务性和非事务性表一致的快照(适用于0.2.2以上版本)
- 快速的文件压缩
- 支持导出binlog
- 多线程恢复(适用于0.2.1以上版本)
- 以守护进程的工作方式，定时快照和连续二进制日志(适用于0.5.0以上版本)
- 开源 (GNU GPLv3)

## 安装

```bash
# yum install glib2-devel mysql-devel zlib-devel pcre-devel
# wget https://launchpad.net/mydumper/0.9/0.9.1/+download/mydumper-0.9.1.tar.gz
# tar zxvf  mydumper-0.9.1.tar.gz -C ../software/
# cmake .
# make
# make install
```

## 参数介绍

```bash
  -B, --database              需要备份的数据库，一个数据库一条命令备份，要不就是备份所有数据库，包括mysql。
  -T, --tables-list           需要备份的表，用逗号分隔。
  -o, --outputdir             备份文件目录
  -s, --statement-size        生成插入语句的字节数，默认1000000，这个参数不能太小，不然会报 Row bigger than statement_size for tools.t_serverinfo
  -r, --rows                  试图用行块来分割表，该参数关闭--chunk-filesize
  -F, --chunk-filesize        行块分割表的文件大小，单位是MB
  -c, --compress              压缩输出文件
  -e, --build-empty-files     即使表没有数据，也产生一个空文件
  -x, --regex                 正则表达式匹配，如'db.table'
  -i, --ignore-engines        忽略的存储引擎，用逗号分隔 
  -m, --no-schemas            不导出表结构
  -d, --no-data               不导出表数据
  -G, --triggers              导出触发器
  -E, --events                导出事件
  -R, --routines              导出存储过程
  -k, --no-locks              不执行共享读锁 警告：这将导致不一致的备份
  --less-locking              减到最小的锁在innodb表上.
  -l, --long-query-guard      设置长查询时间,默认60秒，超过该时间则会报错：There are queries in PROCESSLIST running longer than 60s, aborting dump
  -K, --kill-long-queries kill掉长时间执行的查询，备份报错：Lock wait timeout exceeded; try restarting transaction

  -D, --daemon                启用守护进程模式
  -I, --snapshot-interval     dump快照间隔时间，默认60s，需要在daemon模式下
  -L, --logfile               使用日志文件，默认标准输出到终端
  --tz-utc                    备份的时候允许备份Timestamp，这样会导致不同时区的备份还原会出问题，默认关闭，参数：--skip-tz-utc to disable.
  --skip-tz-utc               
  --use-savepoints            使用保存点记录元数据的锁信息，需要SUPER权限
  --success-on-1146           Not increment error count and Warning instead of Critical in case of table doesn't exist
  --lock-all-tables           锁全表，代替FLUSH TABLE WITH READ LOCK
  -U, --updated-since         Use Update_time to dump only tables updated in the last U days
  --trx-consistency-only      Transactional consistency only
  -h, --host                  The host to connect to
  -u, --user                  Username with privileges to run the dump
  -p, --password              User password
  -P, --port                  TCP/IP port to connect to
  -S, --socket                UNIX domain socket file to use for connection
  -t, --threads               备份执行的线程数,默认4个线程
  -C, --compress-protocol     在mysql连接上使用压缩协议
  -V, --version               Show the program version and exit
  -v, --verbose               更多输出, 0 = silent, 1 = errors, 2 = warnings, 3 = info, default 2
```

## 备份相关信息

mydumper输出文件
metadata:元数据 记录备份开始和结束时间，以及binlog日志文件位置。
table data:每个表一个文件
table schemas:表结构文件
binary logs: 启用--binlogs选项后，二进制文件存放在binlog_snapshot目录下
daemon mode:在这个模式下，有五个目录0，1，binlogs，binlog_snapshot，last_dump。
备份目录是0和1，间隔备份，如果mydumper因某种原因失败而仍然有一个好的快照，
当快照完成后，last_dump指向该备份。

metadata:元数据 记录备份开始和结束时间，以及binlog日志文件位置。
table data:每个表一个文件
table schemas:表结构文件
binary logs: 启用--binlogs选项后，二进制文件存放在binlog_snapshot目录下
daemon mode:在这个模式下，有五个目录0，1，binlogs，binlog_snapshot，last_dump。
备份目录是0和1，间隔备份，如果mydumper因某种原因失败而仍然有一个好的快照，
当快照完成后，last_dump指向该备份。

## 实践备份

```bash
###备份单个库  
mydumper -u leshami -p pwd -B sakila -o /tmp/bak
###备份所有数据库，全库备份期间除了information_schema与performance_schema之外的库都会被备份
mydumper -u leshami -p pwd -o /tmp/bak
###备份单表
mydumper -u leshami -p pwd -B sakila -T actor -o /tmp/bak
###备份多表
 mydumper -u leshami -p pwd -B sakila -T actor,city -o /tmp/bak
###当前目录自动生成备份日期时间文件夹,不指定-o参数及值时，如文件夹为：export-20150703-145806
mydumper -u leshami -p pwd -B sakila -T actor
###不带表结构备份表
mydumper -u leshami -p pwd -B sakila -T actor -m
###压缩备份及连接使用压缩协议(非本地备份时)
 mydumper -u leshami -p pwd -B sakila -o /tmp/bak -c -C
###备份特定表
mydumper -u leshami -p pwd -B sakila --regex=actor* -o /tmp/bak
###过滤特定库，如本来不备份mysql及test库
 mydumper -u leshami -p pwd -B sakila --regex '^(?!(mysql|test))' -o /tmp/bak
###基于空表产生表结构文件
mydumper -u leshami -p pwd -B sakila -T actor -e -o /tmp/bak
##设置长查询的上限，如果存在比这个还长的查询则退出mydumper，也可以设置杀掉这个长查询
mydumper -u leshami -p pwd -B sakila --long-query-guard 200 --kill-long-queries
###备份时输出详细日志
mydumper -u leshami -p pwd -B sakila -T actor -v 3 -o /tmp/bak
###导出binlog，使用-b参数，会自动在导出目录生成binlog_snapshot文件夹及binlog
 mydumper -u leshami -p pwd -P 3306 -b -o /tmp/bak

#指定备份数据库：备份abc、bcd、cde
mydumper -u backup -p 123456  -h 192.168.180.13 -P 3306 -t 3 -c -l 3600 -s 10000000 -e --regex 'abc|bcd|cde' -o bbb/

#指定不备份的数据库：不备份abc、mysql、test，备份其他数据库 
mydumper -u backup -p 123456  -h 192.168.180.13 -P 3306 -t 3 -c -l 3600 -s 10000000 -e --regex '^(?!(abc|mysql|test))' -o bbb/

```
![](image/20170504150715.png)

##还原实践

还原，表存在先删除（-o）：这里需要注意，使用该参数，备份目录里面需要有表结构的备份文件。



```bash
./myloader -u root -p 123456 -h 192.168.200.25 -P 3306 -o -B test -d /home/zhoujy/bak/

在“10.137.143.156”上对“10.137.143.151”的DB（TaeOss）进行备份
# mydumper -h 10.137.143.151 -u backup -p backup2015 -B TaeOss -t 8 -o /data/rocketzhang
 
3、将备份数据恢复到“10.137.143.152”
# myloader -h 10.137.143.152 -u backup -p backup2015 -B TaeOss -t 8 -o -d /data/rocketzhang
```



```mysql
SELECT * FROM information_schema.PROCESSLIST  WHERE COMMAND != "Sleep" order by TIME DESC;     
```

![](image/20170504145453.png)