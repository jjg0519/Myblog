# Redis 安装配置

## 安装包

```bash
redis-3.2.4.tar.gz
```

## 安装前依赖

```bash
yum -y install gcc-c++   #安装编译包
yum -y install gcc automake autoconf libtool make
```

## 安装步骤

```bash
tar -zxvf ./redis-3.2.4.tar.gz -C /usr/local/
cd /usr/local/redis-3.2.4/
make && make install
```

>注：make完成后，有产生可执行文件
>redis-server：redis服务器的启动程序
>redis-cli：redis命令行工具，也可为客户端
>redis-benchmark：redis性能测试工具（读写）
>redis-stat：redis状态检测工具（状态参数延迟）



## Redis的配置介绍

```bash
daemonize 如果需要在后台运行，把该项改为yes
pidfile 配置多个pid的地址，默认在/var/run/redis.pid
bind 绑定ip，设置后只接受自该ip的请求
port 监听端口，默认为6379
timeout 设置客户端连接时的超时时间，单位为秒
loglevel 分为4级，debug、verbose、notice、warning
logfile 配置log文件地址
databases 设置数据库的个数，默认使用的数据库为0
save 设置redis进行数据库镜像的频率，保存快照的频率，第一个表示多长时间，       第三个表示执行多少次写操作。在一定时间内执行一定数量的写操作时，自动保存快照。可设置多个条件。
rdbcompression 在进行镜像备份时，是否进行压缩
Dbfilename 镜像备份文件的文件名
Dir 数据库镜像备份的文件放置路径
Slaveof 设置数据库为其他数据库的从数据库 
Masterauth 主数据库连接需要的密码验证
Requirepass 设置登录时需要使用的密码
Maxclients 限制同时连接的客户数量
Maxmemory 设置redis能够使用的最大内存
Appendonly 开启append only模式
appendfsync 设置对appendonly.aof文件同步的频率
vm-enabled 是否虚拟内存的支持
vm-swap-file 设置虚拟内存的交换文件路径
vm-max-memory 设置redis使用的最大物理内存大小
vm-page-size 设置虚拟内存的页大小
vm-pages 设置交换文件的总page数量
vm-max-threads 设置VMIO同时使用的线程数量
glueoutputbuf 把小的输出缓存存放在一起
hash-max-zipmap-entries 设置hash的临界值
activerehashing 重新hash
```



## 创建单节点

### 创建配置文件

```bash
[root@own-server redis-3.2.4]# cd /usr/local/redis-3.2.4/
[root@own-server redis-3.2.4]# mkdir -p redis_cluster/7006
[root@own-server redis-3.2.4]# cp /usr/local/redis-3.2.4/redis.conf  ./redis_cluster/7006/
```

### 修改配置文件

7006文件夹中的文件修改对应的配置

```properties
bind 192.168.152.130
daemonize    yes                          //redis后台运行
pidfile  /var/run/redis_7006.pid          //pidfile文件对应7006
port  7006                                //端口7006
appendonly  yes                           //aof日志开启  有需要就开启，它会每次写操作都记录一条日志
```

### 启动关闭服务

```bash
[root@own-server local]# redis-server redis_cluster/7006/redis.conf
[root@own-server local]# /usr/local/redis-3.2.4/src/redis-cli -h 127.0.0.1 -p 7006 shutdown
#或者
[root@own-server local]# /usr/local/redis-3.2.4/src/redis-cli -h 192.168.152.130: -p 7006 shutdown
```

### 查看服务

```bash
[root@own-server local]#ps -ef | grep redis   #查看是否启动成功
[root@own-serverlocal]#netstat -tnlp | grep redis #可以看到redis监听端口
```

### 防火墙配置

```bash
####################################Centos 6
[root@own-server local]# vim /etc/sysconfig/iptables
#--------------------------------------------------#
-A INPUT -m state --state NEW -m tcp -p tcp --dport 7006  -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 17006  -j ACCEPT 
#--------------------------------------------------#
[root@own-server local]# service iptables  restart
[root@own-server local]# iptables -L -n  #查看开放的端口

####################################Centos 7
[root@own-server local]# firewall-cmd --zone=public --add-port=7006/tcp --permanent
[root@own-server local]# firewall-cmd --zone=public --add-port=17006/tcp --permanent
[root@own-server local]# firewall-cmd --reload   
#查看开放的端口
[root@own-server local]# firewall-cmd --zone=public --list-all 
```

## 创建多redis集群节点

### 机器分布

测试我们选择2台服务器，分别为：192.168.1.129 有3个节点，192.168.1.133.有3个节点。

### 创建和修改配置文件

  我先在192.168.1.129创建3个节点：

```bash
[root@shj-129 local]#cd /usr/local/
[root@shj-129 local]#mkdir redis_cluster  //创建集群目录
[root@shj-129 local]#cd /usr/local/redis_cluster
[root@shj-129 local]#mkdir 7000 7001 7002 7003  //分别代表三个节点    其对应端口 7000 7001 7002
#创建7000节点为例，拷贝到7000目录
[root@shj-129 local]#cp /usr/local/redis-3.2.4/redis.conf  ./redis_cluster/7000/   
#拷贝到7001目录
[root@shj-129 local]#cp /usr/local/redis-3.2.4/redis.conf  ./redis_cluster/7001/   
#拷贝到7002目录
[root@shj-129 local]#cp /usr/local/redis-3.2.4/redis.conf  ./redis_cluster/7002/   
```

   分别对7000,7001,7002,7003文件夹中的4个文件修改对应的配置

```properties
daemonize    yes                          //redis后台运行
pidfile  /var/run/redis_7000.pid          //pidfile文件对应7000,7002,7003
port  7000                                //端口7000,7002,7003
cluster-enabled  yes                      //开启集群  把注释#去掉
cluster-config-file  nodes_7000.conf      //集群的配置  配置文件首次启动自动生成 7000,7001,7002
cluster-node-timeout  5000                //请求超时  设置5秒够了
appendonly  yes                           //aof日志开启  有需要就开启，它会每次写操作都记录一条日志
```

   在192.168.1.133创建3个节点：对应的端口改为7003,7004,7005.配置对应的改一下就可以了。

### 启动两台机器各节点

```bash
#129 服务器
[root@shj-129 local]#cd /usr/local
[root@shj-129 local]#redis-server redis_cluster/7000/redis.conf
[root@shj-129 local]#redis-server redis_cluster/7001/redis.conf
[root@shj-129 local]#redis-server redis_cluster/7002/redis.conf


#133 服务器
[root@shj-133 local]#redis-server redis_cluster/7003/redis.conf
[root@shj-133 local]# redis-server redis_cluster/7004/redis.conf
[root@shj-133 local]# redis-server redis_cluster/7005/redis.conf
```

### 查看服务

```bash
[root@shj-129 local]#ps -ef | grep redis   #查看是否启动成功
[root@shj-129 local]#netstat -tnlp | grep redis #可以看到redis监听端口
```

### 集群配置

#### 安装前

前面已经准备好了搭建集群的redis节点，接下来我们要把这些节点都串连起来搭建集群。官方提供了一个工具

redis-trib.rb(/usr/local/redis-3.2.1/src/redis-trib.rb)

看后缀就知道这鸟东西不能直接执行，它是用ruby写的一个程序，所以我们还得安装ruby.

```bash
[root@shj-129 local]#yum -y install ruby ruby-devel rubygems rpm-build 
[root@shj-129 local]#yum -y install rubygems-devel
```

  再用 gem 这个命令来安装 redis接口    gem是ruby的一个工具包.

```bash
[root@shj-129 local]# gem install redis --version 3.0.0   #等一会儿就好了,可能会失败,1是因为rubygems-devel  没安装,2 是因为源无法访问
Successfully installed redis-3.3.2
1 gem installed
Installing ri documentation for redis-3.3.2...
Installing RDoc documentation for redis-3.3.2...
[root@shj-129 local]# 
```

**注意**

```bash
#如果rubygems-devel  安装了还是有问题,可以使用淘宝的源
#1 查看当前使用的源地址。
gem sources
#2 使用淘宝的一个镜像就可以安装redis了
gem sources -a https://ruby.taobao.org/
#3 更新源的缓存,更新源的缓存后即完成了Ruby的gem源修改。
gem sources -u

gem install redis

# 删除一个源
gem sources -r https://ruby.taobao.org/
```

当然，方便操作，两台Server都要安装。

上面的步骤完事了，接下来运行一下redis-trib.rb

```bash
[root@shj-129 local]#/usr/local/redis-3.2.4/src/redis-trib.rb
Usage: redis-trib <command> <options> <arguments ...>
reshard        host:port
	           --to <arg>
	           --yes
	           --slots <arg>
	           --from <arg>
check          host:port
call           host:port command arg arg .. arg
set-timeout    host:port milliseconds
add-node       new_host:new_port existing_host:existing_port
               --master-id <arg>
               --slave
del-node       host:port node_id
fix            host:port
import          host:port
               --from <arg>
help           (show this help)
create         host1:port1 ... hostN:portN
               --replicas <arg>
For check, fix, reshard, del-node, set-timeout you can specify the host and port of any working node in the cluster.
```

看到这，应该明白了吧， 就是靠上面这些操作 完成redis集群搭建的.

#### 防火墙配置

```bash
[root@shj-129 local]# vim /etc/sysconfig/iptables

#129 服务器
-A INPUT -m state --state NEW -m tcp -p tcp --dport 7000  -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 7001  -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 7002  -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 17000  -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 17001  -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 17002  -j ACCEPT 

#133 服务器
-A INPUT -m state --state NEW -m tcp -p tcp --dport 7003  -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 7004  -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 7005  -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 17003  -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 17004  -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 17005  -j ACCEPT 

[root@shj-129 local]# service iptables  restart
[root@shj-129 local]# iptables -L -n  #查看开放的端口
```

#### 创建集群

确认所有的节点都启动，接下来使用参数create 创建 (在192.168.1.129中来创建)

```bash
[root@shj-129 local]# /usr/local/redis-3.2.4/src/redis-trib.rb  create --replicas  1 192.168.1.129:7000 192.168.1.129:7001 192.168.1.129:7002 192.168.1.133:7003 192.168.1.133:7004 192.168.1.133:7005 
```

    解释下， --replicas 1  表示 自动为每一个master节点分配一个slave节点    上面有6个节点，程序会按照一定规则生成 3个master（主）3个slave(从)

 **前面已经提醒过的 防火墙一定要开放监听的端口，否则会创建失败**

 运行中，提示Can I set the above configuration? (type'yes' to accept): yes    //输入yes

 ---------------------------非必需 begin ---------------------------

接下来 提示  Waiting for the cluster tojoin..........  安装的时候在这里就一直等等等，没反应，傻傻等半天，看这句提示上面一句，Sending Cluster Meet Message to join the Cluster.

    这下明白了，我刚开始在一台Server上去配，也是不需要等的，这里还需要跑到Server2上做一些这样的操作。

    在192.168.1.238,redis-cli -c -p 700*  分别进入redis各节点的客户端命令窗口， 依次输入 cluster meet 192.168.1.238 7000……

```bash
[root@shj-129 local]#/usr/local/redis-3.2.4/src/redis-cli -h 192.168.1.133 -c -p 7003
192.168.1.133:7003>cluster meet 192.168.1.133 7003
```

 ---------------------------非必需 end---------------------------

#### 检验集群时候生效

    回到Server1，已经创建完毕了。

    查看一下

```bash
[root@shj-129 local]# /usr/local/redis-3.2.4/src/redis-trib.rb check 192.168.1.129:7000
>>> Performing Cluster Check (using node 192.168.1.129:7000)
M: 6bd3bf269af00685f6c8a33b9bb4cf4b4fc8715c 192.168.1.129:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: a31a3e4274595b23cd73d242c1d9627dfa754fa3 192.168.1.133:7004
   slots: (0 slots) slave
   replicates 6bd3bf269af00685f6c8a33b9bb4cf4b4fc8715c
M: 70661e369ca4d2a55cb5d9cc3a55ac43c628c57b 192.168.1.129:7001
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: c5fbe068baf3e50f80f34269b7f509b5f8731b4a 192.168.1.133:7003
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: dd649dc60b8ca93395a1fbf469f29c81849e9fb2 192.168.1.133:7005
   slots: (0 slots) slave
   replicates 70661e369ca4d2a55cb5d9cc3a55ac43c628c57b
S: fcc42323a209a049ed1fa23b5cb22d527f58a93c 192.168.1.129:7002
   slots: (0 slots) slave
   replicates c5fbe068baf3e50f80f34269b7f509b5f8731b4a
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

    到这里集群已经初步搭建好了。

#### 测试数据(未验证过)

- get 和 set数据

```bash
[root@shj-129 local]# redis-cli -c -p 7000
```

    进入命令窗口，直接 set hello  howareyou

    直接根据hash匹配切换到相应的slot的节点上。

    还是要说明一下，redis集群有16383个slot组成，通过分片分布到多个节点上，读写都发生在master节点。

- 假设测试

    果断先把192.168.1.238服务Down掉，（192.168.1.238有1个Master, 2个Slave） ,  跑回192.168.1.238, 查看一下 发生了什么事，192.168.1.237的3个节点全部都是Master，其他几个Server2的不见了

    测试一下，依然没有问题，集群依然能继续工作。

    原因：  redis集群  通过选举方式进行容错，保证一台Server挂了还能跑，这个选举是全部集群超过半数以上的Master发现其他Master挂了后，会将其他对应的Slave节点升级成Master.

    疑问： 要是挂的是192.168.1.237怎么办？    哥试了，cluster is down!!    没办法，超过半数挂了那救不了了，整个集群就无法工作了。 要是有三台Server，每台两Master，切记对应的主从节点

            不要放在一台Server,别问我为什么自己用脑子想想看，互相交叉配置主从，挂哪台也没事，你要说同时两台crash了，呵呵哒......

- 关于一致性

    我还没有这么大胆拿redis来做数据库持久化哥网站数据，只是拿来做cache，官网说的很清楚，Redis Cluster is not able to guarantee strong consistency. 

#### 集群常见错误

1. 配置完所有主节点后,报" ERR Invalid node address specified"

由于[Redis](http://lib.csdn.net/base/redis)-trib.rb 对域名或主机名支持不好,故在创建集群的时候要使用ip:port的方式

redis-trib.rb create ip1:port1 ip2:port2 ip3:port3

1. 创建集群时报某个err slot 0 is already busy (redis::commanderror)

这是由于之间创建集群没有成功,需要将nodes.conf和dir里面的文件全部删除(注意不要删除了redis.conf)

1. 创建集群时一直处于"Waiting for the cluster to join...................................."的状态

这个问题原因不知,但解决方法是在redis.conf文件中把bind 127.0.0.1本地环回口改为物理接口.

1. 安装ruby redis时长时间没响应,这是由于天朝网络,解决办法是改ruby源(请自行baidu)或手动安装


1. /usr/lib/ruby/gems/1.8/gems/redis-3.3.2/lib/redis/connection/ruby.rb:111:in `_write_to_socket': Connection timed out (Redis::TimeoutError)

   这个是因为安装的redis 版本太高了

   ```bash
   [root@shj-129 local]# gem list

   *** LOCAL GEMS ***

   redis (3.3.2)
   [root@shj-129 local]# gem uninstall redis --version 3.3.2
   Successfully uninstalled redis-3.3.2
   [root@shj-129 local]# gem install redis --version 3.0.0
   ```

2. redis集群 Waiting for the cluster to join 一直等待

   Redis集群创建执行

   ```bash
   /redis-trib.rb create --replicas 1 XXXX:PORT1 XXXX:PORT2 ....
   ```

   一直等待 Waiting for the cluster to join 很久都没有反应

   ```bash
   原因：
   redis集群不仅需要开通redis客户端连接的端口，而且需要开通集群总线端口
   集群总线端口为redis客户端连接的端口 + 10000
   如redis端口为6379
   则集群总线端口为16379
   故，所有服务器的点需要开通redis的客户端连接端口和集群总线端口
   注意：iptables 放开，如果有安全组，也要放开这两个端口

   #--------------------------------------------直接关防火墙最TM省事
   ```

### 启动关闭命令

查看一下启动的Redis实例

```bash
 ps -ef|grep redis
```

如果不是使用脚本启动则需要使用redis-cli  shutdown命令来停止

```bash
[root@shj-124 local]# pwd
/usr/local
##启动
[root@shj-124 local]# redis-server redis_cluster/7006/redis.conf
[root@shj-124 local]# ps -ef|grep redis
root      71359      1 99 14:08 ?        00:00:03 redis-server 192.168.1.124:7006           
root      71363  70150  0 14:08 pts/0    00:00:00 grep redis

[root@shj-129 local]# /usr/local/redis-3.2.4/src/redis-cli -h 127.0.0.1 -p 7003 shutdown
#或者
[root@shj-129 local]# /usr/local/redis-3.2.4/src/redis-cli -h 192.168.1.133 -p 7005 shutdown
```

