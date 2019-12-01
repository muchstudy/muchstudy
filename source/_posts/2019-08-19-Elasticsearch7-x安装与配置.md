---
title: Elasticsearch7.x安装与配置
date: 2019-08-19 23:08:35
tags:
---

## 下载
1.下载地址：https://www.elastic.co/cn/start  
复制下载链接

```shell
# 下载资源
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.3.0-linux-x86_64.tar.gz
# 解压资源
tar -xf elasticsearch-7.3.0-linux-x86_64.tar.gz
```

## 运行
```shell
cd elasticsearch-7.3.0-linux-x86_64
# 显示启动elk
./bin/elasticsearch
# 后台启动
./bin/elasticsearch -d

```

## 其它配置

### 跨域配置
当使用elasticsearch-head插件访问elk时，需要设置允许跨域访问
```shell
vim config/elasticsearch.yml
# 新增如下内容
http.cors.enabled: true
http.cors.allow-origin: "*"
```

### 允许外部网络（非本地）访问9200端口
默认情况下，启动elk服务后，其它服务器是无法访问elk服务的（telnet不通），只允许本地访问。

需要做如下配置：
```shell
vim config/elasticsearch.yml
# 新增如下内容
network.host: 0.0.0.0
http.port: 9200
```

此时，启动会报如下两个错误：
```shell
ERROR: [2] bootstrap checks failed
[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[2]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
```

对于第一个配置，root用户权限下，增加如下配置：
```shell
vi /etc/sysctl.conf
# 新增如下内容
vm.max_map_count=262144
sysctl -p
```

对于第二个错误，增加如下配置：
```shell
vim config/elasticsearch.yml
# 新增如下内容
cluster.initial_master_nodes: [“node-1”]
```

作者公众号：  
<img src='http://muchstudy.com/2019/11/10/%E4%B8%80%E6%96%87%E6%90%9E%E5%AE%9AJS%E5%BC%82%E5%B8%B8%E6%8D%95%E8%8E%B7/YIYING.jpg'>
