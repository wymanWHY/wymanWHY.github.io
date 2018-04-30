---
title: 配置RSH无密码访问中遇到的kerberos坑
date: 2015-03-28 17:57:20
categories: Linux OPS
tags: 
    - RSH
    - kerberos
---



RSH配了半个上午，一直没有成功，简直崩溃。反复了三回之后，我才觉得这事应该无关人品，难道是版本问题？

用which 查看了一下server端的rsh：



    which rsh
    

发现rsh居然存在于kerberos下：

![](http://p7wcdketk.bkt.clouddn.com/18-4-30/16836244.jpg)


再试着查看一下本机所安装的rsh版本信息：

    rpm -qf `which rsh`

显示为：

![](http://p7wcdketk.bkt.clouddn.com/18-4-30/52581489.jpg)

果然幕后黑手是kerberos

用rpm卸载后

立马生效，心情舒畅