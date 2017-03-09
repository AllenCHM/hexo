---
title: mysql远程登录问题
date: 2016-10-20 20:27:48
tags:
- mysql
- 远程登录
- 问题集
---

root远程登录被拒绝
解决方法1：
1、使用命令进入mysql终端

		mysql -u root -p

<!-- more -->
2、使用一下命令

		mysql> use mysql;
		mysql> select host,user form user; (查看用户权限情况)
		mysql> Grant all privileges on *.* to 'root'@'%' identified by ‘password’with grant option;
		mysql> flush privileges; (刷新权限)

