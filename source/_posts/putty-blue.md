---
title: 解决putty终端蓝色过深
date: 2016-10-19 18:36:14
tags:
- putty
- 问题集
---

解决putty 登录到linux上发现ls命令显示的目录蓝色太深看不清问题


方案一（暂时的）：
<!-- more -->
在"window"中选择"Colours",右边选择"ANSI
Blue"颜色修改为**84 84 254**

方案二（长期）：

使用**regedit**命令打开注册表

找到**HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions**中的保存的主机名

修改**Colour14**的值为**84,84,254**
