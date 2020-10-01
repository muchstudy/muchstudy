---
title: nginx+tomcat+memcached配置session共享
date: 2017-03-17 16:05:43
categories:
- 其它
---



## 背景

&emsp;&emsp;两台tomcat，通过nginx来实现负载均衡，使用memcached来解决session共享问题。

## tomcat配置

### 分别拷贝jar包到tomcatA、tomcatB的lib目录下

包含如下这些jar包，从{% asset_link lib.rar 这里下载 %}



```
asm-3.2.jar
kryo-1.04.jar
kryo-serializers-0.11.jar
memcached-session-manager-1.7.0.jar
memcached-session-manager-tc7-1.8.1.jar
minlog-1.2.jar
msm-kryo-serializer-1.7.0.jar
reflectasm-1.01.jar
spymemcached-2.7.3.jar
```

### 配置context.xml

```xml
<Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"
	   memcachedNodes="n1:127.0.0.1:11211"
	lockingMode="auto"
	sticky="false"
	requestUriIgnorePattern= ".*\.(png|gif|jpg|css|js)$"
	sessionBackupAsync= "false"
	sessionBackupTimeout= "100"
	copyCollectionsForSerialization="true"
	transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory"
		/>
```

## nginx配置


```
user  nobody;  
worker_processes  4;  
error_log  logs/error.log;  
events {  
    worker_connections  1024;  
}  

http {  
    include       mime.types;  
    default_type  application/octet-stream;  
    sendfile        on;  
    keepalive_timeout  65;  
    gzip  on;  
    upstream  myserver   {  
              server   127.0.0.1:8080;  
              server   127.0.0.1:8081;  
    }  
    server {  
        listen       80;  
        server_name  127.0.0.2;  
        charset utf-8;  
        location / {  
            root   html;  
            index  index.html index.htm;  
            proxy_pass        http://myserver;  
            proxy_set_header  X-Real-IP  $remote_addr;  
            client_max_body_size  100m;  
        }  


        location ~ ^/(WEB-INF)/ {
            deny all;
        }


        error_page   500 502 503 504  /50x.html;  
        location = /50x.html {  
            root   html;  
        }  


    }  
}  
```

说明：
1. tomcatA的地址为127.0.0.1:8080
2. tomcatB的地址为127.0.0.1:8081
3. 通过在浏览器上输入127.0.0.2，请求会动态分配到A和B上

## 验证Session共享是否配置成功

1.在tomcatA的webapp下新增test.jsp页面

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
	pageEncoding="ISO-8859-1"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
<title>tomcatA</title>
</head>
<body>
	SessionID:<%=session.getId()%>
	<%
		out.println("tomcatA");
	%>
</body>
</html>
```

2.在tomcatB的webapp下新增test.jsp页面

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
  pageEncoding="ISO-8859-1"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
<title>tomcatB</title>
</head>
<body>
  SessionID:<%=session.getId()%>
  <%
    out.println("tomcatB");
  %>
</body>
</html>
```
&emsp;&emsp;浏览器请求127.0.0.2/projectName/test.jsp，多刷新几下，如果seseionID一直不变，而不断切换A和B，则证明配置成功。
