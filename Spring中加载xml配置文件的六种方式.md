#Spring中加载xml配置文件的六种方式

Spring中加载xml配置文件的方式,我总结的有6种, xml是最常见的spring 应用系统配置源。Spring中的几种容器都支持使用xml装配bean，包括： 
>XmlBeanFactory,ClassPathXmlApplicationContext,FileSystemXmlApplicationContext,XmlWebApplicationContext

##XmlBeanFactory 引用资源 
```
Resource resource = new ClassPathResource("appcontext.xml"); 
BeanFactory factory = new XmlBeanFactory(resource); 
```
##ClassPathXmlApplicationContext  编译路径 
```
ApplicationContext factory=new ClassPathXmlApplicationContext("classpath:appcontext.xml"); 
// src目录下的 
ApplicationContext factory=new ClassPathXmlApplicationContext("appcontext.xml"); 
ApplicationContext factory=new ClassPathXmlApplicationContext(new String[] {"bean1.xml","bean2.xml"}); 
// src/conf 目录下的 
ApplicationContext factory=new ClassPathXmlApplicationContext("conf/appcontext.xml"); 
ApplicationContext factory=new ClassPathXmlApplicationContext("file:G:/Test/src/appcontext.xml"); 
```
##用文件系统的路径 
```
ApplicationContext factory=new FileSystemXmlApplicationContext("src/appcontext.xml"); 
//使用了  classpath:  前缀,作为标志,  这样,FileSystemXmlApplicationContext 也能够读入classpath下的相对路径 
ApplicationContext factory=new FileSystemXmlApplicationContext("classpath:appcontext.xml"); 
ApplicationContext factory=new FileSystemXmlApplicationContext("file:G:/Test/src/appcontext.xml"); 
ApplicationContext factory=new FileSystemXmlApplicationContext("G:/Test/src/appcontext.xml"); 
```
## XmlWebApplicationContext是专为Web工程定制的
```
ServletContext servletContext = request.getSession().getServletContext(); 
ApplicationContext ctx = WebApplicationContextUtils.getWebApplicationContext(servletContext ); 
```
##使用BeanFactory 
```
BeanDefinitionRegistry reg = new DefaultListableBeanFactory(); 
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(reg); 
reader.loadBeanDefinitions(new ClassPathResource("bean1.xml")); 
reader.loadBeanDefinitions(new ClassPathResource("bean2.xml")); 
BeanFactory bf=(BeanFactory)reg; 
```
##Web 应用启动时加载多个配置文件 
通过ContextLoaderListener 也可加载多个配置文件，在web.xml文件中利用 
<context-pararn>元素来指定多个配置文件位置，其配置如下: 
Java代码  收藏代码
```xml
<context-param>  
    <!-- Context Configuration locations for Spring XML files -->  
       <param-name>contextConfigLocation</param-name>  
       <param-value>  
       ./WEB-INF/**/Appserver-resources.xml,  
       classpath:config/aer/aerContext.xml,  
       classpath:org/codehaus/xfire/spring/xfire.xml,  
       ./WEB-INF/**/*.spring.xml  
       </param-value>  
   </context-param>  
```
这个方法加载配置文件的前提是已经知道配置文件在哪里，虽然可以利用“*”通配符，但灵活度有限。 