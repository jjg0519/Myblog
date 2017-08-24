# Tomcat 常见错误解决

catalina.sh

```
JAVA_OPTS="-server -XX:PermSize=256M -XX:MaxPermSize=512m"
```

 java -Xms512M -Xmx1024M -XX:PermSize=128M -XX:MaxPermSize=256M -jar /opt/service/module-sys/module-sys/module-sys.jar

 -Xms256M -Xmx512M -XX:PermSize=256m -XX:MaxPermSize=512m





## Caused by: java.lang.StackOverflowError

### 问题描述

```
Caused by: java.lang.IllegalStateException: Unable to complete the scan for annotations for web application []. Possible root causes include a too low setting for -Xss and illegal cyclic inheritance dependencies
	at org.apache.catalina.startup.ContextConfig.processAnnotationsStream(ContextConfig.java:2109)
	at org.apache.catalina.startup.ContextConfig.processAnnotationsJar(ContextConfig.java:1981)
	at org.apache.catalina.startup.ContextConfig.processAnnotationsUrl(ContextConfig.java:1947)
	at org.apache.catalina.startup.ContextConfig.processAnnotations(ContextConfig.java:1932)
	at org.apache.catalina.startup.ContextConfig.webConfig(ContextConfig.java:1326)
	at org.apache.catalina.startup.ContextConfig.configureStart(ContextConfig.java:878)
	at org.apache.catalina.startup.ContextConfig.lifecycleEvent(ContextConfig.java:369)
	at org.apache.catalina.util.LifecycleSupport.fireLifecycleEvent(LifecycleSupport.java:119)
	at org.apache.catalina.util.LifecycleBase.fireLifecycleEvent(LifecycleBase.java:90)
	at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5179)
	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
	... 11 more
Caused by: java.lang.StackOverflowError
	at org.apache.catalina.startup.ContextConfig.populateSCIsForCacheEntry(ContextConfig.java:2269)
	at org.apache.catalina.startup.ContextConfig.populateSCIsForCacheEntry(ContextConfig.java:2269)
	at 
```

###解决方法

```
i was able to workaround this exception by skipping the jar scannining in catalina.properties:

# Additional JARs (over and above the default JARs listed above) to skip when
# scanning for Servlet 3.0 pluggability features. These features include web
# fragments, annotations, SCIs and classes that match @HandlesTypes. The list
# must be a comma separated list of JAR file names.
org.apache.catalina.startup.ContextConfig.jarsToSkip=*.jar
```