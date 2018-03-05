---
title: LVS实验：DR模式
date: 2018-03-05 18:32:57
categories: Linux OPS
tags:
	- LVS
	- ipvsadm
	- docker
---



> NAT模式的相关配置操作请参考[《LVS实验之NAT模式的实现》](http://wyman.wang/2018/03/05/LVS%E5%AE%9E%E9%AA%8C%EF%BC%9ANAT%E6%A8%A1%E5%BC%8F/)


和NAT模式不同的是，在DR实验中，我们新增一台容器作为Load Balancer，架构变更为：

- 宿主机环境：充当客户端访问 web 服务；
- LoadBalancer 的 container：装有 ipvsadm，充当负载均衡调度器；
- RS1 的 container：部署 Nginx web 服务器，提供 Web 访问服务，充当服务器池中的一员；
- RS2 的 container：部署 Nginx web 服务器，提供 Web 访问服务，充当服务器池中的一员；

## 实验步骤 ##

### 一、创建服务器池，并安装必备工具 ###

和NAT模式实验一样，所以别来无恙：

- 安装nginx，vim
- 修改响应页面
- 重启nginx服务

```bash
docker run --name=RS1 -tdi ubuntu
docker run --name=RS2 -tdi ubuntu
docker run --name=LB -tid ubuntu
...
apt-get install vim nginx -y
...
service nginx start
```

### 二、修改web服务器组的内核参数 ###

```bash
# 设置只回答目标IP地址是来访网络接口本地地址的ARP查询请求
echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore

# 为了保险自己可以查看一下是否成功修改
cat /proc/sys/net/ipv4/conf/lo/arp_ignore

# 设置对查询目标使用最适当的本地地址.在此模式下将忽略这个IP数据包的源地址并尝试选择与能与该地址通信的本地地址.
#首要是选择所有的网络接口的子网中外出访问子网中包含该目标IP地址的本地地址. 
#如果没有合适的地址被发现,将选择当前的发送网络接口或其他的有可能接受到该ARP回应的网络接口来进行发送.
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce
# 使得上面的配置立即生效
sysctl -p
```


> ARP 的内核参数详解：
> arp_ignore 部分参数：定义了本机响应 ARP 请求的级别
> 0表示目标 IP 是本机的，则响应 ARP 请求。默认为 0；
> 1如果接收 ARP 请求的网卡 IP 和目标 IP 相同，则响应 ARP 请求；


> arp_announce 参数：定义了发送 ARP 请求时，源 IP 应该填什么。
> 0 表示使用任一网络接口上配置的本地 IP 地址，通常就是待发送的 IP 数据包的源 IP 地址 。默认为 0
> 1 尽量避免使用不属于该网络接口(即发送数据包的网络接口)子网的本地地址作为 ARP 请求的源 IP 地址。大致的意思是如果主机包含多个子网，而 IP 数据包的源 IP 地址属于其中一个子网，虽然该 IP 地址不属于本网口的子网，但是也可以作为ARP 请求数据包的发送方 IP。
> 2 表示忽略 IP 数据包的源 IP 地址，总是选择网络接口所配置的最合适的 IP 地址作为 ARP 请求数据包的源 IP 地址(一般适用于一个网口配置了多个 IP 地址)

### 三、配置web服务器网卡别名 ###

只有目的 IP 是本机器中的一员时才会做相映的处理，所以需要添加网卡别名
```bash
ifconfig lo:0 192.168.0.10 broadcast 192.168.0.10 netmask 255.255.255.255 up
```

** 两台nginx服务器需做同样的设置 **

### 四、设置LoadBalancer调度规则 ###

在Load Balancer容器中，设置相应的ipvsadm规则：
```bash
ipvsadm -A -t 192.168.0.10:80 -s rr         # 定义集群服务
ipvsadm -a -t 192.168.0.10:80 -r 192.168.0.3 -g # 添加 RS1
ipvsadm -a -t 192.168.0.10:80 -r 192.168.0.4 -g # 添加 RS2
ipvsadm -l 
```

> 参数：
-A：添加一个新的集群服务
-t: 使用 TCP 协议
-s: 指定负载均衡调度算法
rr：轮询算法(LVS 实现了 8 中调度算法)
**192.168.0.10:80 定义集群服务的 IP 地址（VIP） 和端口**

> 
-a：添加一个新的 RealServer 规则
-t：tcp 协议
-r：指定 RealServer IP 地址
-g：定义为 DR 模式
上面命令添加了两个集群服务器 RealServer1 和 RealServer2

### 五、设置Load Balancer网卡别名 ###
`
ifconfig eth0:0 192.168.0.10 netmask 255.255.255.0 up`


### 六、完成配置，测试验证 ###
和NAT模式一样，按住F5不断刷新，看页面是否变化，验证LVS是否正常工作


{% asset_img 4.png %}

{% asset_img 5.png %}
