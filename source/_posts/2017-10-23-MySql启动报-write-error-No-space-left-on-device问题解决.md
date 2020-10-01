---
title: 'MySql启动报 write error: No space left on device问题解决'
date: 2017-10-23 11:20:54
categories: DBMS
---

启动MySQL数据库时报如下错误：
```
[root@localhost redmine-3.1.1-1]# ./ctlscript.sh start
171023 09:36:32 mysqld_safe Logging to '/opt/redmine-3.1.1-1/mysql/data/mysqld.log'.
171023 09:36:32 mysqld_safe Starting mysqld.bin daemon with databases from /opt/redmine-3.1.1-1/mysql/data
/opt/redmine-3.1.1-1/mysql/bin/mysqld_safe: line 128: echo: write error: No space left on device
/opt/redmine-3.1.1-1/mysql/bin/mysqld_safe: line 165: 16298 Killed  
```

其中，引人注目的就是`No space left on device`。

使用`df -h`命令查看磁盘使用情况如下：
```
[root@localhost ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/sda2              50G   50G     0 100% /
tmpfs                 3.8G  100K  3.8G   1% /dev/shm
/dev/sda1             485M   54M  407M  12% /boot
/dev/sda5             404G  227M  383G   1% /home
```
可以看到磁盘空间已经被完全占满。  

接着，使用`du -sh *`命令查看`/dev/sda2`中的磁盘占用具体情况，结果如下：
```
[root@localhost /]# du -sh *
8.8M	bin
37M	boot
344K	dev
34M	etc
29M	home
122M	lib
27M	lib64
4.0K	media
0	misc
4.0K	mnt
0	net
42G	opt
2.4G	root
19M	sbin
4.0K	srv
164K	tmp

[root@localhost /]# cd opt
[root@localhost opt]# du -sh *
15M	apache-tomcat-9.0.0.M8
15M	ksar
200M	nexus-3.2.0-01
100M	nexus-3.2.0-01-unix.tar.gz
904M	redmine-3.1.1-1
40G	sonatype-work
```

逐级往下找，发现是nexus的日志文件已经有30多G了，删掉，再次启动mysql问题得到解决。
