---
title: 三步配置SSH无密码访问
date: 2015-01-04 11:52:27
categories: Linux OPS
tags:
	- ssh无密码访问
---

### 一、ssh-keygen -t rsa ###

### 二、ssh-add ~/.ssh/id_rsa ###

> 可以不用，主要是为了避免出现“Agent admitted failure to sign using the key”的情况


### 三、ssh-copy-id -i ~/.ssh/id_rsa.pub root@other_node ###
