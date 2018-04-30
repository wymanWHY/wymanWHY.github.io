---
title: 脚本实现批量关闭闲置用户的VNC端口
date: 2015-02-02 11:24:47
categories: shell
tags: 
	- shell
	- sed
	- awk
	- VNC
	- Xvnc
---




> **VNC**是一个利用端口号允许用户远程登录服务器的客户端程序


> 开启VNC登录端口的用户会在服务器后台运行用户的主控程序Xvnc的进程

> 但开启后用户长期不登录进程会一直驻留在后台，因此，写了个脚本批量关闭闲置的Xvnc进程：
__________________________________________________________


### 一、查看现运行的Xvnc进程 ###


    ps ef |grep Xvnc

![](http://p7wcdketk.bkt.clouddn.com/18-4-30/26316933.jpg)

第一列及第九列是我们需要的信息，分别为用户名及对用的VNC端口号

使用**awk**提炼一下，我们要的信息变为：

![](http://p7wcdketk.bkt.clouddn.com/18-4-30/21306540.jpg)

### 二、查看目前在线用户 ###

```bash
    who
```


可以确定目前正在进行远程登录的用户有哪些

![](http://p7wcdketk.bkt.clouddn.com/18-4-30/78058811.jpg)

### 三、关闭不在线用户的Xvnc进程 ###

**关键命令**：

```bash
    vncserver -kill :$端口号
```

当然，执行该命令前必须切换至开启该端口号的用户下

### 脚本 ###

```bash
#!/bin/bash

## To kill the vnc process not using...

ps -ef|grep Xvnc|awk '{print $1 $9}' > userport_map

while read LINE
do
 USER=$(echo $LINE | awk -F ':' '{print $1}')  #取用户名赋予USER变量
 PORT=$(echo $LINE | awk -F ':' '{print $2}')	#取相应的端口号赋予PORT变量
 w|grep $USER
 case $? in #通过上条命令执行结果的返回值来判断该用户是否在线：0——在线；1——不在线
  1)
	su $USER -c "vncserver -kill :$PORT"  #不在线的话，切换至该用户并执行vncserver -kill 命令
	echo "killing $USER \'s Xvnc process"
 esac
	 
done < userport_map

rm userport_map
```