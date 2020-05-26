---
title: Centos安装Mysql记录
date: 2020-05-24 17:42:49
tags:
---

## 安装
```
# 下载yum源
wget 'https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm'

# 安装yum源
sudo rpm -Uvh mysql57-community-release-el7-11.noarch.rpm

# 查看有哪些版本可安装
yum repolist all | grep mysql

# 切换安装版本
# 关闭5.7版本
sudo yum-config-manager --disable mysql57-community
# 打开5.6版本
sudo yum-config-manager --enable mysql56-community

# 安装
sudo yum install mysql-community-server
```

在centos6下会报如下错误：（centos7正常）
```
Error: Package: mysql-community-server-5.6.48-2.el7.x86_64 (mysql56-community)
           Requires: libc.so.6(GLIBC_2.17)(64bit)
Error: Package: mysql-community-server-5.6.48-2.el7.x86_64 (mysql56-community)
           Requires: systemd
Error: Package: mysql-community-libs-5.6.48-2.el7.x86_64 (mysql56-community)
           Requires: libc.so.6(GLIBC_2.17)(64bit)
Error: Package: mysql-community-server-5.6.48-2.el7.x86_64 (mysql56-community)
           Requires: libstdc++.so.6(GLIBCXX_3.4.15)(64bit)
Error: Package: mysql-community-client-5.6.48-2.el7.x86_64 (mysql56-community)
           Requires: libc.so.6(GLIBC_2.17)(64bit)
```

原因分析如下：
https://unix.stackexchange.com/questions/280385/can-not-install-mysql-server-on-centos-6-7-32bit-error-need-rpm

在网上找了一圈，按照下面一顿操作，发现能够正常安装成功
```
# cd /etc/yum.repos.d/  找到mysql-56-community,将enable置为0 enable=0
sudo vi mysql-community.repo

# 重新安装mysql   
sudo yum install mysql-server
```

在centos6上安装完后，发现由于网络安全原因，机器无法开3306端口；索性换了一台centos7的机器，直接一气呵成安装MySQL8.0


## 基本操作

```
# 查看mysql运行状态
sudo service mysqld status
# 查看端口情况
sudo lsof -i tcp:3306
# 启动mysql，需要加sudo，否则会报FAILED错误
sudo service mysqld start
# 结束服务
sudo service mysqld stop
```

### 修改密码
首次安装后查看默认密码
```
[work@40-31-60 soft]$ sudo grep 'temporary password' /var/log/mysqld.log
2020-05-21T09:56:30.576083Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: UlCbRjdoE4?a
```

默认情况下Mysql是不运行远程连接的，故需要新增远程连接账户
```
# 连接数据库（输入上面查询出来的密码）
mysql -u root -p
```

修改密码
```
# 8.0下报错
mysql> set password for root@localhost = password('root');
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'password('root')' at line 1

# 修改密码(数字、大小写、特殊字符)
alter user 'root'@'localhost' identified by '58daojiaDJ!!';
# 密码复杂度过低会报如下错误
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

# 查看密码强度规则
 mysql>  SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file    |        |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.01 sec)
```

### 添加账户
查看现有用户
```shell
# 选择数据库
mysql> use mysql;
# 用户查询
mysql> select host,user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
| localhost | root             |
+-----------+------------------+
4 rows in set (0.00 sec)
```
新增允许远程连接的账户
```shell
# 新增用户；[admin'@'%]中的%号代表允许任意远程客户端连接
mysql> CREATE USER 'admin'@'%' IDENTIFIED BY '58admin!!AAA';
# 添加权限
mysql> GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%';
```


## 其他

### 为work账户添加sudo权限
root 账户键入`visudo`即可进入sudo配置
找到`root    ALL=(ALL)   ALL`
在这一行下面增加`work ALL=(ALL)  NOPASSWD:ALL`即可


### Node连接异常处理

使用mysql包（ https://www.npmjs.com/package/mysql ）连接服务时报如下错误
```shell
Error: ER_NOT_SUPPORTED_AUTH_MODE: Client does not support authentication protocol requested by server; consider upgrading MySQL client
```

解决方案如下：
```shell
# 选择mysql数据库
use mysql;
# 修改admin的密码，关键在于mysql_native_password关键字指定密码类型
ALTER user 'admin'@'%' IDENTIFIED WITH mysql_native_password by '58admin!!AAA';
# 更新
FLUSH PRIVILEGES;

```

执行完，node端就可以连接上mysql8.0了。

可以到mysql数据库中的user表中查看密码，其它的都为`caching_sha2_password`类型，修改完的这个为`mysql_native_password`

> 资料：https://stackoverflow.com/questions/50093144/mysql-8-0-client-does-not-support-authentication-protocol-requested-by-server


<div style="width:70%;margin:auto">
<img src='http://muchstudy.com/2020/04/04/%E8%81%8A%E8%81%8A%E4%B8%80%E7%BA%BF%E5%BC%80%E5%8F%91%E7%9A%84%E5%9F%BA%E6%9C%AC%E7%B4%A0%E5%85%BB/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.gif'>
</div>
