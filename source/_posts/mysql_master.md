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

