---
title: 通过xmanager远程连接ubuntu
date: 2018-05-08 14:30:13
categories: Linux OPS
tags:
    - XDMCP
    - xfce
---


通过远程登录的方式使用服务器，本打算用vnc，但技术不到家路子没完全走通，改换思路为使用xdmcp，桌面程序用的是轻量化的xubuntu-desktop桌面套件，也就是linus大拿也在用的xfce。底层系统为ubuntu server 18.04，整体环境如下：


|环境|描述|
|:-|:-|
|系统|Ubuntu Server 18.04|
|桌面环境|Xfce|
|远程协议|XDMCP|
|其他|lightDM|


## 安装必要套件 ##

### 1、安装gdm3 ###

```bash
apt-get install gdm3 -y
```

### 2、安装xubuntu-desktop ###

```bash
apt-get install xubuntu-desktop -y
```

## 修改配置文件 ##

### 1、修改LightDM配置 ###

LightDM的配置文件包含：

- /usr/share/lightdm/lightdm.conf.d/*.conf
- /etc/lightdm/lightdm.conf.d/*.conf
- /etc/lightdm/lightdm.conf

新建/etc/lightdm/lightdm.conf文件，编辑以下内容：

```bash
[SeatDefaults]
greeter-show-manual-login=true
xserver-allow-tcp=true

[XDMCPServer]
Enabled=true
Port=177
```

### 2、修改gdm配置 ###

修改文件/usr/share/gdm/gdm.schemas中的`xdmcp/Enable`一段：

```bash
<schema>
<key>xdmcp/Enable</key>
<signature>b</signature>
<default>true</default>
</schema>
```

### 3、修改LightDM的默认会话 ###

系统使用的配置参数保存在 /usr/share/lightdm/lightdm.conf.d/*.conf，如下图：
![](http://p7wcdketk.bkt.clouddn.com/18-5-8/32550270.jpg)

系统使用的默认会话保存在 `/usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf`中:
```bash
[SeatDefaults]
user-session=xxx
```

其中的 xxx 代表 `/usr/share/xsessions/xxx.desktop` 中 

![](http://p7wcdketk.bkt.clouddn.com/18-5-8/34328928.jpg)

我们可以就默认会话文件进行更改：

```bash
vi /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf

[Seat:*]
user-session=ubuntu

#added by myself
[XDMCPServer]
enabled=true
[SeatDefaults]
xserver-allow-tcp=true
```


**到此，配置结束**

重启后检验效果，防火墙要记得增加177端口的访问策略

```
ufw allow 177
```

---
参考文档：

- [xmanager远程Ubuntu1604LTS](https://blog.csdn.net/junbujianwpl/article/details/52792878)

- [(转)Ubuntu LightDM轻量级桌面显示管理器](https://blog.csdn.net/miracle_eicont/article/details/52268863)

- [Ubuntu中LightDM是什么（转）](https://www.cnblogs.com/EasonJim/p/7128317.html)