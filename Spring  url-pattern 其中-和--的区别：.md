#Spring  url-pattern 其中/和/*的区别：
< url-pattern > / </ url-pattern >   不会匹配到*.jsp，即：*.jsp不会进入spring的 DispatcherServlet类 。 
< url-pattern > /* </ url-pattern > 会匹配*.jsp，会出现返回jsp视图时再次进入spring的DispatcherServlet 类， 导致找不到对应的controller所以报404错。 

总之，关于web.xml的url映射的小知识: 
< url-pattern>/</url-pattern>  会匹配到/login这样的路径型url，不会匹配到模式为*.jsp这样的后缀型url 
< url-pattern>/*</url-pattern> 会匹配所有url：路径型的和后缀型的url(包括/login,*.jsp,*.js和*.html等)


在web.xml配置servlet-mapping的时候，如果url-pattern设置为“/” (如下)，很多人都会遇到导入js,css,图片等静态资源出现Firefox调试窗口会报出的404错误，而你的确也不能访问那些资源
```xml
<servlet-mapping>
<servlet-name>dispatcherServlet</servlet-name>
<url-pattern>/</url-pattern>
</servlet-mapping>
```
百度了很久，大致有3种方法：但是我仍然没有解决404问题，后来又google了一下，发现少了一个关键的地方（在百度搜索中都没提到的一个地方，所以认为比较关键 :)），就是在jsp页面中导入静态资源的时候需要用<c:url>标签。

例如：
```
<%@ taglib prefix="c" uri="http://Java.sun.com/jsp/jstl/core" %>
<script type="text/JavaScript" src='<c:url value="/js/jQuery.js"></c:url>'></script>
```
这里的c:url中 value的值也是需要特别注意到地方，见下面3种方法详细说明：


方法1. 修改web.xml文件，增加对静态资源的url映射

如：
```xml
<servlet-mapping>
<servlet-name>default</servlet-name>
<url-pattern>*.js</url-pattern>
</servlet-mapping>
<servlet-mapping>
<servlet-name>default</servlet-name>
<url-pattern>*.css</url-pattern>
</servlet-mapping>
```
在web.xml中添加好配置后，在jsp页面就可以引用这些静态资源了，但需要用<c:url value="">,

如：
```
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<script type="text/javascript" src='<c:url value="/js/jquery.js"></c:url>'></script>
```
这里还需要说明的是：这种方法不能访问WEB-INF目录下的静态资源，也就是js目录必须是web根(可能是webapp,webContent等)目录下，否则是不能引用的；
如果放在WEB-INF目录下，即使你使用<c:url value=“/WEB-INF/js/jquery.js”>也是会出现404错误的。

百度时发现：以下各容器的default servlet名字，而且还提到静态资源servlet映射需要写在dispatcherServlet的前面；我在Jboss-eap-5.1中测试过，前后没有关系；所以可能是容器或者版本的关系吧。

Tomcat, Jetty, JBoss, and GlassFish  默认 Servlet的名字 -- "default"
Google App Engine 默认 Servlet的名字 -- "_ah_default"
Resin 默认 Servlet的名字 -- "resin-file"
WebLogic 默认 Servlet的名字  -- "FileServlet"
WebSphere  默认 Servlet的名字 -- "SimpleFileServlet" 


方法2.在相应的 -servlet.xml中添加spring配置<mvc:default-serlvet-handler>

如：
```
<mvc:default-servlet-handler/>
```
这种方法只需要添加一行代码，在jsp页面中引用时和方法1一样，同样也不能引用WEB-INF下的资源。


方法3.使用spring 3.0.4的新特性,在相应的 -servlet.xml中添加配置<mvc:resource>

如：
```
<mvc:resources location="/js/" mapping="/js/**" />
```
或
```
 <mvc:resources location="/WEB-INF/js/" mapping="/js/**" />
```
这种方法我写了两个配置，不同的地方只是location的值，一个是“/js/”,一个是“/WEB-INF/js/”；两种都可以，根据你自己的目录结构来引用。这就说明使用这种方式可以引用WEB-INF目录下的静态资源；这里的mapping属性的值用了ant的通配符方式，"/js/**"(两个"*")指location的值所表示的目录以及所有子目录；但是在jsp页面中引用时需要注意：

<c:url value="/js/jquery.js"> value的值必须类似于mapping属性的值，如果是3.1方式的配置，则引用的是web根目录下js/jquery.js，如果是3.2方式的配置，则引用的是web根目录下WEB-INF/js/jquery.js;
<c:url value="/js/ui/jquery-ui.js"> 则引用location目录下子目录ui下的jquery-ui.js。


