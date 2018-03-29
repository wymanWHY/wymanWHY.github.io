---
title: 故障：resize2fs 报错
date: 2015-05-29 15:00:42
categories: Linux OPS
tags:
---

对LVM进行扩容的最后，用resize2fs对相关卷生效的时候，报错信息：

> **Bad magic number in super-block while trying to open xxxx
Couldn’t find valid file system superblock**

#### 原因： ####

**系统使用不是ext3文件系统**

#### 解决办法： ####

- 查看当前系统下的该卷用的是什么文件系统

`cat /etc/fstab` 

发现服务器中/dev/mapper/VolGroup00/disk1所用的是reiserfs

- 用命令**resize_reiserfs / dev/mapper/VolGroup00/disk1**即可解决
