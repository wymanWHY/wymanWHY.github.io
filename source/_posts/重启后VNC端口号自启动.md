---
title: 重启后VNC端口号自启动
date: 2014-12-03 20:10:52
categories: Linux OPS
tags:
---

然后编辑/etc/sysconfig/vncservers，以下是文件内容：

> # The VNCSERVERS variable is a list of display:user pairs.

> #
> # Uncomment the line below to start a VNC server on display :1
> # as my 'myusername' (adjust this to your own).  You will also
> # need to set a VNC password; run 'man vncpasswd' to see how
> # to do that.
> #
> # DO NOT RUN THIS SERVICE if your local area network is
> # untrusted!  For a secure way of using VNC, see
> # <URL:http://www.uk.research.att.com/vnc/sshvnc.html>.
> VNCSERVERS="1:user1 2:user2 3:user3"
> VNCSERVERARGS[1]="-geometry 1024x768"
> VNCSERVERARGS[2]="-geometry 1024x768"
> VNCSERVERARGS[3]="-geometry 800x600"

解释一下这个文件：

VNCSERVERS这一行是配置在系统启动时启动几个VNC server，上面的例子里运行了三个VNC server，其中user1在display :1，user2在display :2，user3在display :3。

VNCSERVERARGS这三行，分别为VNC server 1, 2, 3配置启动参数，上面的例子里对user1和user2使用屏幕分辨率1024x768，对user3使用800x600。

其它支持的参数请使用“man vncserver”命令查询。

编辑好这个文件后，保存，然后以root身份运行：

/sbin/service vncserver start

这样user1, user2, user3的vncserver就启动了。

以后每次系统重启时，都会自动启动这三个用户的vncserver。

**注意：**上面三个用户必须已经使用vncpasswd命令设置过vnc密码，不然他的vncserver启动会失败
