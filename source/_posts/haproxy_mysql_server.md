---
title: 架构学习之HAProxy + MySQL主从结构 实现读取负载均衡
date: 2017-03-10 09:54:41
tags:
- MySQL
- HAProxy
- 架构学习
categories: 系统架构
---


## 架构简介

### 架构设计
本系统采用MySQL一主多从模式设计，即1台 MySQL“主”服务器(Master)+多台“从”服务器(Slave)，“从”服务器之间通过Haproxy进行负载均衡，对外只提供一个访问IP，当程序需要访问多台"从"服务器时，只需要访问Haproxy，再由Haproxy将请求分发到各个数据库节点。

我们的程序可以有俩个数据源(DataSourceA,DataSourceB)，一个(DataSourceA)直接连接主库，另外一个(DataSourceB)连接Haproxy，当需要写入操作时可以使用DataSourceA，读取时使用DataSourceB。

设计图如下：

![HAProxy_Mysql](/uploads/haproxy_mysql_0.png)

### 架构问题

主从数据库之间数据同步延时的问题

因为大多数使用MySQL主从同步数据都是异步的，也就是说当主库的数据发生变化时并不能立即的更新从库，这么做的目的也是为了更好的性能，那么设想一下，当用户新增一条记录后立刻去从库查询，可能并不能查到刚刚新增的数据。

然而实际情况并不应该是这样的，我们也不应该这样去设计程序，我们就拿一个类似于CSND博客管理的系统来说，假设一个“博客系统”只有俩部分， 一部分是博客管理后台，用户可以在后台新增，编辑，删除博客。另一部分是门户网站，负责展示所有用户的博客信息，相对于这样一个系统来说， 后台管理模块对数据库的操作压力不是很大，相反门户网站读取博客信息对数据库的压力很大，这也式一般互联网产品的特点，而且最重要的一点是系统可以接受主从同步数据带来的延迟，也就是说当用户在后台新增一条博客时，前台门户网站并不能立即查询到这条信息。一般都是再过一段时间后才会出现在首页，因为大多数系统都有缓存设置，这样正好给主从同步延迟带来时间。

后台用户在新增一条记录后，一般都是立即查询返回博客列表，按照上面说的岂不是查询不到？

这个问题可以这么解决：

1、后台用户在进行 新增，查询，编辑，删除等操作时直接连接主库，这样无论什么操作都是实时的，因为后台操作对数据库的压力不是很大，所以读写全部连接主库应该没什么问题！

2、门户网站查询博客列表时从 “从库集群中查询“，通过负载均衡技术，解决了扩展性，高可用性等问题，同时门户网站首页也不需要实时查询主库中的数据，因为网站本身一般都有缓存，也不是实时的。


## HAProxy 配置

MySQL master IP: 192.168.0.33:3306
MySQL slave1 IP: 192.168.0.34:3306
MySQL slave2 IP: 192.168.0.35:3306
HAProxy : 192.168.0.205

修改HAProxy配置文件/etc/haproxy/ haproxy.cfg

    global
        maxconn 4096
        daemon
        chroot      /var/lib/haproxy
        pidfile     /var/run/haproxy.pid
        #debug
        #quiet
        user haproxy
        group haproxy

    defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        log 127.0.0.1 local0
        retries 3
        option redispatch
        maxconn 2000
        #contimeout      5000
        #clitimeout      50000
        #srvtimeout      50000
        timeout http-request    10s
        timeout queue           1m
        timeout connect         10s
        timeout client          1m
        timeout server          1m
        timeout http-keep-alive 10s
        timeout check           10s

    listen  admin_stats 0.0.0.0:8888 #这个配置是监控页面，绑定到本机8888端口，账号admin，密码admin
        mode        http
        stats uri   /dbs  #http://IP:8888/dbs 即可登录监控后台。
        stats realm     Global\ statistics
        stats auth  admin:admin

    listen  proxy-mysql 0.0.0.0:23306  #代理的端口。我们程序连接从库集群时就访问这个端口。
        mode tcp
        balance roundrobin  #负载均衡方式
        option tcplog
        option mysql-check user haproxy #这里是配置健康检查的,需要在mysql中创建无任何权限用户haproxy，且无密码
        server MySQL1 192.168.0.34:3306 check weight 1 maxconn 2000
        server MySQL2 192.168.0.35:3306 check weight 1 maxconn 2000
        option tcpka

在MySQL中创建用户，用户名为haproxy 且无密码（重要） 否则haproxy无法检测MySQL状态

    CREATE USER 'haproxy'@'%' IDENTIFIED BY '';

配置完成后启动代理 service haproxy start