# Spring的AbstractRoutingDataSource解决多数据源的问题

多数据源问题很常见，例如读写分离数据库配置。

原来的项目出现了新需求，局方要求新增某服务器用以提供某代码，涉及到多数据源的问题。

研究成果如下：

##1首先配置多个datasource

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">  
        <property name="driverClassName" value="net.sourceforge.jtds.jdbc.Driver">  
        </property>  
        <property name="url" value="jdbc:jtds:sqlserver://10.82.81.51:1433;databaseName=standards">  
        </property>  
        <property name="username" value="youguess"></property>  
        <property name="password" value="youguess"></property>  
    </bean>  
    <bean id="dataSource2" class="org.apache.commons.dbcp.BasicDataSource">  
        <property name="driverClassName" value="net.sourceforge.jtds.jdbc.Driver">  
        </property>  
        <property name="url" value="jdbc:jtds:sqlserver://10.82.81.52:1433;databaseName=standards">  
        </property>  
        <property name="username" value="youguess"></property>  
        <property name="password" value="youguess"></property>  
</bean>  
```

## 2 继承AbstractRoutingDataSource

写一个DynamicDataSource类继承AbstractRoutingDataSource，并实现determineCurrentLookupKey方法

```java
package com.standard.core.util;  
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;  
public class DynamicDataSource extends AbstractRoutingDataSource {  
    @Override  
    protected Object determineCurrentLookupKey() {  
        return CustomerContextHolder.getCustomerType();  
    }  
}  

```

## 3 利用ThreadLocal解决线程安全问题

```java

package com.standard.core.util;  
public class CustomerContextHolder {  
    public static final String DATA_SOURCE_A = "dataSource";  
    public static final String DATA_SOURCE_B = "dataSource2";  
    private static final ThreadLocal<String> contextHolder = new ThreadLocal<String>();  
    public static void setCustomerType(String customerType) {  
        contextHolder.set(customerType);  
    }  
    public static String getCustomerType() {  
        return contextHolder.get();  
    }  
    public static void clearCustomerType() {  
        contextHolder.remove();  
    }  
```

## 4 在DAOImpl中切换数据源

```java
CustomerContextHolder.setCustomerType(CustomerContextHolder.DATA_SOURCE_B);   
```

