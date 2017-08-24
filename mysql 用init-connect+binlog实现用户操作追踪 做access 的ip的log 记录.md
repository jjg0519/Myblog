# mysql 用init-connect+binlog实现用户操作追踪

## 创建access 的ip的log 记录

**在MYSQL中，每个连接都会先执行init-connect，进行连接的初始化。我们可以在这里获取用户的登录名称和thread的ID值。**然后配合binlog，就可以追踪到每个操作语句的操作时间，操作人等。实现审计。实验过程：

* 创建登录日志库，登录日志表

```mysql
CREATE DATABASE accesslog;
USE accesslog;
CREATE TABLE accesslog 
(
  id int(11) NOT NULL AUTO_INCREMENT,
  thread_id int(11) DEFAULT NULL, #线程ID，这个值很重要
  log_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP, #登录时间
  localname varchar(30) DEFAULT NULL, #登录名称带IP
  matchname varchar(30) DEFAULT NULL, #登录用户，user的全称
  PRIMARY KEY (id)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

* 在配置文件中配置init-connect参数。登录时插入日志表。如果这个参数是个错误的SQL语句，登录就会失败。

**Linux 下的配置文件为 my.cnf,windows下位my.ini**

```
init-connect='insert into accesslog.accesslog values(null,connection_id(),now(),user(),current_user());'
```

**重启service mysqld 以使其配置文件生效**

* 创建普通用户，不能有super权限。**init-connect对具有super权限的用户不起作用。同时此用户必须要有INSERT权限，如果没有，登录后的任何操作都会导致MYSQL登录失败。**

```mysql
grant insert,select,update on . to 'user1'@'localhost'; #带INSERT权限
grant select,update on . to 'user2'@'localhost'; #不带INSERT权限
```
## 如何查看何跟踪用户行为记录。

```bash
如何查看何跟踪用户行为记录。
去mysql数据库服务器上查看binlog，应该thread_id=1的binlog记录。
[root@db_server binlog]# /usr/local/mysql/bin/mysqlbinlog  --base64-output=DECODE-ROWS  mysql-bin.000018 -v>3.log
[root@db_server binlog]# vim 3.log
# at 1103
#140728 20:12:48 server id 72  end_log_pos 1175 CRC32 0xa323c00e        Query   thread_id=1     exec_time=0     error_code=0
SET TIMESTAMP=1406549568/*!*/;
BEGIN
/*!*/;
# at 1175
#140728 20:12:48 server id 72  end_log_pos 1229 CRC32 0xbb8ca914        Table_map: `access_log`.`t1` mapped to number 72
# at 1229
#140728 20:12:48 server id 72  end_log_pos 1272 CRC32 0x8eed1450        Write_rows: table id 72 flags: STMT_END_F
### INSERT INTO `access_log`.`t1`
### SET
###   @1=10
###   @2='w0'
# at 1272
#140728 20:12:48 server id 72  end_log_pos 1303 CRC32 0x72b26336        Xid = 14
COMMIT/*!*/;

看到thread_id=1，然后，就可以根据thread_id=1来判断执行这条insert命令的来源，还可以在mysql服务器上执行show full processlist;来得到MySQL客户端的请求端口，
mysql> show full processlist;
+----+------------+-------------------+------+---------+------+-------+-----------------------+
| Id | User       | Host              | db   | Command | Time | State | Info                  |
+----+------------+-------------------+------+---------+------+-------+-----------------------+
|  1 | audit_user | 192.168.3.62:44657 | test | Sleep   |  162 |       | NULL                  |
|  3 | root       | localhost         | NULL | Query   |    0 | init  | show full processlist |
+----+------------+-------------------+------+---------+------+-------+-----------------------+
2 rows in set (0.00 sec)


mysql> 
看到Id为1的线程，端口是44657。

我们切换回mysql客户端，去查看端口是44657的是什么进程，如下所示：
[tim@db_client ~]$ netstat -antlp |grep 44657
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 192.168.3.62:44657           192.168.1.12:3307            ESTABLISHED 6335/mysql          
[tim@db_client ~]$ 

获取到该进程的PID 6335，再通过ps -eaf得到该进程所执行的命令，如下所示：
[tim@db_client ~]$ ps -eaf|grep 6335
tim   6335 25497  0 19:59 pts/1    00:00:00 mysql -uaudit_user -p -h 192.168.1.12 -P3307
tim   6993  6906  0 20:16 pts/2    00:00:00 grep 6335
[tim@db_client ~]$

最后查到是通过mysql客户端登陆连接的。加入这个6335是某个web工程的，那么，也可以根据ps -eaf命令查询得到web工程的进程信息。

```

