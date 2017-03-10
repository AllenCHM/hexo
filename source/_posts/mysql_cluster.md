---
title: 架构学习之MySQL主从结构
date: 2017-03-10 09:54:41
tags:
- MySQL
- 架构学习
categories: 系统架构
---


## Mysql 主从架构

### MySQL主从同步搭建

#### 主库设置

IP：192.168.0.33

编辑/etc/my.cnf 文件打开log-bin


    # vim /etc/my.cnf

        log-bin=/application/mysql/data/mysql.bin
        server-id=1

使用命令查看：log_bin应为on状态

    [root@mysql-master-w ~]# mysql -uroot -p123456 -e "show variables like 'log_bin';"

| Variable_name | Value |
|---------------|-------|
| log_bin | ON |

    [root@mysql-master-w ~]# mysql -uroot -p123456 -e "show variables like 'server_id';"


| Variable_name | Value |
|---------------|-------|
| server_id | 1 |


授权可同步用户，登录mysql操作：

    [root@mysql-master-w ~]# grant replication slave on *.* to 'rep'@'192.168.0.%' identified by '123456';

用户rep，在192.168.0.0/24的所有计算机，密码是123456

    select user,host from mysql.user;##查看用户,确保上述添加授权用户正确。

锁表：

    flushtables with read lock; ##登录mysql操作

导出数据：

    mysqldump -uroot -poldboy123 -B-A --events|gzip>/opt/new.sql.gz

到时候将new.sql.gz推到从库服务器。

解锁：

    unlock tables;##mysql中操作

记下文件名，和位置信息

    mysql> show master status;

| File | Position | Binlog_Do_DB | Binlog_Ignore_DB |
|------------------|----------|--------------|------------------|
| mysql-bin.000002 | 412 | | |



#### 从库

服务器ip：192.168.0.34

修改/etc/my.cnf中的server-id=2，切忌不能与主库中的id相同。

将主库备份数据new.sql.gz导入数据库：

    gzip -d new.sql.gz

    mysql -uroot -p123456 <new.sql

将以下内容在从库中执行：

    CHANGE MASTER TO
    MASTER_HOST='192.168.0.33',
    MASTER_PORT=3306,
    MASTER_USER='rep',
    MASTER_PASSWORD='123456',
    MASTER_LOG_FILE='mysql-bin.000002',####此处内容，同主库show master status;
    MASTER_LOG_POS=412;##主库show master status；的Position信息


ok，以上内容会写入，master.info文件中

在数据库中执行：

    slave start； ##执行同步开关
    show slave status\G ##查看数据库是否同步

如果信息中有如下，两个yes，一个0，即表示 成功。

    Slave_IO_Running: Yes
    Slave_SQL_Running:Yes
    Seconds_Behind_Master:0

以上是主从同步的全部过程。

可在主库中创建或更新数据库或表，在从库中查看是否有变化，以达到测试到目的。