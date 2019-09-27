---
title: Eclipse下忽略掉node_modules目录相关配置
date: 2017-06-05 22:59:41
categories: Java
---

## 1.背景

&emsp;&emsp;Eclipse项目中的静态资源采用webpack来打包，在项目中的webapp目录下会生成node_modules目录，里面包含node相关模块。由于资源文件较多，会造成Eclipse编译缓慢，另外这些文件不需要发不到运行服务器上，且不需要做版本控制。

## 2.相关配置

### 2.1 编译Eclipse项目时忽略掉node_modules目录

&emsp;&emsp;选中项目右键-properties,如下图所示

{% asset_img 1.png%}

### 2.2 SVN版本管理时忽略掉node_modules目录

&emsp;&emsp;当使用Eclipse中的SVN来来就行版本控制时，选择node_modules目录后，发现`添加至svn:ignore`为灰色，不可用，无法忽略相关文件。可以通过如下配置来解决该问题。

{% asset_img 2.png%}

### 2.3 项目打包发布时去掉node_modules目录

&emsp;&emsp;在项目的maven pom.xml中build下的配置如下

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-war-plugin</artifactId>
	<version>2.6</version>
	<configuration>
		<packagingExcludes>
			node_modules/**
		</packagingExcludes>
	</configuration>
</plugin>
```
