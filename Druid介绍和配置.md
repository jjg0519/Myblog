# Druid介绍和配置

## 介绍

Druid是一个用于[大数据](http://lib.csdn.net/base/hadoop)实时查询和分析的高容错、高性能开源分布式系统，旨在快速处理大规模的数据，并能够实现快速查询和分析。

Druid的主要特征：

-   为分析而设计——Druid是为OLAP工作流的探索性分析而构建，它支持各种过滤、聚合和查询等类；
-   快速的交互式查询——Druid的低延迟数据摄取架构允许事件在它们创建后毫秒内可被查询到；
-   高可用性——Druid的数据在系统更新时依然可用，规模的扩大和缩小都不会造成数据丢失；
-   可扩展——Druid已实现每天能够处理数十亿事件和TB级数据。

Druid可以做什么:

-   替换DBCP和C3P0。Druid提供了一个高效、功能强大、可扩展性好的数据库连接池。
-   可以监控数据库访问性能，Druid内置提供了一个功能强大的StatFilter插件，能够详细统计SQL的执行性能，这对于线上分析数据库访问性能有帮助。
-   数据库密码加密。直接把数据库密码写在配置文件中，这是不好的行为，容易导致安全问题。DruidDruiver和DruidDataSource都支持PasswordCallback。
-   SQL执行日志，Druid提供了不同的LogFilter，能够支持Common-Logging、Log4j和JdkLog，你可以按需要选择相应的LogFilter，监控你应用的数据库访问情况。
-   扩展JDBC，如果你要对JDBC层有编程的需求，可以通过Druid提供的Filter机制，很方便编写JDBC层的扩展插件。







## 配置

### Maven依赖

```xml
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid</artifactId>
  <version>1.0.20</version>
</dependency>
```



## 常用工具

使用durid的ConfigFilter对数据库密码加密



## 相关阅读

[主流连接池性能对比测试]: 主流连接池性能对比测试.md