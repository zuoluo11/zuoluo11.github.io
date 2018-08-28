---
title: 同一服务器安装多个redis实例
date: 2018-08-28 10:27:49
tags:
categories:
---

Redis用来做缓存的key-value数据库，当相同的系统发布到同一台服务器后，会导致
缓存中的内容相互影响，因此需要为每个系统单独设置一个redis的实例，各个实例通过
端口号来进行数据的相互隔离。
<!--more-->
## 将redis作为windows服务安装

[单实例安装参考地址](https://www.cnblogs.com/zoro-zero/p/6437519.html)

1、下载redis并解压到一个目录下，然后切换到该目录下，也就是redis-server.exe文件所在的目录

2、在cmd下执行 redis-server --service-install redis-windows-conf

3、服务安装成功后启动服务 redis-server --service-start

## 同一台服务器安装多个不同端口号的redis服务

1、将redis-windows-conf拷贝一份并重命名为redis_6380.conf。

2、将redis_6380.conf文件中的三处位置进行修改，包括：端口号，日志文件名，数据快照文件名
```
> *
# Accept connections on the specified port, default is 6379 (IANA #815344).
# If port 0 is specified Redis will not listen on a TCP socket.
port 6380

> *
# Specify the log file name. Also 'stdout' can be used to force
# Redis to log on the standard output.
logfile "redis-server-6380.log"

> *
# The filename where to dump the DB
dbfilename dump_6380.rdb
```

3、按照配置文件的端口号将redis新建为windows服务
redis-server --service-install 配置文件名 --loglevel verbose --service-name redis服务名称 --port 端口
redis-server --service-install redis_6380.conf --loglevel verbose --service-name redisService1 --port 6380

4、启动服务
redis-server --service-start –service-name redisService1

5、windows删除服务
sc delete redis服务名称 
sc delete redisService1