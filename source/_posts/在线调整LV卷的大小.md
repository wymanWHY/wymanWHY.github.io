---
title: 在线调整LV卷的大小
date: 2015-04-11 17:05:48
categories: Linux OPS
tags:
	- LVM
	- Linux
	- resize2fs
---

/tmp目录不知道是哪个挨千刀的设置的，容量太小，时不时就满；

服务器磁盘无法立即扩容，但幸好其他卷的剩余空间足够大，于是，我们想着从隔壁卷(/backup)抽调一些空间出来，分配给/tmp所在LV卷

大致分为两个环节：

# 一、缩减其他LV卷的空间

## 1、卸载需要缩减空间的LV卷所挂载的文件系统

首先，df -kh 查看一下磁盘使用情况
```bash
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-LogVol00
                       40G  26G   12G  70% /
     ...
/dev/sda1             485M   37M  423M   8% /boot
/dev/mapper/VolGroup-LogVol03
                       3G  2.9G   120M   99% /home
/dev/mapper/VolGroup-backup
                       200G  100G   100G   50% /backup
```
将VolGroup-backup卷从文件系统上umount掉
```bash
umount /backup
```

## 2、缩减VolGroup-backup所在的文件系统大小
**这一步很重要，如果没有进行文件系统的缩减而直接对LV卷进行操作，会破坏原有的数据**

我们的需求是将卷VolGroup-backup的空间挤压出几个G来，于是：
```bash
resize2fs /dev/mapper/VolGroup-backup 190G
```


> (将VolGroup-backup缩减至190G，腾出大约10个G的空间)

## 3、缩减VolGroup-backup的LV卷
```bash
lvreduce -L -3G  /dev/mapper/VolGroup-backup
```
> (注意区别：-3G—容量缩减3G；3G—容量调整成3G)


我们可以通过lvs或者vgs查看一下LV卷的整体情况

在vgdislay中可以看到Free PE字段显示剩余空间已经有了几个G

**OK，至此已经完成第一个环节：缩减空间**

# 二、增加目标卷的空间

## 1、扩展卷 VolGroup-backup
```bash
lvextend -L 3G   /dev/mapper/VolGroup-backup
```
## 2、增加VolGroup-backup文件系统空间

```bash
resize2fs /dev/mapper/VolGroup-backup
```

别忘了把/dev/mapper/VolGroup-backup重新挂载到原目录下/backup
```bash
mount /dev/mapper/VolGroup-backup /backup
df -kh # 可以查看一下是否成功
```
**至此，/tmp目录容量大小调整完成**



> **linux kernel 2.6支持在mount状态下扩容但仅限于ext3文件系统**



> 注意resize2fs操作的先后顺序：
>**在缩容前，记得先执行resize2fs再进行lvreduce；**
> **在扩容时，先执行lvextend再进行resize2fs才能生效。**
> 好比天气回温脱秋裤，脱之前，咱先得把牛仔裤脱了；天冷加秋裤之后，咱也别忘了把牛仔裤穿回去


