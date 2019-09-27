---
title: Linux下搭建Maven私服Nexus3.X
date: 2016-12-21 17:29:37
categories:
- Maven
---

&emsp;&emsp;操作系统为中标麒麟6，安装的Nexus版本为`nexus-3.2.0-01-unix.tar.gz`

## 一、准备

1. 从官网 https://www.sonatype.com/download-oss-sonatype 下载最新版本的资源
1. `Nexus 3.x.x`的JDK环境要求为1.8

## 二、安装配置

&emsp;&emsp;下载下来`nexus-3.2.0-01-unix.tar.gz`后，我使用`SecureFX`把资源上传到了`/opt`目录下。`SecureFX`是一个客户端工具，通过UI界面就能对Linux上的文件做操作。

{% asset_img 1.jpg %}

&emsp;&emsp;接下来使用`SecureCRT`连接上Linux服务器，在命令行输入如下命令
```
# 切换到opt目录
$ cd /opt

# 解压tar文件
$ tar xvzf nexus-3.2.0-01-unix.tar.gz

# 先切换到bin目录
$ cd nexus-3.2.0-01/bin/
# 启动服务器，同时还有stop，restart
$ ./nexus start
```
&emsp;&emsp;默认端口为8081，此时可通过`http://yourIp:8081`访问Nexus服务器。默认用户名与密码为`admin/admin123`.

&emsp;&emsp;如果要修改默认端口，可以到安装目录下找到`etc/nexus-default.properties`，修改`application-port`项，然后restart即可。

&emsp;&emsp;如果要卸载，使用`rm -rf nexus-3.2.0-01`删除掉安装目录即可。-r为子目录一起删，f为不用一一提示。

## 三、其它

&emsp;&emsp;在安装完修改端口为8083后，在浏览器中输入地址发现无法访问。

&emsp;&emsp;首先在Linux服务器上查看8083端口是否启用，通过`netstat -lnp`命令即可。

&emsp;&emsp;确认端口已正在使用服务器没问题后，在客户端的CMD中通过`telnet yourIp 8083`,发现客户端无法连接。

&emsp;&emsp;最后排查防火墙，在防火墙中设置允许8083端口的HTTP访问解决问题。
