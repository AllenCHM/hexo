---
title: ubuntu网络相关
date: 2016-10-26 09:48:48
tags:
- linux
- 网络
- 技巧集
---

在ubuntu中已测试

**静态路由配置**
编辑文件/etc/network/interfaces

<!-- more -->

	auto eth0
	iface eth0 inet static
	address 192.168.0.209
	netmask 255.255.255.0
	gateway 192.168.0.1


**临时DNS配置**
编辑文件/etc/resolv.conf

	nameserver 192.168.0.1


**永久DNS**
编辑文件 /etc/resolvconf/resolv.conf.d/base

	nameserver 192.168.0.1
