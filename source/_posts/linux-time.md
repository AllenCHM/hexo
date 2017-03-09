---
title: ubuntu server 设置时间
date: 2016-10-27 14:40:25
tags:
- linux
- 技巧集
---

查看当前电脑时区：

	data -R
<!-- more -->

设置方法：

1、运行tzselect

选择Asia，再选China，最后选择Beijing

2、复制文件到/etc目录下

	cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime

3、更新时间

	ntpdate time.windows.com
