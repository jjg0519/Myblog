# Tomcat 日志介绍和处理

tomcat默认的日志管理是由JDK中的日志管理器（即[Java](http://lib.csdn.net/base/javase).util.Logger,也就是JULI）来实现的，这种管理方式远不如log4j管理方便

## 日志类型与级别

### Tomcat 日志分为下面５类

catalina 、 localhost 、 manager 、 admin 、 host-manager

### 每类日志的级别分为如下 7 种

SEVERE (highest value) > WARNING > INFO > CONFIG > FINE > FINER > FINEST (lowest value)

### 日志级别的设定方法

例如：1catalina.org.apache.juli.AsyncFileHandler.level = FINE

这里就可以修改等级FINE，如果想关闭日志就设置为**OFF**就可以。

### 配置日志路径

例如：1catalina.org.apache.juli.AsyncFileHandler.directory = ${catalina.base}/logs

就可以对日志记录的路径在设置

### 配置日志前缀

例如：1catalina.org.apache.juli.AsyncFileHandler.prefix = catalina.

也可以按照自己的爱好和需求做修改。

### 全部关闭记录

​    把logging.properties的配置清空，就能达到关闭全部日志的记录，当然也可以在每个等级做设置。