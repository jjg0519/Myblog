# MySQL异步复制安装和配置

[TOC]

## MySQL复制同步类型

**异步复制（Asynchronous replication）**

MySQL默认的复制即是异步的，主库在执行完客户端提交的事务后会立即将结果返给给客户端，并不关心从库是否已经接收并处理，这样就会有一个问题，主如果crash掉了，此时主上已经提交的事务可能并没有传到从上，如果此时，强行将从提升为主，可能导致新主上的数据不完整。

**全同步复制（Fully synchronous replication）**

指当主库执行完一个事务，所有的从库都执行了该事务才返回给客户端。因为需要等待所有从库执行完该事务才能返回，所以全同步复制的性能必然会收到严重的影响。

**半同步复制（Semisynchronous replication）**

介于异步复制和全同步复制之间，主库在执行完客户端提交的事务后不是立刻返回给客户端，而是等待至少一个从库接收到并写到relay log中才返回给客户端。相对于异步复制，半同步复制提高了数据的安全性，同时它也造成了一定程度的延迟，这个延迟最少是一个TCP/IP往返的时间。所以，半同步复制最好在低延时的网络中使用。

## 主从MySQL安装和配置

### 安装MySQL(RPM方式)(通用)

```bash
# wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
# rpm -ivh mysql-community-release-el7-5.noarch.rpm
# yum install mysql-community-server


#安装成功后重启mysql服务。
# service mysqld restart
初次安装mysql，root账户没有密码。
##进入MySQL控制台
[root@yl-web yl]# mysql -u root 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.26 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.01 sec)
mysql> 
###设置密码
mysql> set password for 'root'@'localhost' =password('password');
Query OK, 0 rows affected (0.00 sec)

###或者 

mysql>use mysql
mysql>update user set password=PASSWORD('123456') where User='root';
mysql> grant all privileges on *.* to root@'%'identified by '123456';
```



### 安装MySQL(源代码方式)(通用)

```bash
##创建mysql 用户
[root@shj-db01 ~]# useradd -s /sbin/nologin mysql
##安装依赖
[root@shj-db01 ~]# yum install gcc gcc-c++ glibc cmake ncurses-devel

[root@shj-db01 ~]# yum install -y perl-Module-Install.noarch
#解压到 /home 目录
[root@shj-db01 ~]# tar -zxvf mysql-5.6.32-linux-glibc2.5-x86_64.tar.gz -C /home/
##简化文件夹名
[root@shj-db01 home]# mv mysql-5.6.32-linux-glibc2.5-x86_64/ mysql
[root@shj-db01 ~]# mkdir /home/mysql/run
[root@shj-db01 ~]# mkdir /home/mysql/data
[root@shj-db01 ~]# mkdir /home/mysql/logs
[root@shj-db01 ~]# chown mysql.mysql /home/mysql/run
[root@shj-db01 ~]# chown mysql.mysql /home/mysql/data

##初始化 data存放目录
[root@shj-db01 ~]#/home/mysql/scripts/mysql_install_db --basedir=/home/mysql --datadir=/home/mysql/data/ --user=mysql 
```

### Master My.cnf 配置文件(注意sync_binlog)

```properties
[client]
socket=/home/mysql/run/mysql.sock

[mysqld]
datadir=/home/mysql/data ###设置 mysql datadir
character_set_server=utf8
default-storage-engine=InnoDB ###设置 mysql 引擎
port=3306
socket=/home/mysql/run/mysql.sock ###设置 socket 存放路径
external-locking=FALSE
back_log=1024
max_connections=2000
table_open_cache = 200
max_allowed_packet = 24M
sort_buffer_size = 2M
read_buffer_size = 2M
join_buffer_size = 2M
read_rnd_buffer_size = 1M
thread_cache_size = 20
thread_concurrency = 8
thread_stack = 512k
query_cache_type = 2
query_cache_limit = 2M
query_cache_size= 512M
query_cache_min_res_unit= 2  #Try number of CPU's*2 for thread_concurrency
tmp_table_size = 256M
bulk_insert_buffer_size = 4M
skip-name-resolve
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
slow_query_log
long_query_time = 2
user=mysql
symbolic-links=0
wait_timeout=86400
innodb_buffer_pool_size=10G ###设置 mysql innode 全局缓存
lower_case_table_names=1 ###忽略大小写，使用 mycat 必须设置

###  这里注意和从库区别,从库 server_id 要和主库区分,从库不需要配置log-bin,binlog_format
log-bin=mysql-bin ###开启 bin 日志，设置主从，必须在主库开启
server-id=10 ###设置 server id，必须在主从库都开启，主从 id 需要不一致
binlog_format=mixed ### binlog 日志格式，mysql 默认采用 statement，建议使mixed

[mysqld_safe]
log-error=/home/mysql/logs/mysqld.log ###log 日志路径
pid-file=/home/mysql/run/mysqld.pid ###设置 pid 路径
```

注意:从库安装参照上文即可。注意配置中的 server_id，并把 binlog 配置去掉。从库不需要
开启 binlog（开启 binlog 会消耗较大性能） log-bin,binlog_format

### Slave My.cnf 配置文件(注意read-only和skip-slave-start)

````properties
[client]
socket=/home/mysql/run/mysql.sock

[mysqld]
#####replicate-do-db=mydb    ##复制的database
replicate-innore-db=test,mysql   ##不进行复制的复制的database
datadir=/home/mysql/data ###设置 mysql datadir
character_set_server=utf8
default-storage-engine=InnoDB ###设置 mysql 引擎
port=3306
socket=/home/mysql/run/mysql.sock ###设置 socket 存放路径
external-locking=FALSE
back_log=1024
max_connections=2000
table_open_cache = 200
max_allowed_packet = 24M
sort_buffer_size = 2M
read_buffer_size = 2M
join_buffer_size = 2M
read_rnd_buffer_size = 1M
thread_cache_size = 20
thread_concurrency = 8
thread_stack = 512k
query_cache_type = 2
query_cache_limit = 2M
query_cache_size= 512M
query_cache_min_res_unit= 2  #Try number of CPU's*2 for thread_concurrency
tmp_table_size = 256M
bulk_insert_buffer_size = 4M
skip-name-resolve
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
slow_query_log
long_query_time = 2
user=mysql
symbolic-links=0
wait_timeout=86400
innodb_buffer_pool_size=10G ###设置 mysql innode 全局缓存
lower_case_table_names=1 ###忽略大小写，使用 mycat 必须设置
read-only=1  			#可以防止没有super权限的用户在slave上进行写
skip-slave-start=1 		#防止slave自启动
###  这里注意和从库区别,从库 server_id 要和主库区分,从库不需要配置log-bin,binlog_format
server-id=10 ###设置 server id，必须在主从库都开启，主从 id 需要不一致


[mysqld_safe]
log-error=/home/mysql/logs/mysqld.log ###log 日志路径
pid-file=/home/mysql/run/mysqld.pid ###设置 pid 路径
````

### My.cnf 注意事项

首先 分别在master和slave上设置不同的server_id=1/101，启用master上的log-bin=1，启用slave上的relog-log=relay-bin; 在master上设置： binlog_format=row；二进制日志的格式。maser上最好还设置sync_binlog=1 和 innodb_flush_log_at_trx_commit=1防止发生服务器崩溃时 导致复制破坏。

>sync_binlog”：这个参数是对于MySQL系统来说是至关重要的，他不仅影响到Binlog对MySQL所带来的性能损耗，而且还影响到MySQL中数据的完整性。对于“sync_binlog”参数的各种设置的说明如下：
>
>sync_binlog=0，当事务提交之后，MySQL不做fsync之类的磁盘同步指令刷新binlog_cache中的信息到磁盘，而让Filesystem自行决定什么时候来做同步，或者cache满了之后才同步到磁盘。
>
>sync_binlog=n，当每进行n次事务提交之后，MySQL将进行一次fsync之类的磁盘同步指令来将binlog_cache中的数据强制写入磁盘。
>
>在MySQL中系统默认的设置是sync_binlog=0，也就是不做任何强制性的磁盘刷新指令，这时候的性能是最好的，但是风险也是最大的。因为一旦系统Crash，在binlog_cache中的所有binlog信息都会被丢失。而当设置为“1”的时候，是最安全但是性能损耗最大的设置。因为当设置为1的时候，即使系统Crash，也最多丢失binlog_cache中未完成的一个事务，对实际数据没有任何实质性影响。
>
>**从以往经验和相关测试来看，对于高并发事务的系统来说，“sync_binlog”设置为0和设置为1的系统写入性能差距可能高达5倍甚至更多。**

在slave上最好还配置：read-only=1 和 skip-slave-start=1 前者可以防止没有super权限的用户在slave上进行写，后者防止在启动 slave数据库时，自动启动复制线程。以后需要手动start slave来启动复制线程。注意slave没有必要启用 log-bin=1，除非需要搭建二级slave。

### 启动 mysql命令(通用)

```bash
[root@shj-db01 ~]# /home/mysql/bin/mysqld_safe --defaults-file=/home/mysql/my.cnf &
```



### 初始化密码(通用,Slave可不用)

```bash
##------------------------------------方法一
[root@shj-db01 ~]# /home/mysql/bin/mysqladmin  -uroot  --socket=/home/mysql/run/mysql.sock password  '123456'
[root@shj-db01 ~]# /home/mysql/bin/mysqladmin -uroot -h 127.0.0.1 -P3306 password '123456'
##-------------------------------------

##------------------------------------方法二
mysql>use mysql
mysql>update user set password=PASSWORD('123456') where User='root';
##-------------------------------------
```

### 启动关闭重启命令

```bash
#启动命令
[root@shj-db01 ~]# /home/mysql/bin/mysqld_safe --defaults-file=/home/mysql/my.cnf &

# 关闭命令
[root@shj-db01 ~]# /home/mysql/bin/mysqladmin -uroot -p'123456' --socket=/home/mysql/run/mysql.sock shutdown
  
#进入控制台
[root@shj-db01 ~]# /home/mysql/bin/mysql -uroot -p'123456' --socket=/home/mysql/run/mysql.sock
```

### 配置mysql远程访问

- 改表。

```mysql
mysql>use mysql;
mysql>update user set host = '%' where user = 'root';
mysql>select host, user from user;
```

- 授权

```mysql
#赋予任何主机访问数据的权限
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
#使修改生效
mysql>FLUSH PRIVILEGES;
```

限定密码,你想root使用123456从任何主机连接到mysql服务器(**)

```mysql
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
mysql>FLUSH PRIVILEGES;
```

如果你想允许用户jack从ip为10.10.50.127的主机连接到mysql服务器，并使用654321作为密码

```mysql
mysql>GRANT ALL PRIVILEGES ON *.* TO 'jack'@'10.10.50.127' IDENTIFIED BY '654321' WITH GRANT OPTION;
mysql>FLUSH RIVILEGES   #这一句是刷新刚才的内容  #一定要刷新，因为操作的是系统授权表
```

## 备份master上的数据库,迁移到slave上

```bash
###备份master上的数据库，迁移到slave上：
[root@localhost ~]# mysqldump -uroot -p --routines --flush-logs --master-data=2 --databases ykee_biz ykee_bid>/root/backup.sql
Enter password:

###排除Database|information_schema|mysql|test|performance_schema这几个库,批量备份

[root@localhost ~]# mysql -e "show databases;" -uroot -p| grep -Ev "Database|information_schema|mysql|test|performance_schema" | xargs mysqldump -uroot -p  --routines --flush-logs --master-data=2 --databases > mysql_dump.sql


##上传sql到从服务器
[root@localhost ~]# scp /root/backup.sql 192.168.137.9:/tmp/backup.sql
The authenticity of host '192.168.137.9 (192.168.137.9)' can't be established.
RSA key fingerprint is a4:cd:c0:13:d1:8c:c0:a5:e7:c4:43:b5:95:17:af:d3.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.137.9' (RSA) to the list of known hosts.
root@192.168.137.9's password:
```

>注意:
>
>因为slave的搭建需要一致性的备份，所以需要启用 --lock-all-tables(master-data=1/2会自动启用--lock-all-tables)或者--single-transaction；
>
>另外还需要知道该一致性备份的数据，对应的master上的binary log的文件名，以及在该文件中的position，所以必须启用 master-data选项。
>
>因为--master-data会启用--lock-all-tables 所以数据才是一致性的；但是导致了全局锁，不能进行任何修改操作；下面我们使用--single-transaction进行优化：
>
>mysqldump -uroot -p --routines --flush-logs --single-transaction --master-data=2 --databases db1 db2 > /root/backup.sql; (--flush-logs非必须)
>
>这样全局锁仅仅在备份的开始短暂的持有。不会再备份的整个过程中持有全局锁。

## Master分配Slave账号

```mysql
####################主库
[root@shj-db01 ~]# /home/mysql/bin/mysql -u root -p'123456' --socket=/home/mysql/run/mysql.sock

mysql> grant super,replication slave on *.* to 'rep'@'192.168.1.%' identified by 'repdb2016'

mysql> grant replication slave, replication client on *.* to repl@’192.168.%.%’ identified by ‘123456’
```

## Slave上执行备份的脚本,连上Master,开启复制线程

### slave 导入数据

```mysql
mysql> source /tmp/backup.sql
```

### 获取Master的binlogfile和position

#### 备份文件中找到 --master-data 输出的 binary log 的文件名和postion:

```bash
[root@localhost ~]# head -50 /tmp/backup.sql
-- MySQL dump 10.13  Distrib 5.6.32, for Linux (x86_64)
--
-- Host: localhost    Database: percona
-- ------------------------------------------------------
-- Server version	5.6.32-log
/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
--
-- Position to start replication or point-in-time recovery from
--
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000006', MASTER_LOG_POS=120;
--
-- Current Database: `percona`
--
```

#### 在主库查看 binlogfile 日志和 position

```mysql
mysql>show master status;
```

| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
| ---------------- | -------- | ------------ | ---------------- | ----------------- |
| mysql-bin.000005 |          | 6570         |                  |                   |

### 执行 change master to, start slave:

```mysql
mysql> change master to
    -> master_user='rep',
    -> master_password='repdb2016',
    -> master_host='192.168.1.20',
    -> master_port=3306,
    -> master_log_file='mysql-bin.000011',
    -> master_log_pos=88007609;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql>  start slave;
Query OK, 0 rows affected (0.02 sec)

mysql> show slave status\G
... ...
  Slave_IO_Running: Yes
  Slave_SQL_Running: Yes
......

#slave_IO_Runing 和 slave_sql_runing 状态都是yes表示搭建成功。

```

### 在Slave上查看复制线程的状态：

```mysql
mysql> show slave status\G
... ...
  Slave_IO_Running: Yes
  Slave_SQL_Running: Yes
......

#slave_IO_Runing 和 slave_sql_runing 状态都是yes表示搭建成功。
```

### 关闭重启Slave(注意临时表)

在重新启动Slave 的mysqld服务时，Stop Slave后，一定要检查**Slave_open_temp_tables** 这个状态值是否已经是0，如果不是，要重新start slave, 再stop slave，查看，直接是0后，才stop mysql。因为mysql重新启动后，在Slave上的所有临时表都没有了，这样重新进行复制时,后面还有对临时表的操作的binlog事件，因为Slave上的临时表已不存在，此时肯定会出错了。

```mysql
mysql> show  GLOBAL STATUS LIKE 'Slave_open_temp_tables';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| Slave_open_temp_tables | 0     |
+------------------------+-------+
1 row in set

mysql> 
```



## 主库防火墙配置,开放3306端口

```bash
[root@shj-db01 ~]# vim /etc/sysconfig/iptables
#------------------------------------------------------
		-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306  -j ACCEPT 
#------------------------------------------------------
# 重启防火墙
[root@shj-db01 soft]# service iptables restart
iptables：将链设置为政策 ACCEPT：filter                    [确定]
iptables：清除防火墙规则：                                 [确定]
iptables：正在卸载模块：                                   [确定]
iptables：应用防火墙规则：       						  [确定]

## 检验防火墙是否成功
[root@shj-db02 mysql]# yum install telnet
[root@shj-db02 mysql]# telnet 192.168.1.121 3306
Trying 192.168.1.121...
Connected to 192.168.1.121.
Escape character is '^]'.
N
5.6.32-lo_Xz})M*(!,AEt9\H1YO_qmysql_native_passwordConnection closed by foreign host.
```

## 主从同步一致性检查和修复

### -table-checksum

percona-toolkit系列工具中的一个， 可以用来检测主、 从[数据库](http://lib.csdn.net/base/mysql)中数据的一致性。

#### 创建checksums用户

```mysql
mysql> GRANT SELECT, PROCESS, SUPER, REPLICATION SLAVE ON *.* TO 'checksums'@'192.168.1.20' IDENTIFIED BY '123456';
mysql> grant all privileges on percona.* TO 'checksums'@'%' IDENTIFIED BY '123456';
```

#### pt-table-checksum检查一致性

```bash

#检查ykee_biz库主从一致,并把信息保存在percona.ykee_biz
[root@mysql01 soft]# pt-table-checksum --nocheck-replication-filters --replicate=percona.ykee_biz --databases=ykee_biz  h='192.168.1.20',u='checksums',p='123456',P=3306 
TS ERRORS  DIFFS     ROWS  CHUNKS SKIPPED    TIME TABLE
04-04T11:11:19      0      1   226002       8       0   2.916 ykee_biz.cst_customer
    
#DIFFS=1 表示有主从不一致的情况
```

### t-table-sync







## 优化和常见问题

### mysql之slave_skip_errors选项

要说slave_skip_errors选项，就不得不提mysql的replication机制，总的来说它分了三步来实现mysql主从库的同步

1. master将改变记录到二进制日志(binary log)中（这些记录叫做二进制日志事件，binary log events）；
2. slave将master的binary log events拷贝到它的中继日志(relay log)；
3. slave重做中继日志中的事件，将改变反映它自己的数据。

但是在主从同步中会出现因为从库执行某些sql语句失败而导致主从备份关系失效，如果要修复这种失效就需要用到slave_skip_errors参数（使用sql_skip_errors_counter也是可以的）。

slave_skip_errors选项有四个可用值，分别为：off、all、ErorCode、ddl_exist_errors。

![img](http://images.cnitblog.com/blog/483444/201409/051654221578928.png)

根据各个值得字面意思即可知道它们的用法，但是其中ddl_exist_errors值却比较特别，它代表了一组errorCode的组合，分别是：

1007：数据库已存在，创建数据库失败
1008：数据库不存在，删除数据库失败
1050：数据表已存在，创建数据表失败
1050：数据表不存在，删除数据表失败
1054：字段不存在，或程序文件跟数据库有冲突
1060：字段重复，导致无法插入
1061：重复键名
1068：定义了多个主键
1094：位置线程ID
1146：数据表缺失，请恢复数据库

但是还要注意的是，该值只在mysql cluster版的mysqld中才可用，而在mysql Server版的mysqld中不可用。