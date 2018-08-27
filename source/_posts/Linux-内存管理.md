---
title: Linux 内存管理
date: 2018-08-27 09:11:21
categories: Linux OPS
tags:
    - 内存
    - cache
    - buffer
    - swap
---


总结一下Linux下的内存机制，搞清楚swap、cache、buffer几个耳熟能详的概念。

## free 命令 ##

敲一下`free`命令，可以看到目前内存的使用情况**(RHEL6.8)**：
```bash
free 
```

可以看到total、used、free、buffers/cache、buffers、cached几个指标：

```
[root@GDA-master Desktop]# free
             total       used       free     shared    buffers     cached
Mem:       1915236    1170096     745140      15600      53936     330156
-/+ buffers/cache:     786004    1129232
Swap:      2433020          0    2433020
```

这三行分别描述了：

- 1、Mem：从内核层面看到的内存使用情况；

- 2、-/+ buffers/cache：反映了程序实实在在吃掉的内存情况；

- 3、Swap：交换分区的使用情况；

拿出计算器，验证一下发现：
```
             total       used       free     shared    buffers     cached
Mem:       1915236    1170096     745140      15600      53936     330156
```
在Mem这一行中，total(1915236) = used(1170096) + free(745140)，也就是说**物理内存总量=使用量+剩余量**

这解释没毛病。

我们还发现，第二行的-/+ buffers/cache中：
```
             total       used       free     shared    buffers     cached
Mem:       1915236    1170096     745140      15600      53936     330156
-/+ buffers/cache:     786004    1129232
```
total(1915236) = used(786004) + free(1129232)，从应用的层面来看，已分配的缓存/冲量 + 剩余缓存/冲量 = 实际内存大小

另外，我们还发现：**1129232(free buffers/cache) = 745140(free Mem) + 53936(buffers) + 330156(cached)**
```
             total       used       free          shared      buffers        cached
Mem:       1915236    1170096     **745140**      15600      **53936**     **330156**
-/+ buffers/cache:     786004    **1129232**
```
也就是剩余的缓存/冲空间 = 剩余物理内存 + 已用缓冲量 + 已用缓存量。

这说明了啥？

**说明对应用程序来说，使用了的buffer和已经使用的cache，其实都是可用内存，当程序要使用内存的时候，buffer和cache可以随时回收。所以这部分也被认定为空闲空间**


## buffer ##

缓冲——用于处理内存和硬盘之间的写的效率问题。

buffer集中处理原本分散的写操作，减少磁盘碎片和硬盘的反复寻道，提升系统性能。通过buffer可以减少进程间通信需要等待的时间，当存储速度快的设备与存储速度慢的设备进行通信时，存储慢的数据先把数据存放到buffer，达到一定程度存储快的设备再读取buffer的数据，在此期间存储快的设备CPU可以干其他的事情

**buffer解决了空间上的问题**。

## cache ##

缓存——用于缓解CPU和内存之间的读速度匹配问题。

cache把读取过的数据保存起来，重新读取时若命中（找到需要的数据）就不要去读硬盘了，若没有命中就读硬盘。其中的数据会根据读取频率进行组织，把最频繁读取的内容放在最容易找到的位置，把不再读的内容不断往后排，直至从中删除。


**cache解决的时间上的问题**。

## swap ##

swap是从硬盘中开辟出来的一块专门用于对内存页进行交换的空间。也就是所谓的虚拟内存。

当物理内存不够用的时候，内核就会释放缓存区（buffers/cache）里一些长时间不用的程序，然后将这些程序临时放到Swap中，也就是说如果物理内存和缓存区内存不够用的时候，才会用到swap。

更详细的说，**内核会将暂时不用的内存块信息写到交换空间，这样以来，物理内存得到了释放，这块内存就可以用于其它目的，当需要用到原始的内容时，这些信息会被重新从交换空间读入物理内存。**

Linux 进行页面交换是有条件的，不是所有页面在不用时都交换到虚拟内存，linux内核根据“最近最经常使用”算法，仅仅将一些不经常使用的页面文件交换到虚拟内存。

有时我们会看到这么一个现象：linux物理内存还有很多，但是交换空间也使用了很多。例如：一个占用很大内存的进程运行时，需 要耗费很多内存资源，此时就会有一些不常用页面文件被交换到虚拟内存中，但后来这个占用很多内存资源的进程结束并释放了很多内存时，刚才被交换出去的页面文件并不会自动的交换进物理内存，除非有这个必要，那么此刻系统物理内存就会空闲很多，同时交换空间也在被使用，就出现了刚才所说的现象了。

交换空间的页面在使用时会首先被交换到物理内存，如果此时没有足够的物理内存来容纳这些页面，它们又会被马上交换出去，如此一来，虚拟内存中可能没有足够空间来存储这些交换页面，最终会导致linux出现假死机、服务异常等问题，linux虽然可以在一段时间内自行恢复，但是恢复后的系统已经基本不可用了。

## Linux 内存机制 ##
Linux服务器运行一段时间后，由于其内存管理机制，会将暂时不用的内存转为buff/cache，这样在程序使用到这一部分数据时，能够很快的取出，从而提高系统的运行效率，所以这也正是linux内存管理中非常出色的一点，所以乍一看内存剩余的非常少，但是在程序真正需要内存空间时，linux会将缓存让出给程序使用，这样达到对内存的最充分利用，所以真正剩余的内存是free+buff/cache。但是有些时候大量的缓存占据空间，这时候应用程序回去使用swap交换空间，从而使系统变慢，这时候需要手动去释放内存，释放内存的时候，首先执行命令 sync 将所有正在内存中的缓冲区写到磁盘中，其中包括已经修改的文件inode、已延迟的块I/O以及读写映射文件，从而确保文件系统的完整性

## 内存释放 ##

可以通过修改`/proc/sys/vm/drop_caches`来释放内存，`drop_caches`有三种状态：

```
Writing to this will cause thekernel to drop clean caches, dentries and  
    inodes from memory, causing thatmemory to become free.  
    To free pagecache:  
             echo 1 > /proc/sys/vm/drop_caches  
    To free dentries and inodes:  
             echo 2 > /proc/sys/vm/drop_caches  
    To free pagecache, dentries andinodes:  
             echo 3 > /proc/sys/vm/drop_caches  
    As this is a non-destructiveoperation and dirty objects are not freeable, the  
    user should run `sync` first.  
    http://www.kernel.org/doc/Documentation/sysctl/vm.txt  
```

 - 1表示清空页缓存;
 - 2表示清空inode和目录树缓存;
 - 3清空所有的缓存。
 
 默认情况下，`drop_caches`为`0` ：
 
 
 ### [参考] ###
 
 [linux下的缓存机制及清理buffer/cache/swap的方法梳理](https://www.cnblogs.com/kevingrace/p/5991604.html)
 [Linux-内存管理机制、内存监控、buffer/cache异同](https://www.cnblogs.com/JohnABC/p/5799781.html)
 [linux swap理解-为什么linux有足够的内存还进行swap](https://blog.csdn.net/aibisoft/article/details/21193899)
