# Zookeeper安装配置

## 单节点安装 Dubbo 注 册中心(Zookeeper-3.4.6)

### 修改host文件

```bash
[root@centos-server ~]# vim /etc/hosts 
#---------------------------------------------------------------------------------
    # zookeeper servers
    192.168.3.71 edu-provider-01
#---------------------------------------------------------------------------------
```

### 下载安装

```bash
#到 http://apache.fayea.com/zookeeper/下载 zookeeper-3.4.6：
[root@centos-server ~]# wget http://apache.fayea.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
解压 zookeeper 安装包：
[root@centos-server ~]# tar -zxvf zookeeper-3.4.6.tar.gz
# 在/home/soft/zookeeper-3.4.6 目录下创建以下目录：
[root@centos-server ~]# cd /home/soft/zookeeper-3.4.6
[root@centos-server ~]# mkdir data
[root@centos-server ~]# mkdir logs
```

### 修改 zoo.cfg 配置文件：

```bash
## 将zookeeper-3.4.6/conf 目录下的 zoo_sample.cfg 文件拷贝一份，命名为为zoo.cfg
[root@centos-server ~]# cp zoo_sample.cfg zoo.cfg
## 修改配置文件
[root@centos-server ~]# vi zoo.cfg
#--------------------------------------------------------
      # The number of milliseconds of each tick
      tickTime=2000
      # The number of ticks that the initial
      # synchronization phase can take
      initLimit=10
      # The number of ticks that can pass between
      # sending a request and getting an acknowledgement
      syncLimit=5
      # the directory where the snapshot is stored.
      # do not use /tmp for storage, /tmp here is just
      # example sakes.
      dataDir=/home/wusc/zookeeper-3.4.6/data
      dataLogDir=/home/wusc/zookeeper-3.4.6/logs
      # the port at which the clients will connect
      clientPort=2181
      #2888,3888 are election port
      server.1=edu-provider-01:2888:3888
#--------------------------------------------------------
```

其中，
2888 端口号是 zookeeper 服务之间通信的端口。
3888 是 zookeeper 与其他应用程序通信的端口。
edu-provider-01 是在 hosts 中已映射了 IP 的主机名。
initLimit：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。
当已经超过 10 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是5*2000=10 秒。
syncLimit：这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 2*2000=4秒。
server.A=B:C:D：其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 IP 地址或/etc/hosts 文件中映射了 IP 的主机名；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。

### 编辑 myid 文件

在 dataDir=/home/soft/zookeeper-3.4.6/data 下创建 myid 文件编辑 myid 文件，并在对应的 IP 的机器上输入对应的编号。如在 zookeeper 上，myid文件内容就是 1。如果只在单点上进行安装配置，那么只有一个 server.1

```bash
[root@centos-server ~]# vi myid
#---------------------------------------------------------------------------------
1
#---------------------------------------------------------------------------------
```

### 修改系统变量

```bash
#用户下修改 vi /home/wusc/.bash_profile，增加 zookeeper 配置：
#---------------------------------------------------------------------------------
# zookeeper env
export ZOOKEEPER_HOME=/home/wusc/zookeeper-3.4.6
export PATH=$ZOOKEEPER_HOME/bin:$PATH
#---------------------------------------------------------------------------------
#使配置文件生效
[root@centos-server ~]# source /home/wusc/.bash_profile
```

### 修改防火墙端口

在防火墙中打开要用到的端口 2181、2888、3888切换到 root 用户权限，执行以下命令

```bash

[root@centos-server ~]# chkconfig iptables on
[root@centos-server ~]# service iptables start
#编辑/etc/sysconfig/iptables
[root@centos-server ~]# vi /etc/sysconfig/iptables
#---------------------------------------------------------------------------------
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2181 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2888 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3888 -j ACCEPT
#---------------------------------------------------------------------------------
#重启防火墙：
[root@centos-server ~]# service iptables restart
#查看防火墙端口状态：
[root@centos-server ~]# service iptables status
Table: filter
Chain INPUT (policy ACCEPT)
num target prot opt source destination
1 ACCEPT all -- 0.0.0.0/0 0.0.0.0/0 state RELATED,ESTABLISHED
2 ACCEPT icmp -- 0.0.0.0/0 0.0.0.0/0
3 ACCEPT all -- 0.0.0.0/0 0.0.0.0/0
4 ACCEPT tcp -- 0.0.0.0/0 0.0.0.0/0 state NEW tcp dpt:22
5 ACCEPT tcp -- 0.0.0.0/0 0.0.0.0/0 state NEW tcp dpt:2181
6 ACCEPT tcp -- 0.0.0.0/0 0.0.0.0/0 state NEW tcp dpt:2888
7 ACCEPT tcp -- 0.0.0.0/0 0.0.0.0/0 state NEW tcp dpt:3888
8 REJECT all -- 0.0.0.0/0 0.0.0.0/0 reject-with icmp-host-prohibited
Chain FORWARD (policy ACCEPT)
num target prot opt source destination
1 REJECT all -- 0.0.0.0/0 0.0.0.0/0 reject-with icmp-host-prohibited
Chain OUTPUT (policy ACCEPT)
num target prot opt source destination
```

### 启动和测试

```bash
启动并测试 zookeeper（要用 soft 用户启动，不要用 root）:
(1) 使用 methew 用户到/home/methew/zookeeper-3.4.6/bin 目录中执行：
[root@centos-server ~]# zkServer.sh start
(2) 输入 jps 命令查看进程：
[root@centos-server ~]# jps
1456 QuorumPeerMain
1475 Jps
其中，QuorumPeerMain 是 zookeeper 进程，启动正常
```

### 常用命令

```bash
#查看状态：
[root@centos-server ~]# zkServer.sh status
#查看 zookeeper 服务输出信息：
由于服务信息输出文件在/home/soft/zookeeper-3.4.6/bin/zookeeper.out
[root@centos-server ~]# tail -500f zookeeper.out
# 停止 zookeeper 进程：
[root@centos-server ~]# zkServer.sh stop
```

### 设置开机启动

```bash
#-----------------------------------------Centos 7 启用rc.local ---------------
#给 /etc/rc.d/rc.local 可执行权限：
chmod +x /etc/rc.d/rc.local
#开启 rc-local.service 服务：
#systemctl   enable   rc-local.service
#systemctl   start  rc-local.service

#还是不行的话

直接在/etc/rc.local文件中加入：source /etc/profile
#-----------------------------------------------------------------------

#其他版本

#配置 zookeeper 开机使用 root 用户启动：
[root@centos-server ~]# vim /etc/rc.local
[root@centos-server ~]# su -root -c '/home/soft/zookeeper-3.4.6/bin/zkServer.sh start'
```

或者

```bash
#以root用户登录系统： 
#进入init.d文件夹
cd /etc/init.d/ 
#创建并打开zookeeper文件
vi zookeeper
# zookeeper文件如下　
#---------------------------------------------------
    #!/bin/bash
    #chkconfig:2345 20 90
    #description:zookeeper
    #processname:zookeeper
    export JAVA_HOME=/home/soft/jdk1.7.0_79
    export PATH=$JAVA_HOME/bin:$PATH
    case $1 in
             start) su root /home/soft/zookeeper-3.4.6/bin/zkServer.sh start;;
             stop) su root /home/soft/zookeeper-3.4.6/bin/zkServer.sh stop;;
             status) su root /home/soft/zookeeper-3.4.6/bin/zkServer.sh status;;
             restart) su root /home/soft/zookeeper-3.4.6/bin/zkServer.shrestart;;
             *)  echo "requirestart|stop|status|restart"  ;;
    esac
#---------------------------------------------------
#加权限，把 zookeeper修改为可运行的文件,命令参考如下：
 chmod +x zookeeper  
#使用chkconfig命令把 zookeeper命令加入到系统启动队列中： 
chkconfig --add zookeeper
#使用chkconfig命令把 zookeeper命令移除系统启动队列
chkconfig --del zookeeper

#查看zookeeper的状态： 
chkconfig --list zookeeper
#测试
service zookeeper start
service zookeeper stop
service zookeeper restart
service zookeeper status
```



