#Java获取路径方法&相对路径读取xml文件方法
```java
request.getRealPath("/");//不推荐使用获取工程的根路径 
request.getRealPath(request.getRequestURI());//获取jsp的路径，这个方法比较好用，可以直接在servlet和jsp中使用 
request.getSession().getServletContext().getRealPath("/");//获取工程的根路径，这个方法比较好用，可以直接在servlet和jsp中使用 
this.getClass().getClassLoader().getResource("").getPath();//获取工程classes 下的路径，这个方法可以在任意jsp，servlet，java文件中使用，因为不管是jsp，servlet其实都是java程序，都是一个 class。所以它应该是一个通用的方法。
```
0、关于绝对路径和相对路径

1.基本概念的理解绝对路径：绝对路径就是你的主页上的文件或目录在硬盘上真正的路径，(URL和物理路径)例 如：C:xyz est.txt 代表了test.txt文件的绝对路径。http://www.sun.com/index.htm也代表了一个URL绝对路径。相对路径：相对与某个基 准目录的路径。包含Web的相对路径（HTML中的相对目录），例如：在Servlet中，"/"代表Web应用的跟目录。和物理路径的相对表示。例 如："./" 代表当前目录,"../"代表上级目录。这种类似的表示，也是属于相对路径。另外关于URI，URL,URN等内容，请参考RFC相关文档标准。RFC 2396: Uniform Resource Identifiers (URI): Generic Syntax,(http://www.ietf.org/rfc/rfc2396.txt)2.关于JSP/Servlet中的相对路径和绝对路径。 2.1服务器端的地址服务器端的相对地址指的是相对于你的web应用的地址，这个地址是在服务器端解析的（不同于html和javascript中的相对 地址，他们是由客户端浏览器解析的）

1、request.getRealPath

方法：request.getRealPath("/") 
得到的路径：C:\Program Files\Apache Software Foundation\Tomcat 5.5\webapps\strutsTest\

方法：request.getRealPath(".") 
得到的路径：C:\Program Files\Apache Software Foundation\Tomcat 5.5\webapps\strutsTest\.

方法：request.getRealPath("") 
得到的路径：C:\Program Files\Apache Software Foundation\Tomcat 5.5\webapps\strutsTest

request.getRealPath("web.xml") 
C:\Program Files\Apache Software Foundation\Tomcat 5.5\webapps\strutsTest\web.xml

2、request.getParameter(""); 
    ActionForm.getMyFile();

方法：String filepath = request.getParameter("myFile"); 
得到的路径：D:\VSS安装目录\users.txt

方法：String filepath = ActionForm.getMyFile(); 
得到的路径：D:\VSS安装目录\users.txt

-------------------------------------------------- 
strutsTest 为工程名

myFile 在ActionForm中，为private String myFile; 
在jsp页面中：为<html:file property="myFile"></html:file>

--------------------------------------------------

3、获得系统路径

在Application中: 
System.getProperty("user.dir")

在Servlet中: 
ServletContext servletContext = config.getServletContext(); 
String rootPath = servletContext.getRealPath("/");

在jsp中: 
application.getRealPath("")

4、其他1

1.可以在servlet的init方法里

String path = getServletContext().getRealPath("/");

这将获取web项目的全路径

例如 :E:\eclipseM9\workspace\tree\

tree是我web项目的根目录

2.你也可以随时在任意的class里调用

this.getClass().getClassLoader().getResource("").getPath();

这将获取 到classes目录的全路径

例如 : /D:/workspace/strutsTest/WebRoot/WEB-INF/classes/

还有 this.getClass().getResource("").getPath().toString();

这将获取 到 /D:/workspace/strutsTest/WebRoot/WEB-INF/classes/bl/

这个方法也可以不在web环境里确定路径,比较好用

3.request.getContextPath();

获得web根的上下文环境

如 /tree

tree是我的web项目的root context

5、其他2

java获取路径几种途径- -
      1. jdk如何判断程序中的路径呢？一般在编程中，文件路径分为相对路径和绝对路径，绝对路径是比较好处理的，但是不灵活，因此我们在编程中对文件进行操作的时候，一般都是读取文件的相对路径，

相对路径可能会复杂一点，但是也是比较简单的，相对的路径，主要是相对于谁，可以是类加载器的路径，或者是当前 java文件下的路径，在jsp编程中可能是相对于站点的路径，相对于站点的路径，我们可以通过 getServletContext().getRealPath("\") 和request.getRealPath("\"):这个是取得站点的绝对路径； 而getContextPath():取得站点的虚拟路径；

      2. class.getClassLoader.getPath():取得类加载器的路径：什么是类加载器呢？ 一般类加载器有系统的和用户自己定义的；系统的ClassLoader就是jdk提供的，他的路径就是jdk下的路径，或者在 jsp编程，比如Tomcat ，取得的类加载器的位置就是tomaca自己设计的加载器的路径，

明白了这些之后，对于文件路径的操作就会相当的清楚，我们在编程的时候，只要想清楚我们所操作的文件是相对于什么路径下的，取得相对路径就可以了.

6、总结

1、获取web服务器下的文件路径 
request.getRealPath("/") 
application.getRealPath("")【jsp中 】 
ServletContext().getRealPath("")

System.getProperty("user.dir")【不同位置调用，获取的路径是动态变化的】

2、获取本地路径

jsp中，<html:file property="myFile"/>

request.getParameter("myFile"); 
ActionForm.getMyFile(); 
获取的值相同：如D:\VSS安装目录\users.txt

*********************************

this.getClass().getClassLoader().getResource("").getPath(); 
＝＝/D:/workspace/strutsTest/WebRoot/WEB-INF/classes/ 
this.getClass().getResource("").getPath().toString(); 
＝＝/D:/workspace/strutsTest/WebRoot/WEB-INF/classes/bl/

3、获取相对路径

request.getContextPath();

如：/strutsTest

 
 
java使用相对路径读取xml文件
博客分类：
 
java
javaXMLJavaWeb 

一、xml文件一般的存放位置有三个： 
1.放在WEB-INF下； 
2.xml文件放在/WEB-INF/classes目录下或classpath的jar包中； 
3.放在与解析它的java类同一个包中，不一定是classpath； 

二、相对应的两种使用相对路径的读取方法： 

方法一：（未验证） 
将xml文件放在WEB-INF目录下，然后 
程序代码： 
InputStream is=getServletContext().getResourceAsStream( "/WEB-INF/xmlfile.xml" ); 

方法二：将xml文件放在/WEB-INF/classes目录下或classpath的jar包中，则可以使用ClassLoader的静态方法getSystemResourceAsStream(String s)读取； 
程序代码： 
String s_xmlpath="com\xml\hotspot.xml"; 
InputStream in=ClassLoader.getSystemResourceAsStream(s_xmlpath); 

方法三：xml在随意某个包路径下： 
String s_xmlpath="com\xml\hotspot.xml"; 
ClassLoader classLoader=HotspotXmlParser.class.getClassLoader(); 
InputStream in=classLoader.getResourceAsStream(s_xmlpath);