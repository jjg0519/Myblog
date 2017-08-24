# Java 安装和配置

## 安装

```bash
[root@shj-236 soft]# tar -zxvf jdk-8u101-linux-x64.tar.gz -C /usr/local/
[root@shj-236 soft]# cd /usr/local/
[root@shj-236 local]# ll
total 44
drwxr-xr-x. 2 root root 4096 Sep 23  2011 bin
drwxr-xr-x. 2 root root 4096 Sep 23  2011 etc
drwxr-xr-x. 2 root root 4096 Sep 23  2011 games
drwxr-xr-x. 2 root root 4096 Sep 23  2011 include
drwxr-xr-x. 8 uucp  143 4096 Jun 22  2016 jdk1.8.0_101
drwxr-xr-x. 2 root root 4096 Sep 23  2011 lib
drwxr-xr-x. 2 root root 4096 Sep 23  2011 lib64
drwxr-xr-x. 2 root root 4096 Sep 23  2011 libexec
drwxr-xr-x. 2 root root 4096 Sep 23  2011 sbin
drwxr-xr-x. 5 root root 4096 May 26 16:50 share
drwxr-xr-x. 2 root root 4096 Sep 23  2011 src
[root@shj-236 local]# 
```

## 环境变量

```bash
[root@shj-236 local]# vim /etc/profile
##打开之后在末尾添加
export JAVA_HOME=/usr/local/jdk1.8.0_101
export CLASSPATH=.:%JAVA_HOME%/lib/dt.jar:%JAVA_HOME%/lib/tools.jar  
export PATH=$PATH:$JAVA_HOME/bin  


[root@shj-236 local]#source /etc/profile
```

