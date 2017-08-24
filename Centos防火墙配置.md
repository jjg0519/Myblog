



# Centos 防火墙配置



## Centos 6 iptables 防火墙配置

```bash
##编辑防火墙配置
vi /etc/sysconfig/iptables 
##通过/etc/init.d/iptables status命令查询是否有打开80端口，如果没有可通过两种方式处理： 
##修改vi /etc/sysconfig/iptables命令添加使防火墙开放80端口 
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT 
##关闭/开启/重启防火墙 
/etc/init.d/iptables stop #start 开启 #restart 重启 
##永久性关闭防火墙 
chkconfig --level 35 iptables off /etc/init.d/iptables stop iptables -P INPUT DROP 
```



## Centos 7 Firewall防火墙配置

### 安装

安装它，只需

```bash
yum install firewalld
```

如果需要图形界面的话，则再安装

```bash
yum install firewall-config
```

Firewall 能将不同的网络连接归类到不同的信任级别，Zone 提供了以下几个级别 
drop: 丢弃所有进入的包，而不给出任何响应 
block: 拒绝所有外部发起的连接，允许内部发起的连接 
public: 允许指定的进入连接 
external: 同上，对伪装的进入连接，一般用于路由转发 
dmz: 允许受限制的进入连接 
work: 允许受信任的计算机被限制的进入连接，类似 workgroup 
home: 同上，类似 homegroup 
internal: 同上，范围针对所有互联网用户 
trusted: 信任所有连接

### 常用命令

```bash
systemctl start firewalld         # 启动,
systemctl enable firewalld        # 开机启动
systemctl stop firewalld          # 关闭
systemctl disable firewalld       # 取消开机启动
firewall-cmd --state                           ##查看防火墙状态，是否是running
firewall-cmd --reload                          ##重新载入配置，比如添加规则之后，需要执行此命令
firewall-cmd --get-zones                       ##列出支持的zone
firewall-cmd --get-services                    ##列出支持的服务，在列表中的服务是放行的
firewall-cmd --query-service ftp               ##查看ftp服务是否支持，返回yes或者no
firewall-cmd --add-service=ftp                 ##临时开放ftp服务
firewall-cmd --add-service=ftp --permanent     ##永久开放ftp服务
firewall-cmd --remove-service=ftp --permanent  ##永久移除ftp服务
firewall-cmd --add-port=80/tcp --permanent     ##永久添加80端口 
firewall-cmd --zone=public --add-port=3306/tcp --permanent
iptables -L -n                                 ##查看规则，这个命令是和iptables的相同的
```

具体的规则管理，可以使用 firewall-cmd，具体的使用方法可以

```
firewall-cmd --help
```

查看运行状态

```
firewall-cmd --state
```

查看已被激活的 Zone 信息

```
firewall-cmd --get-active-zones
public
interfaces: eth0 eth1123
```

查看指定接口的 Zone 信息

```
firewall-cmd --get-zone-of-interface=eth0
public12
```

查看指定级别的接口

```
firewall-cmd --zone=public --list-interfaces
eth012
```

查看指定级别的所有信息，譬如 public

```
firewall-cmd --zone=public --list-all
public (default, active)
interfaces: eth0
sources:
services: dhcpv6-client http ssh
ports:
masquerade: no
forward-ports:
icmp-blocks:
rich rules:12345678910
```

查看所有级别被允许的信息

```
firewall-cmd --get-service1
```

查看重启后所有 Zones 级别中被允许的服务，即永久放行的服务

```
firewall-cmd --get-service --permanent1
```

**管理规则**

```
firewall-cmd --panic-on           # 丢弃
firewall-cmd --panic-off          # 取消丢弃
firewall-cmd --query-panic        # 查看丢弃状态
firewall-cmd --reload             # 更新规则，不重启服务
firewall-cmd --complete-reload    # 更新规则，重启服务12345
```

添加某接口至某信任等级，譬如添加 eth0 至 public，再永久生效

```bash
firewall-cmd --zone=public --add-interface=eth0 --permanent
#使用这些命令来永久打开一个新端口（如TCP/80）。
$firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

设置 public 为默认的信任级别

```
firewall-cmd --set-default-zone=public
```

**管理端口** 
列出 dmz 级别的被允许的进入端口

```
firewall-cmd --zone=dmz --list-ports
```

允许 tcp 端口 8080 至 dmz 级别

```
firewall-cmd --zone=dmz --add-port=8080/tcp
firewall-cmd --reload
```

允许某范围的 udp 端口至 public 级别，并永久生效

```
firewall-cmd --zome=public --add-port=5060-5059/udp --permanent1
```

**管理服务** 
添加 smtp 服务至 work zone

```
firewall-cmd --zone=work --add-service=smtp1
```

移除 work zone 中的 smtp 服务

```
firewall-cmd --zone=work --remove-service=smtp1
```

**配置 ip 地址伪装** 
查看

```
firewall-cmd --zone=external --query-masquerade1
```

打开伪装

```
firewall-cmd --zone=external --add-masquerade1
```

关闭伪装

```
firewall-cmd --zone=external --remove-masquerade1
```

**端口转发** 
要打开端口转发，则需要先

```
firewall-cmd --zone=external --add-masquerade1
```

然后转发 tcp 22 端口至 3753

```
firewall-cmd --zone=external --add-forward-port=port=22:proto=tcp:toport=37531
```

转发 22 端口数据至另一个 ip 的相同端口上

```
firewall-cmd --zone=external --add-forward-port=port=22:proto=tcp:toaddr=192.168.1.1001
```

转发 22 端口数据至另一 ip 的 2055 端口上

```
firewall-cmd --zone=external --add-forward-port=port=22:proto=tcp:toport=2055:toaddr=192.168.1.100
```

## CentOS7修改防火墙firewall为iptables

由于习惯了用iptables作为防火墙，所以在安装好centos7系统后，会将默认的firewall关闭，并另安装iptables进行防火墙规则设定

下面介绍centos7关闭firewall安装iptables，并且开启80端口、3306端口的操作记录：

```bash
[root@localhost ~]# cat /etc/redhat-release 

CentOS Linux release 7.2.1511 (Core)
##关闭firewall：
[root@localhost ~]# systemctl stop firewalld.service            #停止firewall
[root@localhost ~]# systemctl disable firewalld.service        #禁止firewall开机启动
[root@localhost ~]# systemctl restart iptables.service                  #最后重启防火墙使配置生效
[root@localhost ~]# systemctl enable iptables.service                  #设置防火墙开机启动
[root@localhost ~]# iptables -L                 #查看防火墙规则,默认的是－t filter，如果是nat表查看，即iptables －t nat -L

#关闭SELINUX

[root@localhost ~]# vim /etc/selinux/config
#SELINUX=enforcing           #注释掉
#SELINUXTYPE=targeted           #注释掉
SELINUX=disabled              #增加
[root@localhost ~]# setenforce 0       #使配置立即生效
```

 