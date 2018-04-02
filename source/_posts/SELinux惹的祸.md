---
title: SELinux惹的祸
date: 2018-04-02 18:30:14
categories: Linux OPS
tags:
   - SELinux
---


最近一周，在进行服务器配置改造的过程中，接二连三遇到一些诡异的问题：

- RSH无密码访问各项配置正确，**却死活不成功**
- ldap各项配置正确，**却死活不成功**
- nfs挂载各项配置正常，**却死活不成功**
- 还有那啥啥啥（忘了），但也是**死活不成功**


究其源头，都在于**没有关闭SELinux**,这货如同bug一般的存在

所以，如果你有什么配置看起来都很正常**却死活不成功**的时候，试试看是不是SELinux在作祟：

    getenforce


关闭：

    - setenforce 0

	- `vi /etc/selinux/config` 
	
	 **SELINUX=disabled**

关闭之后，一了百了