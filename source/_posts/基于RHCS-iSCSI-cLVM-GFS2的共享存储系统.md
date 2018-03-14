---
title: 基于RHCS+iSCSI+cLVM+GFS2的共享存储系统
date: 2016-12-12 10:02:13
categories: Linux OPS
tags:
	- RHCS
	- iSCSI
	- CLVM
	- GFS2
	- 集群
	- 存储
---

{% asset_img 00.jpg %}

共享存储是集群的重要组成部分之一，iSCSI使用TCP/IP网络协议完成原本需要FC光纤通道完成的功能，利用网络实现磁盘等块级存储设备在网络上的存储

### iSCSI的 组成 ###

一个简单的iSCSI系统主要分以下两部分：

- iSCSI target 
- iSCSI Initiator
> （当然还有交换机、HBA卡之类的其他部分）

 **iSCSI target**

iSCSI target 可以理解为用于存储数据的磁盘阵列或具有iSCSI功能的其他存储设备，大多数操作系统都可以通过软件将系统变为一个iSCSI target，将iSCSI target软件安装在PC服务器上，即可实现将PC转变为一台具有iSCSI功能的存储服务器。

**iSCSI Initiator**

iSCSI Initiator是iSCSI的驱动程序，通过安装initiator可以实现iSCSI服务器与iSCSI存储设备的连接，当然，也可以使用HBA卡来完成这项功能。

所以，使用PC服务器来部署一套完整的iSCSI系统，需要安装**iSCIS target** 与 **iSCSI initiator**

-----

### 前奏 ###

- 环境

| 节点 | 系统 | IP |
| ----- | ------ | :----- |
| node1 | RHEL6.8 | 192.168.200.1 |
| node2 | RHEL6.8 | 192.168.200.2 |
| node3 | RHEL6.8 | 192.168.200.3 |
| EPR | RHEL6.8 | 192.168.200.253 |

- 完成[节点间SSH无密码访问](http://wyman.wang/2015/01/04/%E4%B8%89%E6%AD%A5%E9%85%8D%E7%BD%AESSH%E6%97%A0%E5%AF%86%E7%A0%81%E8%AE%BF%E9%97%AE/)

> 环境如上，完成后可以开始正式的部署工作

----

### 一、部署RHCS系统 ###


>  iSCSI共享存储在多用户同时读写的过程中会出现互斥和无法同步的问题，因为是块级别的共享，这个问题无法通过SCSI设备自身解决，需要依赖集群文件系统所提供的锁管理机制（比如RHCS中的DLM）来解决读写互斥和同步的问题，这里使用RHCS套件所提供的GFS文件系统，这也是为啥第一步要先搭建RHCS环境的原因

**1、安装RHCS集群套件**

各节点（node1-node3）需要安装的套件包：

- cman
- rgmanager
- ricci
- gfs2
- lvm2

RHCS集群套件在系统光盘上自带，使用yum即可直接进行安装：

    yum install -y cman* gfs2* rgmanager* ricci* lvm2*

> 其实安装了rgmanager就自动安装好了cman、ricci


启动上述安装的各类服务：

	service cman start
	service rgmanager start
	service ricci start

在管理节点上（EPR:192.168.200.253）安装RHCS集群管理界面luci:

    yum install luci* -y


**2、通过luci配置集群**

启动Luci服务后即可使用浏览器登录https://EPR:8084完成集群的配置与管理

> https://EPR:8084可以输入本机的root账户密码进行登录

在“ManagerCluster”中选择“Create”创建一个集群

{% asset_img 6.JPG %}

经过简单的账号密码节点名的输入配置后,集群成功创建并启动：

{% asset_img 7.JPG %}

> **坑1**：在各节点上创建ricci用户，并创建密码，可避免出现ricci启动失败的情况；

> **坑2**：cman启动失败,将失败节点中的NetworkManage服务关闭后解决; 

> **坑3**：关闭防火墙！


稍等一会，等圈圈转完之后，集群创建成功：

{% asset_img 8.JPG %}

### 二、iSCSI配置 ###



> 之前说到，一个iSCSI系统包含target端和initiator端，这里将作为iSCSI设备的EPR服务器（192.168.200.253）作为存储设备，给它安装target软件包：scsi-target-utils；将node1-node3作为initiator端，安装iscsi-initiator-utils软件包；同样，两个软件包都可以直接yum install

**1、target端配置(192.168.200.253)：**

1）启动target后台服务进程tgtd：
```
    service tgtd start
```

2）创建target
```
    tgtadm -L iscsi --op new --mode target --tid 1 -T iqn.2018-03.com:home 
```

> **说明**: 
> --op:做什么操作，new,delete,show等；
> --mode:针对哪个对象，有session,node,discovery等；
> --tid：定义target的ID；
> -T 指定创建的target叫啥；
> -T参数后接的一串为target在全网中的名称，必须唯一；

3）创建LUN
	tgtadm -L iscsi --op new --mode logicalunit --tid 1 --lun 1 -b /dev/sdb
	
>**说明**：
>--mode logicalunit：嗯，LUN嘛	
>--lun:指定lun的标识ID，此处为1
>-b: 指定划分lun的分区

4）配置访问权限 
	tgtadm -L iscsi --op bind --mode target --tid 1 -I 192.168.200.0/24
> 说明：
> 此处只打开192.168.200.0网段的访问权限，3260端口开启监听；可以通过--user参数加--password参数创建访问用户的权限  

5）查看划分结果
    tgtadm -L iscsi --op show -m target
{% asset_img 4.JPG %}
> **说明**：
> lun0 默认被target驱动器所占用，所有的lun编号从1开始。lun1已经成功创建，由/dev/sdb分区提供空间，所有在192.168.200.0网段中的initiator都可共享访问；
> **Tips**:
> 删除target1中的lun1：tgtadm -L iscsi --op delete --mode logicalunit --tid 1 --lun 1 
> 删除target1的操作：tgtadm -L iscsi --op delete --mode target --tid 1

**2、initiator端配置(node1-node3)**

首先，各节点启动iscsi服务：service iscsi start

1）开启target发现
    iscsiadm -m disvovery -m sendtargets -p 192.168.200.253:3260
> 监听后如果发现网段中存在可访问的target则返回成功的消息：
{% asset_img 0.JPG %}
> **tips**: 成功执行一次target发现后，initiator节点会在/var/lib/iscsi/send_targets目录下生成相关的target信息，下次服务启动会自动连接该target
> **坑**：执行target发现后，节点重启，再启动iscsi服务失败，尝试着删除/var/lib/iscsi/send_targets所有目录，再启动依旧失败，后来重新执行了一次target发现后，iscsi服务启动成功

2）连接所发现的target
	iscsiadm -m node -T iqn.2018-03.com:home -p 192.168.200.253 -l
	
{% asset_img 1.JPG %}
> 或者： iscsiadm -m node --loginall=all #连接所发现的所有target
	
> **tips**: 
> 查询已连接的target信息：iscsiadm -m session -s 
> 断开已连接的target：iscsiadm -m node -T iqn.2018-03.com:home -p 192.168.200.253 -u

3）查看是否成功连接共享存储
	执行fdisk -l，可以发现所有节点都新增了一块磁盘，说明之前的发现操作和连接操作都已成功，共享存储出现在了各个节点上
	
{% asset_img 3.JPG %}
	
> 当然，我们也可以在target上查看已连接该target的所有initiator：
    tgt-admin --show

{% asset_img 19.JPG %}

**到这，iSCSI的配置就完成了**

### 三、GFS配置 ###

GFS与CLVM都是RHCS的组件，GFS提供集群文件系统，CLVM完成集群逻辑卷的管理功能，使得集群中的机器可以用LVM管理共享存储，所以，在创建GFS之前，必须先完成CLVM的部署。

**1、CLVM的配置**

前面已经在各节点安装了clvm，在启动clvm之前，先将clvm中的锁机制设置为集群模式

可以直接修改/etc/lvm/lvm.conf文件中的“locking_type”值改为“3”，也可以直接通过命令完成

    lvmconf --enable-cluster

完成修改后，启动clvm服务：

    service clvmd start

{% asset_img 9.JPG %}

> 当然，所有集群节点都得启动

**2、创建逻辑卷**

在完成了iSCSI的配置后，我们在各节点上都发现多了一块2T的本地磁盘，该磁盘即为192.168.200.253作为target的形式所共享出来的磁盘，所有节点通过initiator发现和连接操作，将共享磁盘映射为本地/dev/sdb分区，我们将该共享磁盘以vg的形式分出去，创建lv再挂载至节点上工集群使用，而集群中的任一节点对该共享磁盘进行的LVM操作都会同步到其它节点，所以以下的操作在任一集群节点上完成即可.

```
pvcreate /dev/sdb
vgcreate vg_cluster /dev/sdb
lvcreate -L 2T -n lv_cluster_home
...
```

> 如果没出什么意外，我们成功创建了一个名为vg_cluster的卷组，在这个卷组中只有1个逻辑卷，名称为lv_cluster_home,大小为2T

各种查看命令走一波：
```
    vgs
    pvs
    lvs
```
{% asset_img 20.JPG %}

确认都OK

**3、创建GFS文件系统**

接下来将逻辑卷格式化为gfs2文件系统，gfs2文件系统具有锁机制，允许多个节点对同一个磁盘分区同时读写

```
    mkfs.gfs2 -p lock_dlm -j 6 -t clustername:my_gfs /dev/vg_cluster/lv_cluster_home
```
> 说明：
> -p lock_dlm :设置DLM锁方式，如果没有此参数，当两个或以上的系统同时挂载此分区是就会想ext3格式一样，两个系统的信息不能同步
> -t  clustername:my_gfs ：指定DLM锁所在的表的名称，clustername就是第一步中所创建的RHCS集群名称，必须和/etc/cluster/cluster.conf中的Cluster标签上的name一致
> -j 6 ：设置了6个日志系统，也就是说允许最多6个节点同时挂载
> tips: 增加日志数量  gfs2_jadd -j 8 /dev/vg_cluster/lv_cluster_home
> 查看可挂载的节点数   gfs2_tool journals /dev/vg_cluster/lv_cluster_home
> {% asset_img 18.JPG %}

在弹出的对话框中，选择y，没有意外的话，文件系统格式化成功

{% asset_img 16.JPG %}

---

**验证一下**

在node1上将/dev/vg_cluster/lv_cluster_home挂载到/home下面，新建一个名为1的文件：

	mount -t gfs2 /dev/vg_cluster/lv_cluster_home /home
	touch 1


在node2上也将/dev/vg_cluster/lv_cluster_home挂载到/home下面，ls一下，发现1文件存在

{% asset_img 21.JPG %}


在node1上的1文件上写点东西，看看node2上能否立刻同步

    echo "hello gfs2 to every nodes" > 1

在node2上查看一下1文件
	
{% asset_img 22.JPG %}

OK,node2上也有了


**好了，基于RHCS+iSCSI+cLVM+GFS2的共享存储已经正常运行了**
 

    
