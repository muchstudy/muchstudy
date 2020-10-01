---
title: Servlet中Cookie增、删、改、查
date: 2016-11-18 14:43:04
categories:
- Java
tags:
- 工具类
---

&emsp;&emsp;整理了一个在Servlet中对Cookie增删改查的工具类，首先要注意的是在服务器端是无法对Cookie做修改的，只能做到覆盖创建。

**引用StackOverflow上James Sumners的<a href="http://stackoverflow.com/questions/5093250/how-do-you-update-an-existing-cookie-in-jsp">回答</a>：**
>Per section 3.3.4 of <a href="http://www.ietf.org/rfc/rfc2965.txt">RFC 2965</a>, the user agent does not include the expiration information in the cookie header that is sent to the server. Therefore, there is no way to update an existing cookie's value while retaining the expiration date that was initially set based solely on the information associated with the cookie.

> So the answer to this question is: you can't do that.

**工具类如下：**

```java
package com.demo.util;

import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.net.URLEncoder;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class CookieUtils {

	/**
	 * 新增Cookie
	 * @param response
	 * @param name
	 * @param value
	 */
	public static void addCookie(HttpServletResponse response, String name,String value) {
		try {
			//特殊字符需要编码
			Cookie cookie = new Cookie(name,URLEncoder.encode(value, "UTF-8"));
			cookie.setMaxAge(60*60*24*7);	//- 单位为秒，7天有效
			cookie.setPath("/");			//- 根路径
			//JavaEE5兼容
			try{
				cookie.setHttpOnly(true);	//- 防XSS
			}catch (NoSuchMethodError e) {
	        	e.printStackTrace();
	        }
			response.addCookie(cookie);
		} catch (UnsupportedEncodingException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
    }

	/**
	 * 删除Cookie
	 * @param request
	 * @param response
	 * @param name
	 * @return
	 */
	public static boolean deleteCookie(HttpServletRequest request,HttpServletResponse response, String name) {
        if (request.getCookies() != null) {
            for (Cookie cookie : request.getCookies()) {
                if (cookie.getName().equals(name)) {
                	cookie.setMaxAge(0);
                	//经测试发现还需设置如下两个值,之所以这样，原理为覆盖掉Cookie，而不是常规意义中的删除
									cookie.setValue("");
									cookie.setPath("/");
                	response.addCookie(cookie);
									return true;
                }
            }
        }

        return false;
    }

	/**
	 * 覆盖掉之前的cookie
	 * @param request
	 * @param name
	 * @param value
	 */
	public static void overrideCookie(HttpServletRequest request,HttpServletResponse response, String name,String value) {
        if (request.getCookies() != null) {
            for (Cookie cookie : request.getCookies()) {
                if (cookie.getName().equals(name)) {
                	cookie.setValue(value);
									cookie.setPath("/");
									//cookie.getMaxAge()为-1，服务器端无法获取MaxAge
									cookie.setMaxAge(60*60*24*7);
									response.addCookie(cookie);
                }
            }
        }
    }

	/**
	 * 获取Cookie
	 * @param request
	 * @param name
	 * @return
	 */
	public static Cookie getCookie(HttpServletRequest request, String name) {
        if (request.getCookies() != null) {
            for (Cookie cookie : request.getCookies()) {
                if (cookie.getName().equals(name)) {
                    return cookie;
                }
            }
        }

        return null;
    }
	/**
	 * 获取Cookie对应的值
	 * @param request
	 * @param name
	 * @return
	 */
	public static String getCookieValue(HttpServletRequest request, String name) {
        if (request.getCookies() != null) {
            for (Cookie cookie : request.getCookies()) {
                if (cookie.getName().equals(name)) {
                    try {
						return URLDecoder.decode(cookie.getValue(), "UTF-8");
					} catch (UnsupportedEncodingException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
						return null;
					}
                }
            }
        }
        return null;
    }
}

```
