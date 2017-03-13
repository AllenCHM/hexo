---
title: 架构学习之Keepalived介绍
date: 2017-03-10 09:54:41
tags:
- Keepalived
- 架构学习
categories: 架构学习
---

## Keepalived 介绍

### 什么是Keepalived

&emsp;&emsp;    Keepalived是一个基于VRRP协议实现的LVS服务高可用方案,可以利用其来避免单点故障。
一个LVS服务会有2台服务器运行Keepalived,一台为主服务器(master),一台为备份服务器(backup),
但是对外表现为一个虚拟IP,主服务器会发送特定的消息给备份服务器,但备份服务器收不到这个消息时候,
即主服务器宕机的时候,备份服务器就会接管虚拟IP,继续提供服务,从而保证了高可用性。Keepalived是VRRP的
完美实现。

### VRRP协议介绍

&emsp;&emsp;    在现实的网络环境中,两台需要通信的主机大多数情况下并没有直接的物理连接。对于这样的情况,他们之间
路由怎样选择?主机如何选定到达目的主机的下一跳路由,这个问题通常的解决方法有两种:

* 在主机上使用动态路由协议(RIP、OSPF等)

* 在主机上配置静态路由

&emsp;&emsp;    很明显,在主机上配置动态路由是非常不切实际的,因为管理、维护成本以及是否支持等诸多原因。配置静态路由
就变得十分流行,但路由(或者说默认网关default gateway)却经常成为单点故障。VRRP的目的就是为了
解决静态路由单点故障问题,VRRP通过竞选(election)协议来动态的将路由任务交给LAN中虚拟路由器中的某台
VRRP路由器

&emsp;&emsp;    VRRP(虚拟路由冗余协议 Virtual Router Redundancy Protocol，简称VRRP)是由IETF提出的解决
局域网中配置静态网关出现单点失效现象的路由协议，1998年已推出正式的RFC2338协议标准。VRRP广
泛应用在边缘网络中，它的设计目标是支持特定情况下IP数据流量失败转移不会引起混乱，允许主机使用单路
由器，以及及时在实际第一跳路由器使用失败的情形下仍能够维护路由器间的连通性。

&emsp;&emsp;    类似的协议还有cisco的HSRP(热备份路由器协议 Hot Standby Router Protocol)

#### VRRP工作机制

&emsp;&emsp;     在一个VRRP虚拟路由器中,有多台物理的VRRP路由器,但是这多台的物理的机器并不能同时工作,而是由一
台成为Master的负责路由工作,其它的都是Backup,Master并非一成不变,VRRP让每个VRRP路由器参与
竞选,最终获胜的就是Master。Master拥有一些特权,比如:拥有虚拟路由器的IP地址,我们的主机就是用这个
IP地址作为静态路由的。拥有特权的Master要负责转发发送网关地址的包和响应ARP请求。

&emsp;&emsp;    VRRP通过竞争协议来实现虚拟路由器功能,所有的协议报文都是通过IP多播(multicast)包形式发送的。虚拟路由器由VRID(范围0-255)
和一组IP地址组成,对外表现为一个周知的MAC地址。所以,在一个虚拟路由器中,不管谁是Master,对外都是相同的MAC和IP(称为
VIP)。客户端主机并不需要因为Master的改变而修改自己的路由配置,对客户端来说,这种主从切换是透明的。

&emsp;&emsp;      在一个虚拟路由器中,只有作为Master的VRRP路由器会一直发送VRRP通告信息(VRRPAdvertisement message),Backup不会
抢占Master,除非它的优先级(priority)更高。但Master不可用时(Backup收不到通告信息), 多台Backup中优先级
最高的这台会被抢占为Master。这种抢占是非常快说的(<1s),以保证服务的连续性。由于完全醒考虑,VRRP包使用了加密协议进行加密

#### VRRP工作流程

##### 1、初始化

&emsp;&emsp;     路由器启动时,如果路由器的优先级是255(最高优先级,路由器拥有路由器地址),要发送VRRP通告信息,并发送广播ARP信息通告
路由器IP地址对应的MAC地址为路由虚拟MAC,设置通告信息定时器准时发送VRRP通告信息,转为Master状态;否则进入Backup
状态,设置定时器定时检查是否收到Master的通告信息。

##### 2、Master

* 设置定时通告定时器
* 用VRRP虚拟MAC地址响应路由器IP地址的ARP请求
* 转发目的MAC是VRRP虚拟MAC的数据包
* 如果是虚拟路由器IP的拥有者,将接受目的地址是虚拟路由器IP的数据包,否则丢弃
* 当收到shutdown的事件时删除通告定时器,发送优先级为0的通告包,转初始化状态
* 如果定时通告定时器超时时,发送VRRP通告信息
* 收到VRRP通告信息时,如果优先级为0,发送VRRP通告信息;否则判数据的优先级是否高于本机,或相等
而且实际IP地址大于本地实际IP,设置定时通告定时器,复位主机超时定时器,转Backup状态;否则的话,
丢弃该通告包

##### 3、Backup

* 设置主机超时定时器
* 不能响应针对虚拟路由器IP的ARP请求信息
* 丢弃所有目的MAC地址是虚拟路由器MAC地址的数据包
* 不接受目的是虚拟路由器IP的所有数据包
* 当收到shutdown的事件时删除主机超时定时器,转初始化状态
* 主机超时定时器超时的时候,发送VRRP通告信息,广播ARP地址信息,转Master状态
* 收到VRRP通告信息时,如果优先权为0,表示进入Master选举;否则判断数据的优先级是否高于本机,如果高的话承认Master有效,复位
主机超时定时器;否则的话,丢弃该通告包

#### ARP查询处理

&emsp;&emsp;    当内部主机通过ARP查询虚拟路由器IP地址对应的MAC地址时,Master路由器回复的MAC地址为虚拟
的VRRP的MAC地址,而不是实际网卡的MAC地址,这样路由器切换时让内网机器察觉不到;而在路由器重启时,
不能主动发送本机网卡的实际MAC地址。如果虚拟路由器开启的ARP代理(proxy_ARP)功能,代理的ARP回应
也回应VRRP虚拟MAC地址


## Keepalived 原理

Keepalived是模块化设计,不同模块负责不同的功能,下面是keepalived的组件

![Keepalived 组件](/uploads/keepalived_0.png)

在这个机构图中，处于内核的IPVS和NETLINK，其中NETLINK是提供高级路由及其他相关的网络功能，
如果在负载均衡器上启用iptables/netfilter，将会直接影响它的性能。对于图中不同模块功能的介绍如下：

* VRRP Stack 负责负载均衡器之间的失败切换FailOver。LVS中负责DR调度器组的

* Checkers 负责检查调度器后端的Real server 或者 Upstream Server的健康状况。
检查RS的状态，决定是否将其剔除

* WatchDog 负责监控checkers和VRRP进程的状况

* IPVS wrapper 用来发送设定的规则到内核IPVS

* Netlink Reflector 用来设定VRRP的vip地址。 VIP也有‘人’管


keepalived运行时，会启动3个进程，分别为：Core(核心进程)，Check和VRRP 
* Core：负责主进程的启动，维护和全局配置文件的加载；
* Check：负责健康检查
* VRRP：用来实现VRRP协议

总结：在VRRP协议的基础上实现了服务器主机的负载均衡，VRRP负责调度器之间的高可用。