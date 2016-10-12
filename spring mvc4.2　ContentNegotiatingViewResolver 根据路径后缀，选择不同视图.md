#spring mvc4.2　ContentNegotiatingViewResolver 根据路径后缀，选择不同视图

###spring　mvc配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context" xmlns:mvc="http://www.springframework.org/schema/mvc" xsi:schemaLocation="http://www.springframework.org/schema/beans  
                               http://www.springframework.org/schema/beans/spring-beans.xsd  
                               http://www.springframework.org/schema/context  
                               http://www.springframework.org/schema/context/spring-context.xsd  
                               http://www.springframework.org/schema/mvc  
                               http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <mvc:annotation-driven content-negotiation-manager="contentNegotiationManager" />
    <mvc:view-controller path="/" view-name="index" />
    <bean id="contentNegotiationManager" class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
        <property name="mediaTypes">
            <value>
                html=text/html json=application/json
            </value>
        </property>
        <property name="defaultContentType" value="text/html" />
    </bean>
    <bean class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
        <property name="order" value="0" />
        <property name="contentNegotiationManager" ref="contentNegotiationManager" />
        <property name="viewResolvers">
            <list>
                <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
                    <property name="prefix" value="/WEB-INF/jsp/" />
                    <property name="suffix" value=".jsp"></property>
                </bean>
            </list>
        </property>
        <property name="defaultViews">
            <list>
                <bean class="com.alibaba.fastjson.support.spring.FastJsonJsonView">
                    <property name="charset" value="UTF-8" />
                </bean>
            </list>
        </property>
    </bean>
    <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter" />
</beans>

```

##写个测试controller

```java
    package com.doctor.springframework.web.view;  
      
    import java.util.HashMap;  
    import java.util.Map;  
      
    import org.springframework.stereotype.Controller;  
    import org.springframework.web.bind.annotation.RequestMapping;  
    import org.springframework.web.servlet.ModelAndView;  
        
    @Controller  
    public class SimpleController {  
          
        @RequestMapping({"/test.html","/test.json"})  
        public ModelAndView test() {  
            Map<String, String> map = new HashMap<>();  
            map.put("test", "json");  
            map.put("test-html", "html");  
            map.put("what", "what");  
            ModelAndView modelAndView = new ModelAndView("/contentViewTest");  
            modelAndView.addObject(map);  
            return modelAndView;  
              
        }  
          
    }  
```

##测试代码：
```
    package com.doctor.springframework.web.view;  
      
    import org.apache.http.client.fluent.Request;  
    import org.apache.http.client.fluent.Response;  
    import org.junit.After;  
    import org.junit.Before;  
    import org.junit.Test;  
      
    import com.doctor.embeddedjetty.EmbeddedJettyServer3;  
    /** 
     * ContentNegotiatingViewResolverPractice 根据路径后缀，选择不同视图 
     * @author doctor 
     * 
     * @time   2015年1月7日 上午10:08:24 
     */  
    public class ContentNegotiatingViewResolverPractice2 {  
        private EmbeddedJettyServer3 embeddedJettyServer;  
        private int port;  
        @Before  
        public void init() throws Throwable {  
            port = 8989;  
            embeddedJettyServer = new EmbeddedJettyServer3(port,"/contentNegotiatingViewResolverPractice/webapp", SpringContextConfig.class, SpringMvcConfig2.class);  
            embeddedJettyServer.start();  
        }  
      
        @Test  
        public void test() throws Throwable {  
      
            Response response = Request.Get("http://localhost:8989/test.json").execute();  
            System.out.println(response.returnContent().asString());  
      
            response = Request.Get("http://localhost:8989/test.html").execute();  
            System.out.println(response.returnContent().asString());  
              
            response = Request.Get("http://localhost:8989").execute();  
            System.out.println(response.returnContent().asString());  
      
        }  
      
        @After  
        public void destroy() throws Throwable {  
            embeddedJettyServer.stop();  
        }  
    }  
```

##输出内容：

```
    {"hashMap":{"test":"json","test-html":"html","what":"what"}}  
      
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">  
    <html>  
    <head>  
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">  
    <title>Insert title here</title>  
    </head>  
    <body>  
        hello contentNegotiatingViewResolverTest  
    </body>  
    </html>  
    <html>  
    <body>  
    <h2>Hello World! doctor</h2>  
    </body>  
    </html>  
```

