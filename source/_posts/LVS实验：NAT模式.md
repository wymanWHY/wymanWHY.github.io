---
title: LVS实验：NAT模式
date: 2017-07-05 18:17:10
categories: Linux OPS
tags:
	- LVS
	- ipvsadm
	- docker
---



实验利用用两台docker容器完成web请求响应，使用nginx来验证LVS功能

实验的LVS集群按功能模块分为：** Load Balancer + RealServer **两个模块

其中：

- Load Balancer由宿主机实现，位于集群系统的前端，通过ipvsadm实现对后端服务器实现负载均衡功能，对外 IP 地址也成为 VIP（虚拟 IP 地址）

- Real Server分别由两台安装了nginx的容器代替，完成web请求响应，服务器分别命名为RS1、RS2


## 实验步骤 ##

### 一、安装 ipvsadm 工具 ###

实验环境在ubuntu下进行，使用apt-get完成ipvsadm的安装

```bash
sudo apt-get install ipvsadm -y

# 使用 ipvsadm
sudo ipvsadm -L
```

### 二、创建Real Server池 ###

#### 1、使用docker创建两台容器 ####

```bash
docker run --name=RS1 -tdi ubuntu
docker run --name=RS2 -tdi ubuntu
```

#### 2、登录两台容器，记录各自的IP ####

```bash
docker exec RS1 -it /bin/bash
...
ifconfig 
```

- RS1：IP 地址为 192.168.0.2
- RS2：IP 地址为 192.168.0.3

#### 3、安装工具 ####

```bash
apt-get install vim nginx -y
```

#### 4、为区别两台容器所响应，分别修改各自的响应页面 ####

```bash
vi /usr/share/nginx/html/index.html
```

![](http://p7wcdketk.bkt.clouddn.com/18-4-30/59868212.jpg)

**PS: 改好后记得重启nginx服务 **

在宿主机上使用浏览器验证两台容器的web服务是否正常工作：

![](http://p7wcdketk.bkt.clouddn.com/18-4-30/59868212.jpg)

![](http://p7wcdketk.bkt.clouddn.com/18-4-30/5519228.jpg)


### 三、修改调度规则 ###

#### 1、Load Balancer为宿主机，修改宿主机上的内核路由转发： ####
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

#### 2、使用 ipvsadm 添加 ipvs 规则，并定义集群服务 ####
```bash
sudo ipvsadm -A -t LB本机IP:80 -s rr         #定义集群服务
sudo ipvsadm -a -t LB本机IP:80 -r 192.168.0.2 -m #添加 RealServer1
sudo ipvsadm -a -t LB本机IP:80 -r 192.168.0.3 -m #添加 RealServer2
sudo ipvsadm -l                 #查看 ipvs 定义的规则
```

> # 添加集群服务参数详解：
-A：添加一个新的集群服务
-t: 使用 TCP 协议
-s: 指定负载均衡调度算法
rr：轮询算法(LVS 实现了 8 中调度算法)
LB本机IP:80 定义集群服务的 IP 地址（VIP） 和应用端口


> -a：添加一个新的 RealServer 规则
-t：tcp 协议
-r：指定 RealServer IP 地址
-m：定义为 NAT 
上面命令添加了两个服务器 RS1 和 RS2

### 四、配置完成，测试验证 ###

使用浏览器打开本机IP，按住F5作死地刷新，看到页面不一致则说明LVS起作用了


![](http://p7wcdketk.bkt.clouddn.com/18-4-30/20472635.jpg)


![](http://p7wcdketk.bkt.clouddn.com/18-4-30/41208663.jpg)


