#PreparedStatement和Statement区别
[TOC]
##原理说明
　　PreparedStatement: 数据库会对sql语句进行预编译，下次执行相同的sql语句时，数据库端不会再进行预编译了，而直接用数据库的缓冲区，提高数据访问的效率（但尽量采用使用？号的方式传递参数），如果sql语句只执行一次，以后不再复用。    从安全性上来看，PreparedStatement是通过?来传递参数的，避免了拼sql而出现sql注入的问题，所以安全性较好。在开发中，推荐使用 PreparedStatement。

　　PreparedStatement的第一次执行消耗是很高的。它的性能体现在后面的重复执行。例如, 假设我使用Employee ID, 
使用prepared的方式来执行一个针对Employee表的查询。JDBC驱动会发送一个网络请求到数据解析和优化这个查询，而执行时会产生另一个网络请求。**在JDBC驱动中，减少网络通讯是最终的目的**。如果我的程序在运行期间只需要一次请求, 
那么就使用Statement. 对于Statement, 同一个查询只会产生一次网络到数据库的通讯。

　　对于使用PreparedStatement池的情况下, 本指导原则有点复杂。当使用PreparedStatement池时, 如果一个查询很特殊, 
并且不太会再次执行到, 那么可以使用Statement。如果一个查询很少会被执行,但连接池中的Statement池可能被再次执行, 
那么请使用PreparedStatement。在不是Statement池的同样情况下, 请使用Statement。

　　Update大量的数据时, 先Prepare一个INSERT语句再多次的执行, 会导致很多次的网络连接。要减少JDBC的调用次数改善性能, 
你可以使用PreparedStatement的AddBatch()方法一次性发送多个查询给数据库. 例如, 让我们来比较一下下面的例子。

　　例 1: 多次执行Prepared Statement
```
PreparedStatement ps = conn.prepareStatement("INSERT into employees values (?, ?, ?)");
for (n = 0; n < 100; n++) {
	ps.setString(name[n]);    
	ps.setLong(id[n]);    
	ps.setInt(salary[n]);    
	ps.executeUpdate();
} 
```
　　例 2: 使用Batch
```java
PreparedStatement ps = conn.prepareStatement("INSERT into employees values (?, ?, ?)");
for (n = 0; n < 100; n++) {
	ps.setString(name[n]);
	ps.setLong(id[n]);
	ps.setInt(salary[n]);
	ps.addBatch();
}
ps.executeBatch(); 
```
在例 1中, PreparedStatement被用来多次执行INSERT语句。
　　在这里, 执行了100次INSERT操作, 共有101次网络往返。其中,1次往返是预储statement, 另外100次往返执行每个迭代。在例2中, 
　　当在100次INSERT操作中使用addBatch()方法时, 只有两次网络往返。1次往返是预储statement, 另一次是执行batch命令。虽然Batch命令会用到更多的数据库的CPU周期, 但是通过减少网络往返，性能得到提高。
	
**记住, JDBC的性能最大的增进是减少JDBC驱动与数据库之间的网络通讯**

##JDBC连接总结
* 防范sql注入
* 预编译sql语句。提高效率
* 对于数据的参数提供了方便的设置方式
* 减少JDBC驱动与数据库之间的网络通讯
* 关闭自动提交功能，提高系统性能 　　
>　　在第一次建立与数据库的连接时，在缺省情况下，连接是在自动提交模式下的。为了获得更好的性能，可以通过调用带布尔值false参数的Connection类的setAutoCommit()方法关闭自动提交功能，如下所示:　
>```
    conn.setAutoCommit(false);
>```
>　　值得注意的是，一旦关闭了自动提交功能，我们就需要通过调用Connection类的commit()和rollback()方法来人工的方式对事务进行管理

* 在动态SQL或有时间限制的命令中使用Statement对象
>　　在执行SQL命令时，我们有二种选择：可以使用PreparedStatement对象，也可以使用Statement对象。无论多少次地使用同一个SQL命令，PreparedStatement都只对它解析和编译一次。
>　　当使用Statement对象时，每次执行一个SQL命令时，都会对它进行解析和编译。这可能会使你认为，使用PreparedStatement对象比使用Statement对象的速度更快。然而，我进行的测试表明，在客户端软件中，情况并非如此。因此，在有时间限制的SQL操作中，除非成批地处理SQL命令，我们应当考虑使用Statement对象。

* 当执行同一个PreparedStatement对象时，它就会被再解析一次，但不会被再次编译。在缓冲区中可以发现预编译的命令，并且可以重新使用。使用PreparedStatement对象带来的编译次数的减少能够提高数据库的总体性能。
* 在成批处理重复的插入或更新操作中使用PreparedStatement对象 ,如果成批地处理插入和更新操作，就能够显著地减少它们所需要的时间