---
title: Gitlab 8.x runner安装与配置
date: 2018-07-13 18:41:31
tags:
---

## 介绍

&emsp;&emsp;Gitlab 8.x之后默认集成了Gitlab CI，意味着支持了持续集成相关功能。每一次集成操作都需要对应的runner来跑代码构建、测试、发布等操作。Runner实际上就是为Gitlab的持续集成指定一个环境。

## 安装

> 官方文档地址：https://docs.gitlab.com/runner/install/

&emsp;&emsp;Gitlab Runner的版本需要跟Gitlab对应,<a href='https://docs.gitlab.com/runner/'>这里</a>有一个对照表。最新的版本对照表中并没有Gitlab8.X对应的Runner版本，查了一下Gitlab8.X对应的Runner版本为`1.X`,所以这里选择`runner 1.11.2`版本。

&emsp;&emsp;这里运行Gitlab与Runner的环境均为CentOS，之前尝试在windows上安装runner，对接Linux上的Gitlab，发现在Gitlab runner运行的控制台出现乱码问题。

0.准备

在opt下创建gitlab-runner目录并进入该目录，后续执行的操作与所有的资源都放在这个目录中

```bash
cd /opt
mkdir gitlab-runner
cd gitlab-runner/
```

1.下载

下载安装资源到gitlab-runner目录中

```bash
sudo wget https://gitlab-ci-multi-runner-downloads.s3.amazonaws.com/v1.11.2/binaries/gitlab-ci-multi-runner-linux-386
```

2.添加运行权限

```bash
sudo chmod +x gitlab-ci-multi-runner-linux-386
```

3.创建用户

```bash
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
```

4.安装

```bash
./gitlab-ci-multi-runner-linux-386 install --user=gitlab-runner --working-directory=/opt/gitlab-runner
sudo gitlab-ci-multi-runner-linux-386 start
```


## 配置

&emsp;&emsp;经过上面的步骤，Runner就已经跑起来了，剩下的还需要Runner与项目对接起来。Runner的类型分为<a href='https://docs.gitlab.com/ee/ci/runners/README.html'>Shared, specific and group Runners</a>。这里选择specific类型，即单独的项目使用。

&emsp;&emsp;在Gitlab项目的setting-runner中,配置过程中会使用到`url`和`token`如下所示：

<div style='text-align:center'>{% asset_img setting.png %}<div>

1.运行register命令

```bash
./gitlab-ci-multi-runner-linux-386 register
```
之后就按照提示就行了

2.输入url地址  
3.输入token  
4.输入描述，任意即可  
5.输入标签，这里直接Enter跳过  
6.选择Runner executor，这里选择shell  

到这里就已经注册成功了，输入`./gitlab-ci-multi-runner-linux-386 list`就能看到上面的注册的条目。

官方文档地址：https://docs.gitlab.com/runner/register/index.html

## 其它

&emsp;&emsp;上面两个步骤做完后，此时按理说Gitlab就能调用Runner跑持续集成了，实际当中还会碰到其它问题，整理如下。

### 权限问题

&emsp;&emsp;如果在Gitlab的Build控制台上报`无法创建文件夹`、`无法运行bash`等,证明创建的`GitLab Runner`权限不够。
此时，我这里是修改GitLab Runner的权限跟root保持一致。


```bash
vim /etc/passwd
```
通过上面命令可以编辑用户对应的权限，我这里打开默认为`gitlab-runner:x:601:601:GitLab Runner:/home/gitlab-runner:/bin/bash`,权限组修改为跟root的一致`gitlab-runner:x:0:0:GitLab Runner:/home/gitlab-runner:/bin/bash`。(root的权限组名为0)

> 这里在另外一台机器上还碰到这样修改了也不好使的问题，最终gitlab-runner install的时候，直接指定为root，而不新创建用户。

### 环境问题

由于Runner运行需要环境支撑，比如git、node、npm等，需要在Runner所在的服务器上准备好所有的依赖。

- Linux Node安装

```bash
# 下载
wget https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-x64.tar.xz
# 解压
tar -xf  node-v8.11.3-linux-x64.tar.xz
# 建立软链接，实现全局访问
ln -s /opt/gitlab-runner/node-v8.11.3-linux-x64/bin/node /usr/local/bin/node
ln -s /opt/gitlab-runner/node-v8.11.3-linux-x64/bin/npm /usr/local/bin/npm
```

此时，输入`node -v`就能看到node的版本了。

使用软连接方式可能对非root用户无效，可以转而使用配置环境变量的方式

```bash
# 修改配置文件
vim /etc/profile
#set for nodejs,新增NODE_HOME并放到PATH上
export JAVA_HOME=/opt/soft/java
export NODE_HOME=/opt/gitlab-runner/node-v8.11.3-linux-x64  
export CLASSPATH=$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export PATH=$PATH:$JAVA_HOME/bin:$NODE_HOME/bin

```

> 在vim环境下点击i进入插入状态,编辑完成后按Esc键,然后输入 :wq 按回车保存退出。


备注：内外环境还需修改NPM的镜像源，比如修改为`npm config set registry https://registry-npm.daojia-inc.com/`

## 附录 部分GitLab-Runner常用命令
1.gitlab-runner帮助：gitlab-runner –help

2.gitlab-runner指定命令帮助：gitlab-runner <commond> –help

3.注册runner：gitlab-runner register

4.注销runner：gitlab-runner unregister

5.当前运行的runner：gitlab-runner list

6.启动runner：gitlab-runner start

7.停止runner：gitlab-runner stop

8.重启runner：gitlab-runner restart

9.查询runner状态：gitlab-runner status
