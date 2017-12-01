# Linux中Tomcat shutdown.sh后进程仍然存在解决办法

[TOC]

在catalina.sh脚本的代码前，加入以下语句：

```bash
[root@shj-124 xsimple]# vim /opt/web/tomcat.xsimple/bin/catalina.sh

#设置记录CATALINA_PID。#该设置会在启动时候bin下新建一个CATALINA_PID文件#关闭时候从CATALINA_PID文件找到pid，kill。。。同时删除CATALINA_PID文件
PRGDIR=`dirname "$PRG"`
if [ -z "$CATALINA_PID" ];then
 CATALINA_PID=$PRGDIR/CATALINA_PID
fi

```



```bash
[root@shj-124 xsimple]# vim /opt/web/tomcat.xsimple/bin/shutdown.sh 
#最后一行加上-force
exec "$PRGDIR"/"$EXECUTABLE" stop -force "$@"


```

