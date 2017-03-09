---
title: mysql-python安装错误
date: 2016-10-19 11:19:20
tags:
- mysql-python
- 错误集
---

mysql-python 安装错误

在安装 mysql-python时，会出现：
<!-- more -->

    sh: mysql_config: not found
    Traceback (most recent call last):
    File "setup.py", line 15, in <module>
    metadata, options = get_config()
    File "/home/zhxia/apps/source/MySQL-python-1.2.3/setup_posix.py", line 43, in get_config
    libs = mysql_config("libs_r")
    File "/home/zhxia/apps/source/MySQL-python-1.2.3/setup_posix.py", line 24, in mysql_config
    raise EnvironmentError("%s not found" % (mysql_config.path,))
    EnvironmentError: mysql_config not found
    
原因是没有安装libmysqlclient-dev

    apt-get install libmysqlclient-dev


一般到此为止再重复安装即可如若不行，继续往下



找到mysql_config文件的路径

    sudo updatedb
    locate mysql_config
    
mysql_config的位置为：/usr/bin/mysql_config

在mysql-python源码包中找到：setup_posix.py 文件

将mysql_config.path值改为：/usr/bin/mysql_config

然后 python setup.py install 
