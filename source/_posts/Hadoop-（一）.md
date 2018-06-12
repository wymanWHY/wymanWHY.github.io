---
title: Hadoop （一）
date: 2018-05-30 19:21:47
categories: 大数据
tags:
    - Hadoop
    - HDFS
    
---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1527689495125&di=0735139ba32a7f618368f6f4aae2e44d&imgtype=0&src=http%3A%2F%2Fimage.techweb.com.cn%2Fupload%2Froll%2F2016%2F02%2F16%2F201602163360_816.jpg)

<!--more-->

## Hadoop 安装配置 ##

### 配置hadoop-env.sh ###

hadoop启动时所需要的一些环境变量的设置，其中比较常用的环境变量的设置就是
JAVA_HOME。

```python
export JAVA_HOME=/root/soft/jdk1.8.0_92
```

### 配置core-site.xml ###
hadoop一些核心内容的设置，其中包括HDFS和MapReduce的基本设置。
```python
nfiguration>
<property>
    <name>fs.default.name</name>
    <value>hdfs://ivan.local:9000</value> #程序入口
</property>
<property>
    <name>hadoop.tmp.dir</name>
    <value>/root/log/hadoopLog/hadoop/tmp</value>
    <description>A base for other temporary directories.</description>
</property>
```

### hdfs-site.xml ###

HDFS的配置。

```
<configuration>
<property>
    <name>dfs.name.dir</name> #
    <value>file:///root/log/hadoop/dfs/name</value>  
    <description>Determines where on the local filesystem the DFS name node should store </description>
</property>
<property>
    <name>dfs.data.dir</name> #磁盘共享空间
    <value>file:///root/log/hadoop/dfs/data</value> #将该目录共享出去（绝对路径）
    <description>Determin. If this is a comma-delimited </description>
</property>
<property>
    <name>dfs.replication</name>
    <value>1</value>  #伪分布式，默认为3份
    <description>Default block replicied when the file is created. The default </description>
</property>
<property>
    <name>dfs.permissions</name> #权限认证，false禁止用户使用，true打开权限
    <value>false</value>
</property>

</configuration>
```

### mapred-site.xml ###

MapReduce的设置：

```
<configuration>

    <property>
        <name>mapreduce.framework.name</name> #任务提交到yarn上执行
        <value>yarn</value>
    </property>

</configuration>
```


### yarn-site.xml ###
```
<configuration>

<!-- Site specific YARN configuration properties -->
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>ivan.local</value>
</property>
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>

</configuration>
```


### 启动 ###
首先，记得格式化
```
bin/hadoop namenode - format
```

启动程序
```
$HADOOP/sbin/start-all.sh
```

查看启动结果：

```
[root@ivan sbin]# jps
10160 ResourceManager
9665 NameNode
10310 Jps
9767 DataNode
10264 NodeManager
9947 SecondaryNameNode
```

可以通过firefox界面来查看HDFS的整体情况:
`http://localhost:50070` 

![](http://p7wcdketk.bkt.clouddn.com/18-5-30/29629223.jpg)

8088端口则是YARN的运行平台情况
`http://localhost:8088` 

![](http://p7wcdketk.bkt.clouddn.com/18-5-30/42662646.jpg)


试着上传文件：

`hadoop fs -put /root/test /`

或者

`hadoop fs -put /root/test hdfs://ivan.local:9000`

查看上传的文件：

`hadoop fs -ls /`

![](http://p7wcdketk.bkt.clouddn.com/18-5-30/55925033.jpg)

或者`http://localhost:50070/ ->  Utilities -> Browse the file system`

![](http://p7wcdketk.bkt.clouddn.com/18-5-30/10031524.jpg)

## HDFS ##

### HDFS设计架构 ###

HDFS中的文件以块(block)方式存储， 每个块带下远比多数文件系统来的大(预设64M)

通过副本机制提高可靠度和读取吞吐量

每个区块至少分到三台DataNode上

使用m/s模式（主/从模式）单一 master (NameNode)来协调存储元数据(metadata)，因此需要注意防止出现NameNode节点的单点故障，做好HA

客户端对文件没有缓存机制 (No data caching)

HDFS主要组件分为两个：`NameNode` 和 `DataNode`:

|NameNode|DataNode|
|:-|:-|
|存储元数据|存储文件内容|
|元数据保存在内存中|文件内容保存在磁盘上|
|保存文件-block-datanode之间的映射关系|维护block id到datanode本地文件的映射关系|

Block大小和副本数由Client端上传文件到HDFS时设置，其中副本数可以变更， Block大小是不可以再上传后变更的。因为当DN读取block的时候，它会计算checksum， 如果计算后的checksum，与block创建时值不一样，说明该Block已经损坏。
- client读取其它DN上的block； NN标记该块已
经损坏，然后复制block达到预期设置的文
件备份数
- DN在其文件创建后三周验证其checksum

### NameNode(NN) ###

NameNode主要功能提供供名称查询服务，它是一个jetty服务器，NameNode保存metadate信息，包括
- 文件owership和permissions
- 文件包含哪些块（Block）
- Block保存在哪个DataNode（由DataNode启动时上报）

NameNode的metadate信息在启动后会加载到内存，metadata存储到磁盘文件名为”fsimage”，在运行的时候加载到内存中的(读写比较快)，操作日志写到edits中（类似于LSM树中的log）。因此，NN中的fsimage+edits构成了HDFS系统的存储拓扑。如果NN宕机，恢复后可以通过fsimage和edits恢复系统。

要注意的是：
- Block的位置信息不会保存到fsimage
- NameNode的metadate信息在启动后会加载到设置一个**Block 64MB**，如果上传文件小于该值，**仍然会占用一个Block的命名空间（NameNode metadata）**，**但是物理存储上不会占用64MB的空间**



### DataNode（DN） ###

DataNode保存Block，也就是文件块, 启动DN线程的时候会向NN汇报block信息

通过向NN发送心跳保持与其联系（ 3秒一次），如果NN 10分钟没有收到DN的心跳，则认为其已经lost，并copy其上的block到其它DN（待机制触发后进行拷贝）

如果故障节点恢复，因为故障时系统会重新拷贝宕机DN中的Block到其他DN，因此实际上系统中的副本总数会比之前多出一份，而多出来的副本系统并不处理（Hadoop只管少、不管多）

**注意：DN只有在启动时才将block信息上报至NN，启动后与NN的心跳通信只是用于告知系统——DN还存活**



### Block的副本放置策略 ###

第一个副本：放置在上传文件的DN；如果是集群外提交，则随机挑选一台磁盘不太满， CPU不太忙
的节点

第二个副本：放置在于第一个副本不同的机架的节点上

第三个副本：与第一个副本相同机架的其他节点上

更多副本：随机节点

### HDFS应用场景 ###


|适用|不适用|
|:-|:-|
|超大文件|低延时的数据访问|
|流式数据访问：一次写入，多次读取；传输时间与寻址时间|大量小文件|
|商用硬件|多用户写入，任意修改|

**HDFS不是万能的，但是搭配Hadoop生态链上的其他产品，可以满足绝大多数的业务需求**


### 安全模式 ###

namenode启动的时候，首先将映像文件(fsimage)载入内存，并执行编辑日志(edits)中的各项操作。一旦在内存中成功建立文件系统元数据的映射，则创建一个新的fsimage文件(这个操作不需要SecondaryNameNode)和一个空的编辑日志。之后NameNode开始监听RPC和HTTP请求。此时NameNode运行在安全模式。即namenode的文件系统对于客服端来说是只
读的。 (显示目录，显示文件内容等。写、删除、重命名都会失败)。系统中数据块的位置并不是由namenode维护的，而是以块列表形式存储在datanode中。即namenode的文件系统对于客服端来说是只读的。 (显示目录，显示文件内容等。写、删除、重命名都会失败)。系统中数据块的位置并不是由namenode维护的，而是以块列表形式存储在datanode中。在系统的正常操作期间， namenode会在内存中保留所有快位置的映射信息。

#### 安全模式的相关命令 ####

查看`namenode`处于哪个状态：
    `hadoop dfsadmin –safemode get`

进入安全模式(hadoop启动的时候是在安全模式)：
    `hadoop dfsadmin –safemode enter`

离开安全模式：
    `hadoop dfsadmin -safemode leave`


### HDFS文件的读写 ###

- 读：文件上传只HDFS后，系统中会存在3份副本，在client发出读取请求时，系统会选择最优的那份副本，返回给client
- 
- ![](http://p7wcdketk.bkt.clouddn.com/18-5-31/86102844.jpg)

- 写：在client发起写请求后，首先告知要写的文件大小，NN则返回它可以写入的地址，当真正执行写操作的时候，client只执行一次写操作，写入到其中给定的一个DN后，剩余的2份副本，则由HDFS系统自行完成复制。以此减少client的io压力。

![](http://p7wcdketk.bkt.clouddn.com/18-5-31/62267340.jpg)

> **如果要实现大文件的高效写入，可以通过划分为多个client将文件分块完成写入**

当一个Block大小为64M，共3个节点，自动存储文件副本为2份的HDFS系统，存储两份文件，一份大小为156M，一份128M，那么这两份文件在这个HDFS系统中的存放模样，是这样的：

![](http://p7wcdketk.bkt.clouddn.com/18-5-31/17722590.jpg)

> 156M的文件，共需要3个Block;128M的文件，会占用2个Block，因此，系统共会产生5个Block完成2份文件的存储。


## Tips ##

#### A ####

HDFS在系统中共为文件保存了3份副本，完全可以不必再做raid了，如果非要做，倒是可以在NN上做，用于保存fsimage数据的丢失

#### B ####

HDFS只能做到组级别的权限管理

#### C ####

当NN将块标记为missing block后，副本全部损坏，无法读取


#### D ####

NN宕机后恢复，需要时间>45min

#### E ####

官方称2.5版本之上已经解决脑裂问题，但还是需要保证配置的正确

#### F ####

大数据中只有CPU可以虚拟化，内存不可虚拟化


#### G ####

集群服务器的性能尽量平均，**如果做不到，则高配做计算，低配做存储**

而当各服务器CPU，内存大小一致，只是存储空间不一致，则容易造成“短板效应”，以最小空间的为准，等到最小空间的节点写满，则其他节点也没法执行写操作

#### H ####

当出现数据倾斜时，开启`start-balancer.sh` 脚本