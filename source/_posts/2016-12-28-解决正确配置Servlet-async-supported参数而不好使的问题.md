---
title: 解决正确配置Servlet async-supported参数报错问题
date: 2016-12-28 17:44:17
categories:
- Java
---

## 一、描述
环境：
>JDK 1.7  
>Servlet 3.0  
>tomcat 7

&emsp;&emsp;Servlet 3.0新增异步处理支持，配置servlet参数`<async-supported>true</async-supported> `，或通过注解方式启用`@WebServlet(urlPatterns = "/demo",asyncSupported = true) `。当正确配置后，发现照例报如下错误：
```
java.lang.IllegalStateException: A filter or servlet of the current chain does not support asynchronous operations.
	org.apache.catalina.connector.Request.startAsync(Request.java:1660)
	org.apache.catalina.connector.Request.startAsync(Request.java:1653)
	org.apache.catalina.connector.RequestFacade.startAsync(RequestFacade.java:1022)
```

## 二、解决办法
&emsp;&emsp;找了一圈，所有的解决办法都在说可能是参数配置未配置正确，或者是需要在server.xml上配置。最后，在<a href="http://stackoverflow.com/questions/10970829/async-servlet-exception">StackOverflow</a>上找到了如下答案

```java
request.setAttribute("org.apache.catalina.ASYNC_SUPPORTED", true);
```

## 三、其它

Servlet 3.0 新特性详解：https://www.ibm.com/developerworks/cn/java/j-lo-servlet30/
