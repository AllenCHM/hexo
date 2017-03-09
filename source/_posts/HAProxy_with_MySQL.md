---
title: 架构学习之HAProxy
date: 20167-3-9 18:54:41
tags:
- HAProxy
- MySQL
- 读写分离
---


## HAProxy 简介

HAProxy是一个使用c语言编写的自由及开放源代码的软件，提供高可用性、负载均衡，以及基于TCP和HTTP的应用程序代理。

HAProxy支持会话保持及七层处理，所以特别适合负载大的web站点。可以支持数以万计的并发连接。并且它的运行模式使得它可以简单安全的整合进当前的框架中，同时可以保护web服务器不被暴露到网络上。

HAProxy实现了一种事件驱动，单一进程模型，此模型支持非常大的并发连接数。多进程或多行程模型内受内存限制、系统调度器限制以及无处不在的锁限制，很少能处理数千并发连接。事件驱动模型因为在有更好的资源和时间管理的用户空间（User－Space）实现所有这些任务，所以没有这些问题。此模型的弊端是，在多核系统上，这些程序通常扩展性较差。这就是为什么他们必须进行优化以使每个CPU时间片(Cycle)做更多的工作。

包括GitHub、Bitbucket、Stack Overflow、Reddit、Tumblr、Twitter及亚马逊网络服务都在使用HAProxy。

<br>

## 配置HAProxy Session 亲缘性的三种方式

HAProxy附在均衡保持客户端和服务器Session亲缘性的三种方式：

### 1、用户IP识别

HAProxy将用户IP经过hash计算后指定到固定的真实机器上（类似于Nginx的IP hash指令）

**配置指令** `` *balance source* ``


### 2、cookie识别

HAProxy将WEB服务端发送给客户端的cookie中插入（或添加前缀）HAProxy定义的后端的服务器COOKIE ID。

**配置指令例举** ``*cookie SESSION_COOKIE insert indirect nocache*``

用Firebug可以观察到用户的请求头的cookie里 有类似“Cookie jessionid＝xxxxxxx; SESSION_COOKIE=app1”其中SESSION_COOKIE就是HAProxy添加的内容。

### 3、session识别

HAProxy将后端服务器产生的session和后端服务器识别在HAProxy中的一张表里。客户端请求时先查询这张表。

**配置指令例举** ``*appsession JSESSIONID len 64 timeout 5h request-learn*``

### #vi /usr/local/haproxy/haproxy.cfg
	
	backend COOKIE_srv
	mode http
	cookie SESSION_COOKIE insert indirect nocache
	server REALsrv_70 184.82.239.70:80 cookie 11 check
	inter 1500 rise 3 fall 3 weight 1
	server REALsrv_120 220.162.237.120:80 cookie 12 
	check inter 1500 rise 3 fall 3 weight 1
	backend SOURCE_srv
	mode http
	balance source
	server REALsrv_70 184.82.239.70:80 cookie 11 check 
	inter 1500 rise 3 fall 3 weight 1
	server REALsrv_120 220.162.237.120:80 cookie 12 
	check inter 1500 rise 3 fall 3 weight 1
	backend APPSESSION_srv
	mode http
	appsession JSESSIONID len 64 timeout 5h request-learn
	server REALsrv_70 184.82.239.70:80 cookie 11 check 
	inter 1500 rise 3 fall 3 weight 1
	server REALsrv_120 220.162.237.120:80 cookie 12 
	check inter 1500 rise 3 fall 3 weight 1

