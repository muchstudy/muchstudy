---
title: Tomcat环境下设置HTTP强缓存
date: 2017-01-14 18:54:44
categories:
- HTTP
tags:
- 缓存
---

&emsp;&emsp;在之前的一篇文章<a href="http://muchstudy.com/2016/08/18/HTTP%E7%BC%93%E5%AD%98%E8%AF%A6%E8%A7%A3/">《HTTP缓存详解》</a>中详细的整理了关于HTTP缓存的知识点，这一篇通过实践，具体验证如何设置HTTP的强缓存，让客户端直接从本地缓存中拿资源，而不发起网络请求。

## 一、设置HTTP强缓存

&emsp;&emsp;可通过`Expires`与`Cache-Control`控制资源何时过期。`Expires`通过设置一个具体过期日期来控制，`Cache-Control`是设置一个距离第一次请求之后多久的时间段来控制。

### 1.自定义Filter

过滤器代码如下：
```java
public class FilterCache implements Filter {
	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		// TODO Auto-generated method stub
	}
	@Override
	public void doFilter(ServletRequest request, ServletResponse response,FilterChain chain) throws IOException, ServletException {
            HttpServletResponse res = (HttpServletResponse) response;  
            //res.setDateHeader("expries", new Date().getTime()+60*60*24*1000);	//- 设置一天失效，经测试Chrome下不生效
            res.setHeader("Cache-Control", "max-age=10");  					//- 这里的单位为秒，10代表第一次请求10s后过期
	}
	@Override
	public void destroy() {
		// TODO Auto-generated method stub
	}
}
```

web.xml配置如下：
```xml
<!-- 过滤所有的js文件 -->
<filter>
	<filter-name>FilterCache</filter-name>
	<filter-class>com.demo.cache.FilterCache</filter-class>
</filter>
<filter-mapping>
	<filter-name>FilterCache</filter-name>
	<url-pattern>*.js</url-pattern>
</filter-mapping>
```

### 2.使用容器的Filter

&emsp;&emsp;tomcat容器提供自有的过滤器来设置HTTP缓存，只需在web.xml中进行配置即可，相信其它服务器也有相关的配置项，例如Nginx、WebLogic等。

配置如下：

```xml
<filter>
 <filter-name>ExpiresFilter</filter-name>
 <filter-class>org.apache.catalina.filters.ExpiresFilter</filter-class>
 <init-param>
    <param-name>ExpiresByType image</param-name>
    <param-value>access plus 10 days</param-value>
 </init-param>
 <init-param>
    <param-name>ExpiresByType text/css</param-name>
    <param-value>access plus 10 days</param-value>
 </init-param>
 <init-param>
    <param-name>ExpiresByType application/javascript</param-name>
    <param-value>access plus 10 days</param-value>
 </init-param>
</filter>

<filter-mapping>
 <filter-name>ExpiresFilter</filter-name>
 <url-pattern>/*</url-pattern>
 <dispatcher>REQUEST</dispatcher>
</filter-mapping>

```

官方文档：
- tomcat7:http://tomcat.apache.org/tomcat-7.0-doc/config/filter.html#Expires_Filter
- tomcat8:http://tomcat.apache.org/tomcat-8.0-doc/config/filter.html#Expires_Filter

## 二、特别说明

### 1.客户端环境准备

在Chrome下，需进行如下设置（F12接着按F1即可打开settings面板）：

{% asset_img DisableCache.jpg  %}


### 2.刷新浏览器方式

&emsp;&emsp;经过测试，只有通过浏览器地址栏回车的方式强缓存才会生效；以F5、浏览器刷新按钮、Ctrl+R方式刷新页面均会发起HTTP网络请求，即使缓存未过期。

## 三、结果

在Chrome下测试结果如下：

{% asset_img chrome.jpg  %}

&emsp;&emsp;设置`Cache-Control`的过期时间为10s，第一次请求返回200，在size栏下显示文件大小；第二次刷新页面，size栏显示`from memory cache`,即证明浏览器未发起网路请求，直接从本地缓存中获取资源。在IE下会显示`来自缓存`，Firefox下显示`已缓存`。

&emsp;&emsp;经测试，使用`Cache-Control`设置缓存在Chrome、Firefox、Edge、IE11下均有效，使用`Expires`只在IE11与Edge下生效。
