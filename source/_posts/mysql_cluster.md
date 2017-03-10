---
title: 架构学习之MySQL主从结构
date: 2017-03-10 09:54:41
tags:
- MySQL
- 架构学习
categories: 系统架构
---


## Mysql 主从架构

### 复制概述

  Mysql内建的复制功能是构建大型，高性能应用程序的基础。将Mysql的数据分布到多个系统上去，这种分布的机制，是通过将Mysql的某一台主机的数据复制到其它主机（slaves）上，并重新执行一遍来实现的。复制过程中一个服务器充当主服务器，而一个或多个其它服务器充当从服务器。主服务器将更新写入二进制日志文件，并维护文件的一个索引以跟踪日志循环。这些日志可以记录发送到从服务器的更新。当一个从服务器连接主服务器时，它通知主服务器从服务器在日志中读取的最后一次成功更新的位置。从服务器接收从那时起发生的任何更新，然后封锁并等待主服务器通知新的更新。

请注意当你进行复制时，所有对复制中的表的更新必须在主服务器上进行。否则，你必须要小心，以避免用户对主服务器上的表进行的更新与对从服务器上的表所进行的更新之间的冲突。

### mysql支持的复制类型：

**基于语句的复制**：在主服务器上执行的SQL语句，在从服务器上执行同样的语句。MySQL默认采用基于语句的复制，效率比较高。一旦发现没法精确复制时，   会自动选着基于行的复制。

**基于行的复制**：把改变的内容复制过去，而不是把命令在从服务器上执行一遍. 从mysql5.0开始支持

**混合类型的复制**: 默认采用基于语句的复制，一旦发现基于语句的无法精确的复制时，就会采用基于行的复制。

### 复制解决的问题

MySQL复制技术有以下一些特点：

**数据分布 (Data distribution )**

**负载平衡(load balancing)**

**备份(Backups)**

**高可用性和容错行 High availability and failover**


### 复制如何工作
整体上来说，复制有3个步骤：

(1)master将改变记录到二进制日志(binary log)中（这些记录叫做二进制日志事件，binary log events）；

(2)slave将master的binary log events拷贝到它的中继日志(relay log)；

(3)slave重做中继日志中的事件，将改变反映它自己的数据。

下图描述了复制的过程：

![MySQL 主从复制过程](/uploads/mysql_cluster_0.gif)

该过程的第一部分就是master记录二进制日志。在每个事务更新数据完成之前，master在二日志记录这些改变。MySQL将事务串行的写入二进制日志，即使事务中的语句都是交叉执行的。在事件写入二进制日志完成后，master通知存储引擎提交事务。
下一步就是slave将master的binary log拷贝到它自己的中继日志。首先，slave开始一个工作线程——I/O线程。I/O线程在master上打开一个普通的连接，然后开始binlog dump process。Binlog dump process从master的二进制日志中读取事件，如果已经跟上master，它会睡眠并等待master产生新的事件。I/O线程将这些事件写入中继日志。
SQL slave thread（SQL从线程）处理该过程的最后一步。SQL线程从中继日志读取事件，并重放其中的事件而更新slave的数据，使其与master中的数据一致。只要该线程与I/O线程保持一致，中继日志通常会位于OS的缓存中，所以中继日志的开销很小。
此外，在master中也有一个工作线程：和其它MySQL的连接一样，slave在master中打开一个连接也会使得master开始一个线程。复制过程有一个很重要的限制——复制在slave上是串行化的，也就是说master上的并行更新操作不能在slave上并行操作。




## MySQL主从同步搭建

### 主库设置

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



### 从库设置
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