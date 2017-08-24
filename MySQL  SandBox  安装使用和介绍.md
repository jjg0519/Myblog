# MySQL  SandBox  安装使用和介绍

## 介绍

## 常用命令

make_sandbox : 最简单创建sandbox
low_level_make_sandbox : 创建单个sandbox，微调选项但不直接使用
make_replication_sandbox :创建master-slave架构
make_multiple_sandbox : 创建相同版本的sandbox
make_multiple_custom_sandbox :创建不同版本的sandbox
make_sandbox_from_source : 从build目录创建一个sandbox
make_sandbox_from_installed : 从已安装的二进制文件创建一个sandbox
sbtool : sandbox管理工具

## CPAN 安装方式

### 安装cpan

```bash
yum install cpan -y
```

### 安装软件依赖的包

```bash
yum install perl-Test-Simple -y
```

### 安装MySQL Sandbox 

```bash
cpan MySQL::Sandbox
```

### 设置环境变量(否则会抛错)

```bash
[root@localhost ~]# echo 'export SANDBOX_AS_ROOT=1' >> /root/.bash_profile
[root@localhost ~]# source /root/.bash_profile 
```



## 源码安装方式

```bash
# wget https://launchpad.net/mysql-sandbox/mysql-sandbox-3/mysql-sandbox-3/+download/MySQL-Sandbox-3.0.25.tar.gz
# tar zxvf MySQL-Sandbox-3.0.25.tar.gz -C ../software/
# perl Makefile.PL
# make
# make test
# make install
```



## Sanbox 安装MySQL

### 下载mysql二进制软件包

mysql-5.6.12-linux-glibc2.5-x86_64.tar.gz

### 创建mysql5.6(单个节点)实例

```bash
[root@localhost mysql]# cd /opt/mysql/
[root@localhost mysql]# make_sandbox mysql-5.6.12-linux-glibc2.5-x86_64.tar.gz
unpacking /opt/mysql/mysql-5.6.12-linux-glibc2.5-x86_64.tar.gz
Executing low_level_make_sandbox --basedir=/opt/mysql/5.6.12 \
        --sandbox_directory=msb_5_6_12 \
        --install_version=5.6 \
        --sandbox_port=5612 \
        --no_ver_after_name \
        --my_clause=log-error=msandbox.err
    The MySQL Sandbox,  version 3.0.44
    (C) 2006-2013 Giuseppe Maxia
installing with the following parameters:
upper_directory                = /root/sandboxes
sandbox_directory              = msb_5_6_12
sandbox_port                   = 5612
check_port                     = 
no_check_port                  = 
datadir_from                   = script
install_version                = 5.6
basedir                        = /opt/mysql/5.6.12
tmpdir                         = 
my_file                        = 
operating_system_user          = root
db_user                        = msandbox
remote_access                  = 127.%
bind_address                   = 127.0.0.1
ro_user                        = msandbox_ro
rw_user                        = msandbox_rw
repl_user                      = rsandbox
db_password                    = msandbox
repl_password                  = rsandbox
my_clause                      = log-error=msandbox.err
master                         = 
slaveof                        = 
high_performance               = 
prompt_prefix                  = mysql
prompt_body                    =  [\h] {\u} (\d) > 
force                          = 
no_ver_after_name              = 1
verbose                        = 
load_grants                    = 1
no_load_grants                 = 
no_run                         = 
no_show                        = 
do you agree? ([Y],n) 
#看见各种提示都给出了，相信童鞋们都看的懂，选择Y同意。
do you agree? ([Y],n) y
loading grants
.... sandbox server started
Your sandbox server was installed in $HOME/sandboxes/msb_5_6_12
[root@localhost mysql]# 
#最后会有安装路径的提示，默认在家目录下的sandboxes下。我们可以看看
[root@localhost sandboxes]# pwd
/root/sandboxes
[root@localhost sandboxes]# ll
total 40
-rwxr-xr-x. 1 root root   54 Jun  4 05:25 clear_all
drwxr-xr-x. 4 root root 4096 Jun  4 05:25 msb_5_6_12
-rw-r--r--. 1 root root 3621 Jun  4 05:25 plugin.conf
-rwxr-xr-x. 1 root root   56 Jun  4 05:25 restart_all
-rwxr-xr-x. 1 root root 2139 Jun  4 05:25 sandbox_action
-rwxr-xr-x. 1 root root   58 Jun  4 05:25 send_kill_all
-rwxr-xr-x. 1 root root   54 Jun  4 05:25 start_all
-rwxr-xr-x. 1 root root   55 Jun  4 05:25 status_all
-rwxr-xr-x. 1 root root   53 Jun  4 05:25 stop_all
-rwxr-xr-x. 1 root root   52 Jun  4 05:25 use_all
[root@localhost sandboxes]# 

#默认安装以后就启动了;
[root@localhost ~]# pgrep -fl mysql
2151 /bin/sh /opt/mysql/5.6.12/bin/mysqld_safe --defaults-file=/root/sandboxes/msb_5_6_12/my.sandbox.cnf
2331 /opt/mysql/5.6.12/bin/mysqld --defaults-file=/root/sandboxes/msb_5_6_12/my.sandbox.cnf --basedir=/opt/mysql/5.6.12 --datadir=/root/sandboxes/msb_5_6_12/data --plugin-dir=/opt/mysql/5.6.12/lib/plugin --user=root --log-error=/root/sandboxes/msb_5_6_12/data/msandbox.err --pid-file=/root/sandboxes/msb_5_6_12/data/mysql_sandbox5612.pid --socket=/tmp/mysql_sandbox5612.sock --port=5612
[root@localhost ~]# 
```

### 启动关闭登陆Sanbox 中的mysql

启动停止脚本在/root/sandboxes/msb_5_6_12

```bash
[root@own-server soft]# ll /root/sandboxes/msb_5_6_12/
total 108
-rwxr-xr-x. 1 root root 1329 Apr 12 18:06 add_option
-rwxr-xr-x. 1 root root 1807 Apr 12 18:06 change_paths
-rwxr-xr-x. 1 root root 1593 Apr 12 18:06 change_ports
-rwxr-xr-x. 1 root root 2950 Apr 12 18:06 clear
-rw-r--r--. 1 root root 2327 Apr 12 18:06 connection.json
drwx------. 5 root root 4096 Apr 12 18:11 data
-rw-r--r--. 1 root root  166 Apr 12 18:06 default_connection.json
-rw-r--r--. 1 root root 1215 Apr 12 18:06 grants_5_7_6.mysql
-rw-r--r--. 1 root root  905 Apr 12 18:06 grants.mysql
-rwxr-xr-x. 1 root root 1414 Apr 12 18:06 json_in_db
-rwxr-xr-x. 1 root root 2216 Apr 12 18:06 load_grants
-rwxr-xr-x. 1 root root 1521 Apr 12 18:06 msb
-rwxr-xr-x. 1 root root 1909 Apr 12 18:06 my
-rwxr-xr-x. 1 root root 1209 Apr 12 18:06 mycli
-rw-r--r--. 1 root root 1635 Apr 12 18:06 my.sandbox.cnf
-rwxr-xr-x. 1 root root 1541 Apr 12 18:06 mysqlsh
-rwxr-xr-x. 1 root root  923 Apr 12 18:06 proxy_start
-rw-r--r--. 1 root root 1140 Apr 12 18:06 README
-rwxr-xr-x. 1 root root  982 Apr 12 18:06 restart
-rwxr-xr-x. 1 root root 1767 Apr 12 18:06 send_kill
-rwxr-xr-x. 1 root root 2199 Apr 12 18:06 show_binlog
-rwxr-xr-x. 1 root root 2433 Apr 12 18:06 show_relaylog
-rwxr-xr-x. 1 root root 2752 Apr 12 18:06 start
-rwxr-xr-x. 1 root root 1252 Apr 12 18:06 status
-rwxr-xr-x. 1 root root 1844 Apr 12 18:06 stop
drwxr-xr-x. 2 root root    6 Apr 12 18:09 tmp
-rwxr-xr-x. 1 root root 1929 Apr 12 18:06 use
-rw-r--r--. 1 root root   86 Apr 12 18:06 USING
[root@own-server soft]# 


#start 启动mysql
#stop 关闭mysql
#use  登陆mysql控制台



```

### 查看Sandbox 中mysql 占用端口

```bash
#各个mysql实例的端口号就是mysql的版本号
[root@localhost msb_10_0_10]# netstat -nltp | grep mysqld
tcp        0      0 127.0.0.1:5612              0.0.0.0:*                   LISTEN      2547/mysqld   
tcp        0      0 127.0.0.1:10010             0.0.0.0:*                   LISTEN      3015/mysqld   

## 也可以用Sandbox 安装mysql 后自动生成的命令
[root@own-server rsandbox_mysql-5_6_12]# ./status_all 
REPLICATION rsandbox_mysql-5_6_12
master on
port: 19077
node1 on
port: 19078
node2 on
port: 19079
[root@own-server rsandbox_mysql-5_6
```

### 创建mysql5.6(多个节点)实例

对于需要部署多个实例呢？同一个版本部署多实例，其实也非常的简单，提供了make_multiple_sandbox这个命令，具体的参数自行help。或者查阅官方文档。
**默认部署3个实例，想要部署更多实例可以加参数--how_many_nodes = number，上面部署完成以后我们看看**

```bash
[root@mysql-server-01 mysql]# make_multiple_sandbox mariadb-10.0.12-linux-x86_64.tar.gz 
installing node 1
installing node 2
installing node 3
group directory installed in $SANDBOX_HOME/multi_msb_mariadb-10_0_12
[root@mysql-server-01 mysql]
[root@mysql-server-01 mysql]# ll /root/sandboxes/
total 52
-rwxr-xr-x. 1 root root   58 Apr 12 18:12 clear_all
drwxr-xr-x. 4 root root 4096 Apr 12 18:10 msb_5_6_12
drwxr-xr-x. 5 root root 4096 Apr 12 18:12 multi_msb_mysql-5_6_12
-rw-r--r--. 1 root root 3621 Apr 12 18:12 plugin.conf
-rwxr-xr-x. 1 root root   60 Apr 12 18:12 restart_all
-rwxr-xr-x. 1 root root 2145 Apr 12 18:12 sandbox_action
-rwxr-xr-x. 1 root root   62 Apr 12 18:12 send_kill_all
-rwxr-xr-x. 1 root root   58 Apr 12 18:12 start_all
-rwxr-xr-x. 1 root root   59 Apr 12 18:12 status_all
-rwxr-xr-x. 1 root root   57 Apr 12 18:12 stop_all
-rwxr-xr-x. 1 root root 4518 Apr 12 18:12 test_replication
-rwxr-xr-x. 1 root root   56 Apr 12 18:12 use_all
[root@mysql-server-01 mysql]# ll /root/sandboxes/multi_msb_mysql-5_6_12/
total 72
-rwxr-xr-x. 1 root root  523 Apr 12 18:12 check_slaves
-rwxr-xr-x. 1 root root  456 Apr 12 18:12 clear_all
-rw-r--r--. 1 root root 7778 Apr 12 18:12 connection.json
-rw-r--r--. 1 root root  626 Apr 12 18:12 default_connection.json
-rwxr-xr-x. 1 root root   54 Apr 12 18:12 n1
-rwxr-xr-x. 1 root root   54 Apr 12 18:12 n2
-rwxr-xr-x. 1 root root   54 Apr 12 18:12 n3
drwxr-xr-x. 4 root root 4096 Apr 12 18:11 node1
drwxr-xr-x. 4 root root 4096 Apr 12 18:12 node2
drwxr-xr-x. 4 root root 4096 Apr 12 18:12 node3
-rw-r--r--. 1 root root 1088 Apr 12 18:12 README
-rwxr-xr-x. 1 root root  216 Apr 12 18:12 restart_all
-rwxr-xr-x. 1 root root  484 Apr 12 18:12 send_kill_all
-rwxr-xr-x. 1 root root  456 Apr 12 18:12 start_all
-rwxr-xr-x. 1 root root  342 Apr 12 18:12 status_all
-rwxr-xr-x. 1 root root  449 Apr 12 18:12 stop_all
-rwxr-xr-x. 1 root root  321 Apr 12 18:12 use_all
[root@mysql-server-01 mysql]# 

[root@mysql-server-01 multi_msb_mysql-5_6_12]# ./n1
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 5
Server version: 10.0.12-MariaDB-log MariaDB Server

Copyright (c) 2000, 2014, Oracle, SkySQL Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

node1 [localhost] {msandbox} ((none)) > 
```

### 创建主从Master_slave(1Master -2Slave)

```bash
mysql-5.6.12-linux-glibc2.5-x86_64.tar.gz  mysql_backup.tar.gz
[root@own-server sandboxes]# make_replication_sandbox /home/soft/mysql-5.6.12-linux-glibc2.5-x86_64.tar.gz 
installing and starting master
installing slave 1
installing slave 2
starting slave 1
... sandbox server started
starting slave 2
.. sandbox server started
initializing slave 1
initializing slave 2
replication directory installed in $HOME/sandboxes/rsandbox_mysql-5_6_12
[root@own-server sandboxes]# pwd
/root/sandboxes
[root@own-server sandboxes]# ll -h
total 56K
-rwxr-xr-x. 1 root root   58 Apr 12 19:32 clear_all
drwxr-xr-x. 4 root root 4.0K Apr 12 18:10 msb_5_6_12
drwxr-xr-x. 5 root root 4.0K Apr 12 18:12 multi_msb_mysql-5_6_12
-rw-r--r--. 1 root root 3.6K Apr 12 19:32 plugin.conf
-rwxr-xr-x. 1 root root   60 Apr 12 19:32 restart_all
drwxr-xr-x. 5 root root 4.0K Apr 12 19:32 rsandbox_mysql-5_6_12
-rwxr-xr-x. 1 root root 2.1K Apr 12 19:32 sandbox_action
-rwxr-xr-x. 1 root root   62 Apr 12 19:32 send_kill_all
-rwxr-xr-x. 1 root root   58 Apr 12 19:32 start_all
-rwxr-xr-x. 1 root root   59 Apr 12 19:32 status_all
-rwxr-xr-x. 1 root root   57 Apr 12 19:32 stop_all
-rwxr-xr-x. 1 root root 4.5K Apr 12 19:32 test_replication
-rwxr-xr-x. 1 root root   56 Apr 12 19:32 use_all
[root@own-server sandboxes]# 

```





## 参考

[mysql sandbox 经典讲义]: doc/mysqlsandbox经典讲义.pdf	"mysql sandbox 经典讲义"