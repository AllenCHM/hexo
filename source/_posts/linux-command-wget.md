---
title: linux命令使用技巧
date: 2016-10-23 10:58:32
tags:
- linux命令
- wget
- 技巧集
---

**代理模式**

有时候需要下载国外插件，不得不使用代理模式

	wget -e "http_proxy=192.168.0.209:8080" www.google.com

<!-- more -->
或者使用环境变量

	http_proxy = "http://username:password@192.168.0.209:8080"
	export http_proxy


curl 使用代理

	curl -x 192.168.0.209:8080 http://www.baidu.com


检查nginx配置文件是否正确

	nginx -t -c nginx.conf
