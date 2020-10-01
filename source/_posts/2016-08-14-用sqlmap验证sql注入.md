---
title: 用sqlmap验证sql注入
date: 2016-08-14 23:00:03
categories:
- 安全
tags:
- sql注入
- sqlmap
---

### 一.安装python  
https://www.python.org/downloads/  
使用python 2.x即可

### 二.step by step安装完后添加环境变量path  
![描述](/images/20160814/1.png)

### 三.下载sqlmap  
http://sqlmap.org/  
解压到本地磁盘即可

### 四.使用sqlmap验证注入项  
例如：通过存在的注入点扫描出有哪些数据库，还可通过该注入点把整个数据库的数据下载到本地，基本上只要是拼字符串的sql都存在注入问题
![描述](/images/20160814/2.png)


### 五.sqlmap常用参数  
![描述](/images/20160814/3.png)

post注入  
 使用 --data参数
 ```
 sqlmap -u “192.168.216.147/checklogin.php” –data “myusername=admin&mypassword=admin&Submit=Login” –level=5 –risk=3 –dbs
 ```  
