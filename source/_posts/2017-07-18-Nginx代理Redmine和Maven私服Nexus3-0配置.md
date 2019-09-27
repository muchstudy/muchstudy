---
title: Nginx代理Redmine和Maven私服Nexus3.0配置
date: 2017-07-18 16:13:36
categories:
- 其它
---

## 背景

&emsp;&emsp;在这之前部门申请了一个独立IP，这个独立IP对应一台PC机，这台机器上安装了产品问题受理平台Redmine以及Maven私服。由于PC机性能有限，新找了一台双网卡的服务器接到独立IP上，为了不重新迁移环境，让PC机跟服务器组成了内网环境。在服务器上搭建Nginx来把请求转发到内网的PC机上。

## Nginx配置

```
#user  nobody;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    #nexus支持
    proxy_send_timeout 120;
    proxy_read_timeout 300;
    proxy_buffering    off;
    keepalive_timeout  5 5;
    tcp_nodelay        on;

    #代理到后台服务器
    upstream ws_redmine {
        server 192.168.100.100:8082;
    }
    upstream ws_nexus {
        server 192.168.100.100:8083;
    }
    server {
        listen       8082;
        server_name  localhost;

        location / {
            proxy_pass   http://ws_redmine;
        }

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    server {
        listen       8083;
        server_name  localhost;

        # nexus支持
        # allow large uploads of files - refer to nginx documentation
        client_max_body_size 1G;

        location / {
            proxy_pass   http://ws_nexus;
            # nexus支持
            proxy_set_header Host $host:$server_port;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

}

```

说明：
- 同时开启两个端口，分别对应redmine和nexus服务器
- 配置nexus的转发时如果按照redmine的配置，会报400 bad request错误，之前以为是nginx的配置问题，最后翻nexus的官方文档，发现需要额外的配置。安装官方文档配置好后，发现转发时端口号缺失。查了一圈资料，原来是<a href="https://books.sonatype.com/nexus-book/reference3/install.html#reverse-proxy">官方文档</a>错误，需要修改`proxy_set_header Host $host`，变为`proxy_set_header Host $host:$server_port`.

## 其它记录

- 查看Linux端口占用情况：`lsof -i:80`,80为端口号
- 关闭占用端口的应用进程：`kill -9 11071`，11071为上一步中显示出来的端口占用进程号
- Linux中启动Nginx：在nginx安装目录下运行`nginx -c nginx.conf`，不知道安装在哪儿可以使用`whereis nginx`查看
- Linux中关闭Nginx:`pkill -9 nginx`
