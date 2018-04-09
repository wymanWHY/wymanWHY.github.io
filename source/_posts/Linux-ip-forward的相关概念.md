---
title: Linux ip_forward的相关知识点
date: 2015-12-09 14:54:09
Categories: Linux OPS
tags:
   - ip_forward
---

### 简介 ###

IP地址分为**公有ip地址**和**私有ip地址**，**公有ip地址**是由INIC(internet network information center)负责的，这些IP地址分配给了注册并向INIC提出申请的组织机构。**私有ip地址**属于非注册地址，专门为组织内部使用。**私有ip地址**是不可能直接用来跟WAN通信的，要么利用帧来通信（FRE帧中继，HDLC,PPP）,要么需要路由的转发(nat)功能把私有地址转换为公有地址才行。

### 数据转发原理 ###

- 首先内网主机向外网主机发送数据包，由于内网主机与外网主机不在同一网段，所以数据包暂时发往内网默认网关GIP处理，而本网段的主机对此数据包不做任何回应。由于内网主机的SIP是私有的，禁止在公网使用，所以必须将数据包的SIP修改成公网上的可用IP，这就是网关收到数据包之后首先要做的事情--IP地址转换
- 然后网关再把数据包发往外网主机。外网主机收到数据包之后，只认为这是网关发送的请求，并不知道内网主机的存在，更不知道源IP地址是SIP而不是FIP，也没必要知道，目的主机处理完请求，把回应信息发还给网关的FIP。网关收到后，将目的主机返回的数据包的目标IP即FIP修改为发出请求的内网主机的IP地址即SIP，并根据路由表将其发给内网主机。这就是网关的第二个工作--数据包的路由转发。内网主机只要查看数据包的DIP与发送请求的SIP相同，就会回应，这就完成了一次请求。 


**出于安全考虑，Linux系统默认是禁止数据包转发的。所谓转发即当主机拥有多于一块的网卡时，其中一块收到数据包，根据数据包的目的ip地址将包发往本机另一网卡，该网卡根据路由表继续发送数据包。这通常就是路由器所要实现的功能。**

#### 配置Linux下的IP转发功能 ####

- **CentOS6**:

方法1：
```
[root@centos6 ~]# echo 1 > /proc/sys/net/ipv4/ip_forward
```
> 注意：这种方式重启网络服务或主机后失效

> 若要其自动执行，可将命令echo "1" > /proc/sys/net/ipv4/ip_forward 写入脚本/etc/rc.d/rc.local 或者 在/etc/sysconfig/network脚本中添加 FORWARD_IPV4="YES"

方法2：
```
vim /etc/sysctl.conf
...
net.ipv4.ip_forward = 0 #添加一行
```

- **CentOS7**:
```
[root@CentOS7 ~]# vim /etc/sysctl.d/99-sysctl.conf 
 ...
 结尾添加：
 net.ipv4.ip_forward = 1
:wq
[root@CentOS7 ~]# sysctl -p
net.ipv4.ip_forward = 1   //查看修改结果.
[root@CentOS7 ~]#
```