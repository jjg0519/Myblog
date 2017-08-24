# MySQL (免安装版本)安装

## 安装前准备

```bash
##创建mysql 用户
[root@shj-db01 ~]# useradd -s /sbin/nologin mysql
##安装依赖
[root@shj-db01 ~]# yum install gcc gcc-c++ glibc cmake ncurses-devel
#解压到 /home 目录
[root@shj-db01 ~]# tar -zxvf mysql-5.6.32-linux-glibc2.5-x86_64.tar.gz -C /home/
##简化文件夹名
[root@shj-db01 home]# mv mysql-5.6.32-linux-glibc2.5-x86_64/ mysql
[root@shj-db01 ~]# mkdir /home/mysql/run
[root@shj-db01 ~]# mkdir /home/mysql/data
[root@shj-db01 ~]# mkdir /home/mysql/logs
[root@shj-db01 ~]# chown mysql.mysql /home/mysql/run
[root@shj-db01 ~]# chown mysql.mysql /home/mysql/data

```

## 初始化MySQL

```bash
##初始化 data存放目录
[root@shj-db01 ~]#/home/mysql/scripts/mysql_install_db --basedir=/home/mysql --datadir=/home/mysql/data/ --user=mysql 
```

## My.cnf 配置文件

```properties
[client]
socket=/home/mysql/run/mysql.sock

[mysqld]
basedir=/home/mysql
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
# bin log 删除要用purge binary  logs 命令从文件系统删除这些文件,而不要用手工方式删除,这一点也非常重要,因为在数据库和文件系统存在内在的引用关系
[mysqld_safe]
log-error=/home/mysql/logs/mysqld.log ###log 日志路径
pid-file=/home/mysql/run/mysqld.pid ###设置 pid 路径
```

注意:从库安装参照上文即可。注意配置中的 server_id，并把 binlog 配置去掉。从库不需要
开启 binlog（开启 binlog 会消耗较大性能） log-bin,binlog_format

## 配置系统服务

```bash
##如果要配置多个mysql,可以 /etc/init.d/mysqld_3306
# cp -af /home/mysql/support-files/mysql.server /etc/init.d/mysqld
# vi /etc/init.d/mysqld
# 修改两处位置
basedir=/home/mysql
datadir=/home/mysql/data
socket=/home/mysql/run/mysql.sock   ##my.cnf 
# 执行如下命令
# chmod 755 /etc/init.d/mysqld
# chkconfig --add mysqld
# chkconfig --level 345 mysqld on
```

## 启动 mysql命令

```bash
[root@shj-db01 ~]# cd /home/mysql
[root@shj-db01 ~]# /home/mysql/bin/mysqld_safe --defaults-file=/home/mysql/my.cnf &

##如果配置成了系统服务可以通过下面启动
[root@shj-db01 ~]# service mysqld start
```

### ##初始化密码

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

### ##启动关闭重启命令

```bash
#启动命令
[root@shj-db01 ~]# /home/mysql/bin/mysqld_safe --defaults-file=/home/mysql/my.cnf &

# 关闭命令
[root@shj-db01 ~]# /home/mysql/bin/mysqladmin -uroot -p'123456' --socket=/home/mysql/run/mysql.sock shutdown
  
#进入控制台
[root@shj-db01 ~]# /home/mysql/bin/mysql -uroot -p'123456' --socket=/home/mysql/run/mysql.sock
```

## 配置mysql远程访问

- 改表。

```mysql
mysql>use mysql;
mysql>update user set host = '%' where user = 'root';
mysql>select host, user from user;
```

- 授权

限定密码,你想root使用shjdb2016从任何主机连接到mysql服务器(**)

```mysql
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
mysql>FLUSH PRIVILEGES;
```

如果你想允许用户jack从ip为10.10.50.127的主机连接到mysql服务器，并使用654321作为密码

```mysql
mysql>GRANT ALL PRIVILEGES ON *.* TO 'jack'@'10.10.50.127' IDENTIFIED BY '654321' WITH GRANT OPTION;
mysql>FLUSH RIVILEGES   #这一句是刷新刚才的内容  #一定要刷新，因为操作的是系统授权表
```

## 防火墙配置

```bash
[root@centos-server-6 mysql]# vim /etc/sysconfig/iptables
```



## 常用命令

```bash




mysqlbinlog --no-defaults --start-datetime="2016-11-17 00:00:00" --stop-datetime="2016-11-19 23:59:59" ./mysql-bin.000011 --result-file=mysql-bin.000011.sql
```

```mysql
show variables like 'log_bin';   # 查看是否开启日志
show master status;              # 查看日志状态
```



## 常见错误

###本地不需要密码访问,远程却需要密码

在本地MySQL中用grant新建一个用户并赋予权限，语句如下：

​	grant all privileges on whjc.* to whjc@"%" identified by "whjc";	然后新建连接用密码登录报“mysql 1045 access denied for user”错误，更奇怪的是用空密码反而能登录。

​	解决方法：	在MySQL user表中有一行Host为localhost，User和Password列都为空的记录，删除这条记录，然后flush privileges即可。正是因为有这条记录才导致上述问题。







# 安装MySQL5.6

## 安装必要组件

```bash
yum install –y autoconf automake imake libxml2-devel\
 expat-devel cmake gcc gcc-c++ libaio libaio-devel bzr bison libtool ncurses5-devel
```

## 下载解压MySQL

```bash
# cd /usr/local/src
# wget -c http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.14-linux-glibc2.5-x86_64.tar.gz/from/http://cdn.mysql.com/ -O mysql-5.6.14-linux-glibc2.5-x86_64.tar.gz
# tar zxvf mysql-5.6.14-linux-glibc2.5-x86_64.tar.gz –C ../
# cd /usr/local/
# ln -s mysql-5.6.14-linux-glibc2.5-x86_64 mysql
```

## 创建Mysql用户组和用户，及数据库存放目录

```bash
# mkdir -p /data/mysql_data_3306
# mkdir -p /data/mysql_log
# mkdir -p /data/log-bin
# groupadd mysql
# useradd mysql -g mysql -M -s /sbin/nologin
# chown -R mysql.mysql /data/mysql_data_3306 /data/mysql_log /data/log-bin
# chown -R mysql.mysql /usr/local/mysql-5.6.14-linux-glibc2.5-x86_64
```

## 修改配置文件my.cnf

vi /etc/my.cnf

```mysql
[mysqld]
# GENERAL #
user = mysql
default-storage-engine = InnoDB
socket = /data/mysql_data_3306/mysql.sock
pid-file = /data/mysql_data_3306/mysql.pid
port = 3306
# MyISAM #
key_buffer_size = 1344M
myisam_recover = FORCE,BACKUP
# SAFETY #
max_allowed_packet = 16M
max_connect_errors = 1000000
skip_name_resolve
# DATA STORAGE #
datadir = /data/mysql_data_3306/
long_query_time = 1
# BINARY LOGGING #
log-bin = /data/log-bin/mysql-bin-3306
expire-logs-days = 14
sync-binlog = 1
server-id = 1
max_binlog_size = 500M
# REPLICATION #
relay-log = /data/log-bin/relay-bin-3306
slave-net-timeout = 60
# CACHES AND LIMITS #
tmp_table_size = 32M
max_heap_table_size = 32M
max_connections = 500
thread_cache_size = 50
open_files_limit = 65535
table_definition_cache = 4096
table_open_cache = 4096
# INNODB #
innodb_data_file_path = ibdata1:128M;ibdata2:10M:autoextend
innodb_flush_method = O_DIRECT
innodb_log_files_in_group = 2
innodb_lock_wait_timeout = 50
innodb_log_file_size = 256M
innodb_flush_log_at_trx_commit = 1
innodb_file_per_table = 1  ##使用表空间
innodb_thread_concurrency = 8
innodb_buffer_pool_size = 8G
# LOGGING #
log-error = /data/mysql_log/mysql-error-3306.log
log-queries-not-using-indexes = 1
slow-query-log = 1
long_query_time = 1
slow-query-log-file = /data/mysql_log/mysql-slow-3306.log
# FOR SLAVE #
#binlog-format = ROW
#log-slave-updates = true
#gtid-mode = on
#enforce-gtid-consistency = true
#master-info-repository = TABLE
#relay-log-info-repository = TABLE
#sync-master-info = 1
#slave-parallel-workers = 2
#binlog-checksum = CRC32
#master-verify-checksum = 1
#slave-sql-verify-checksum = 1
#binlog-rows-query-log_events = 1
#report-port = 3306
#report-host = 10.1.1.10
```

## 配置系统服务

```bash
# cp -af /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld_3306
# vi /etc/init.d/mysqld_3306
修改两处位置：
basedir=/usr/local/mysql
datadir=/data/mysql_data_3306
 
执行如下命令
# chmod 755 /etc/init.d/mysqld_3306
# chkconfig --add mysqld_3306
# chkconfig --level 345 mysqld_3306 on
```

## 初始化数据库

```bash
# cd /usr/local/mysql
# ./scripts/mysql_install_db --user=mysql --defaults-file=/etc/my.cnf
```

## 启动数据库进程

```bash
service mysqld_3306 start
```

## 修改root密码

```bash
# /usr/local/mysql/bin/mysql -p -uroot -S /tmp/mysql.sock #这里直接回车就能进入数据库系统
Mysql> delete from mysql.user where user='';
Mysql> update mysql.user set password=PASSWORD(‘xxxxxxxx’) where user='root';
Mysql>flush privileges;
```



