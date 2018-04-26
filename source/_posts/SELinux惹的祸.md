---
title: 都是SELinux惹的祸
date: 2018-04-02 18:30:14
categories: Linux OPS
tags:
   - SELinux
---


最近半个月，在进行服务器配置改造的过程中，接二连三遇到一些诡异的问题：

- RSH无密码访问各项配置正确，**却死活不成功**
- ldap各项配置正确，**却死活不成功**
- nfs挂载各项配置正常，**却死活不成功**
- 还有那啥啥啥（对，什么都不顺），依旧**死活不成功**


要不是突然想起**SELinux**，还真以为是水逆惹的祸，如果你都不知道这货，[来来来，SELinux了解一下](http://wyman.wang/2017/08/11/%E5%85%B3%E4%BA%8ESELinux%EF%BC%88%E4%B8%80%EF%BC%89/)

作为一个以增强访问安全策略为目标的模块，这货如同bug一般的存在，经常干扰应用程序的正常运行，这一点和它的编写者——**美国国家安全局（NSA）**——非常类似，一个喜欢打着维护国家安全的旗号四处扰乱秩序的古惑仔。

所以，如果你有什么配置看起来都很正常**却死活不成功**的时候，试试看是不是SELinux在作祟：
查看：

```
    getenforce

     #enforcing  #开启状态    
     #permissive #宽容模式
     #Disabled   #关闭状态

```

关闭：

     setenforce 0       #临时关闭，进入permissive，重启失效

	 vi /etc/selinux/config  #永久关闭，重启生效
	...
	 SELINUX=disabled

关闭之后，一了百了

**当然，关闭之前你得确认你的环境不需要SELinux的介入也能得到必要的安全保障。**