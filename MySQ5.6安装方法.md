## MySQL (免安装版本)安装

### 安装前准备

```bash
##创建mysql 用户
[root@shj-db01 ~]# useradd -s /sbin/nologin mysql
##安装依赖
[root@shj-db01 ~]# yum install gcc gcc-c++ glibc cmake ncurses-devel
#解压到 /home 目录
[root@shj-db01 ~]# tar -zxvf mysql-5.6.32-linux-glibc2.5-x86_64.tar.gz -C /home/
##简化文件夹名
[root@shj-db01 home]# mv ../mysql-5.6.32-linux-glibc2.5-x86_64/* ../mysql
[root@shj-db01 ~]# mkdir /home/mysql/run
[root@shj-db01 ~]# mkdir /home/mysql/data
[root@shj-db01 ~]# mkdir /home/mysql/logs
[root@shj-db01 ~]# chown mysql.mysql /home/mysql/run
[root@shj-db01 ~]# chown mysql.mysql /home/mysql/data

##需要安装perl
[root@shj-db01 ~]#yum -y install perl perl-devel
##初始化 data存放目录
[root@shj-db01 ~]#/home/mysql/scripts/mysql_install_db --basedir=/home/mysql --datadir=/home/mysql/data/ --user=mysql 
```

### My.cnf 配置文件

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

###############------------------查询优化需要配置,普通安装不需要
innodb_buffer_pool_size=10G ###设置 mysql innode 全局缓存
###############------------------查询优化需要配置,普通安装不需要

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

### 启动 mysql命令

```bash
[root@shj-db01 ~]# /home/mysql/bin/mysqld_safe --defaults-file=/home/mysql/my.cnf &
```



### 初始化密码

```bash

##------------------------------------方法一
[root@shj-db01 ~]# /home/mysql/bin/mysqladmin  -uroot  --socket=/home/mysql/run/mysql.sock password  'shjdb2016'

##不需要执行
##[root@shj-db01 ~]# /home/mysql/bin/mysqladmin -uroot -h 127.0.0.1 -P3306 password 'shjdb2016'
##-------------------------------------

##------------------------------------方法二
mysql>use mysql
mysql>update user set password=PASSWORD('shjdb2016') where User='root';
##-------------------------------------
```



### 启动关闭重启命令

```bash
#启动命令
[root@shj-db01 ~]# /home/mysql/bin/mysqld_safe --defaults-file=/home/mysql/my.cnf &

# 关闭命令
[root@shj-db01 ~]# /home/mysql/bin/mysqladmin -uroot -p'shjdb2016' --socket=/home/mysql/run/mysql.sock shutdown
  
#进入控制台
[root@shj-db01 ~]# /home/mysql/bin/mysql -uroot -p'shjdb2016' --socket=/home/mysql/run/mysql.sock
```

### 配置mysql远程访问

执行这个就可以了

```mysql
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'shjdb2016' WITH GRANT OPTION;
mysql>FLUSH PRIVILEGES;
```



其他方法:

*   改表。

```mysql
mysql>use mysql;
mysql>update user set host = '%' where user = 'root';
mysql>select host, user from user;
```

*   授权


```mysql
#赋予任何主机访问数据的权限
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
#使修改生效
mysql>FLUSH PRIVILEGES;
```

限定密码,你想root使用shjdb2016从任何主机连接到mysql服务器****

```mysql
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'shjdb2016' WITH GRANT OPTION;
mysql>FLUSH PRIVILEGES;
```

如果你想允许用户jack从ip为10.10.50.127的主机连接到mysql服务器，并使用654321作为密码

```mysql
mysql>GRANT ALL PRIVILEGES ON *.* TO 'jack'@'10.10.50.127' IDENTIFIED BY '654321' WITH GRANT OPTION;
mysql>FLUSH RIVILEGES   #这一句是刷新刚才的内容  #一定要刷新，因为操作的是系统授权表
```



### MySQL主从库配置

#### 主库分配从库账号

```mysql
####################主库
[root@shj-db01 ~]# /home/mysql/bin/mysql -u root -p'shjdb2016' --socket=/home/mysql/run/mysql.sock

mysql> grant super,replication slave on *.* to 'rep'@'192.168.1.%' identified by 'repdb2016'
```

#### 在主库查看 bin 日志和 post (很重要)

```mysql
mysql>show master status;
```

| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
| ---------------- | -------- | ------------ | ---------------- | ----------------- |
| mysql-bin.000005 |          | 6570         |                  |                   |

#### 从库配置同步

```bash
mysql> change master to
    -> master_user='rep',
    -> master_password='repdb2016',
    -> master_host='192.168.1.20',
    -> master_port=3306,
    -> master_log_file='mysql-bin.000001',
    -> master_log_pos=120;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql>  start slave;

Query OK, 0 rows affected (0.02 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Connecting to master
                  Master_Host: 192.168.1.121
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000005
          Read_Master_Log_Pos: 334
               Relay_Log_File: mysqld-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000005
             Slave_IO_Running: Connecting
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 334
              Relay_Log_Space: 120
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 2003
                Last_IO_Error: error connecting to master 'rep@192.168.1.121:3306' - retry-time: 60  retries: 1
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 0
                  Master_UUID: 
             Master_Info_File: /home/mysql/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 161217 00:32:38
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)

mysql> 
```

#### 主库防火墙配置,开放3306端口

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

#### 从库开启slave

```bash
#从库
mysql> stop slave;
Query OK, 0 rows affected (0.00 sec)
mysql> start slave;
Query OK, 0 rows affected (0.01 sec)
mysql> show slave stauts\G   #  可以查看是否成功
```

#### 常见错误:

*    Last_IO_Error: Fatal error: The slave I/O thread stops because master and slave have equal MySQL server ids; these ids must be different for replication to work (or the --replicate-same-server-id option must be used on slave but this does not always make sense; please check the manual before using it).

![img](file:///C:\Users\Administrator\Documents\Tencent Files\930624430\Image\C2C\Image1\MWT1@OV$N}ANN3N5[9]E1GP.png)



*   [root@localhost scripts]# The file /usr/local/mysql/bin/mysqld doesn't exist or is not executable

Please do a cd to the mysql installation directory and restart
this script from there as follows:
./bin/mysqld_safe.
See http://dev.mysql.com/doc/mysql/en/mysqld_safe.html for more
information

解决方法:

[root@localhost mysql5.0.45]# chown -R root .
[root@localhost mysql5.0.45]# chown -R mysql data
[root@localhost mysql5.0.45]# ./bin/mysqld_safe --user=mysql &
