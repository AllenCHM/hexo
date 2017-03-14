---
title: 架构学习之MySQL双主架构
date: 2017-03-10 09:54:41
tags:
- MySQL
- 架构学习
categories: 架构学习
---


## Mysql 双主架构

### 架构思路

![Mysql 双主架构图](/uploads/mysql_master_0.png)

1.两台mysql都可读写，互为主备，默认只使用一台（masterA）负责数据的写入，
另一台（masterB）备用

2.masterA是masterB的主库，masterB又是masterA的主库，它们互为主从

3.两台主库之间做高可用,可以采用keepalived等方案（使用VIP对外提供服务）

4.所有提供服务的从服务器与masterB进行主从同步（双主多从）

5.建议采用高可用策略的时候，masterA或masterB均不因宕机恢复后而抢占VIP（非抢占模式）

这样做可以在一定程度上保证主库的高可用,在一台主库down掉之后,可以在极短的时间内切换到另一台主库上（尽可能减少主库宕机对业务造成的影响），减少了主从同步给线上主库带来的压力；

但是也有几个不足的地方:

1.masterB可能会一直处于空闲状态（可以用它当从库，负责部分查询）；

2.主库后面提供服务的从库要等masterB先同步完了数据后才能去masterB上去同步数据，
这样可能会造成一定程度的同步延时；

### 实验测试

MySQL 使用apt-get安装的5.7版本


#### 修改MySQL的配置文件

mysql master 1 的配置文件

```
server_id = 1
log_bin=mysql_bin    # 打开二进制日志功能，作为主库时必须设置
log_slave_updates    # 做为从库时，数据库的修改也会写到bin_log里
binlog_ignore_db = mysql
binlog_ignore_db = information_schema
binlog_ignore_db = performance_schema
replicate_wild_ignore_table = mysql.%
replicate_wild_ignore_table = information_schema.%
replicate_wild_ignore_table = performance_schema.%
expire_logs_days=5   # 表示自动删除5天以前的binlog，可选
```

mysql master 2 的配置文件

```
server_id = 2
log_bin=mysql_bin
log_slave_updates
binlog_ignore_db = mysql
binlog_ignore_db = information_schema
binlog_ignore_db = performance_schema
replicate_wild_ignore_table = mysql.%
replicate_wild_ignore_table = information_schema.%
replicate_wild_ignore_table = performance_schema.%
expire_logs_days=5
```

重启MySQL服务
```
service mysql restart
```

#### Master-Master配置

##### 配置mysql master 1为主库
```
> reset master;(清空master的binlog，平时慎用，可选)
> flush tables with read lock;
> show master status;
```

##### 配置mysql master mysql master 1的从库
```
> stop slave;
> CHANGE MASTER TO MASTER_HOST='192.168.0.33', MASTER_USER='root', MASTER_PASSWORD='1', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=120;
> start slave;
> show slave status\G
```
其中MASTER_LOG_FILE和MASTER_LOG_POS的参数为上一步中查看到的结果


#### 配置mysql master 2 为主库
```
> reset master;(清空master的binlog，平时慎用，可选)
> flush tables with read lock;
> show master status;
```

#### 配置mysql master 1 为mysql master 2的从库
```
> unlock tables;
> stop slave;
> CHANGE MASTER TO MASTER_HOST='192.168.0.36', MASTER_USER='root', MASTER_PASSWORD='1', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=120;
> start slave;
> show slave status\G
```

#### 解除mysql master 2 上的表锁
```
> unlock tables;
```
