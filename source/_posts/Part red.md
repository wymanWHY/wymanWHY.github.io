---
title: Linux系统启动过程
date: 2018-07-26 11:05:21
categories:  Linux OPS
tags: 
    - grub
    - 内核

---

Linux系统启动过程是个老生常谈的东西了，今天在整理文件中意外发现一张图：
![](http://p7wcdketk.bkt.clouddn.com/18-7-26/14947512.jpg)
一不小心想起了读研时Linux内核研究热潮那段时光，回忆起了当时和舍友在冬天睡前泡脚时一块讨论关于Linux系统启动时的场景
曾经热衷于技术的我们，不知道现在还有几位在技术一线工作着。
热情应该和脑子里关于Linux内核的知识一样，都退得差不多没了吧。


整个引导和启动过程分为三个阶段


## Part red ##
![](http://p7wcdketk.bkt.clouddn.com/18-7-26/74826546.jpg)
红色部分主要完成了硬件通电之后，初始化方面的工作：
- BIOS自检(POST)
- GRUB Stage1
- GRUB Stage1.5

### BIOS自检 ###

BIOS自检完全是硬件的活，机器通电后，硬件进入POST环节（Power On Self Test）,BIOS主要实现硬件的初始化，POST用于进行硬件基本功能是否正常，
POST完成后，BIOS进入INT 13H中断，进入可引导设备的引导区，通常情况下就是硬盘，从硬盘的引导扇区中读取MBR，启动MBR引导。

此后进入GRUB阶段。

### GRUB Stage1 ### 

引导代码和阶段阶段 1 代码非常小，因为它只存在于硬盘的第一个 512 字节的扇区中。在传统的常规 MBR 中，引导代码实际所占用的空间大小为 446 字节，这节代码通常被称为引导镜像（boot.img）所以，GRUB第一阶段，主要干了这么个活：
把引导代码加载到内存中，读取并执行。
受限于代码的体积，也只能干这么点活
之后，进入了GRUB 1.5阶段。

### GRUB Stage1.5 ###
当stage1还没加载stage1.5时，原则上是不能识别ext2的，当然也无法找到stage 1.5这个文件，所以，stage1.5必须存在于硬盘最前面的32K中，（在MBR之前,core.img），当stage1调用stage1.5时，就直接去该区域将stage1.5找出来使用。stage 1.5 的功能是开始执行存放stage 2 文件的 /boot 文件系统的驱动程序，并加载相关的驱动程序。

## Part yellow ##
![](http://p7wcdketk.bkt.clouddn.com/18-7-26/74826546.jpg)
### GRUB Stage 2 ###
Stage 2 是GRUB的核心程序，主要功能是定位和加载 Linux 内核到内存中，并转移控制权到内核。内核的相关文件位于 /boot 目录下，这些内核文件可以通过其文件名进行识别，其文件名均带有前缀 vmlinuz。statge 2 解析grub的配置文件/boot/grub/grub.conf，然后加载内核镜像到内存中，并将控制权转交给内核。
对GRUB来说，stage2除了不能自己启动外，剩下的事情全都由stage2完成。像是用户在启动时所看到的GRUB启动倒数画面，或是紧接着的启动菜单画面，就都是由stage2所提供的。

### 内核 ###
内核以一个压缩包的形式存在，通过加载到内存解压完成后，内核会立即初始化系统中各设备并做相关的配置工作，其中包括CPU、I/O、存储设备等，关于Linux的设备驱动程序的加载，有一部分驱动程序直接被编译进内核镜像中，另一部分驱动程序则是以模块的形式放在initrd(ramdisk)中。

initrd 的英文含义是 bootloader initialized RAM disk，就是由 boot loader 初始化的内存盘。在 linu2.6内核启动前，boot loader 会将存储介质中的 initrd 文件加载到内存，内核启动时会在访问真正的根文件系统前先访问该内存中的 initrd 文件系统。在 boot loader 配置了 initrd 的情况下，内核启动被分成了两个阶段，第一阶段先执行 initrd 文件系统中的init，完成加载驱动模块等任务，第二阶段才会执行真正的根文件系统中的 /sbin/init 进程。

initramfs 是在 kernel 2.5中引入的技术，实际上它的含义就是：在内核镜像中附加一个cpio包，这个cpio包中包含了一个小型的文件系统，当内核启动时，内核将这个 cpio包解开，并且将其中包含的文件系统释放到rootfs中，内核中的一部分初始化代码会放到这个文件系统中，作为用户层进程来执行
initrd中的init脚本主要是加载各种存储介质相关的设备驱动程序。当所需的驱动程序加载完后，会创建一个根设备，然后将根文件系统rootfs以只读的方式挂载，同时运行/sbin/init程序，执行系统的1号进程。此后系统的控制权就全权交给/sbin/init进程了


## Part blue ##
![](http://p7wcdketk.bkt.clouddn.com/18-7-26/74826546.jpg)
蓝色部分进入系统初始化阶段
### inittab ###
首先,/sbin/init读取/etc/inittab文件来执行相应的脚本进行系统初始化，如设置键盘、字体，装载模块，设置网络等。
### /etc/rc.d/rc.sysinit ###
执行系统初始化脚本(/etc/rc.d/rc.sysinit)，对系统进行基本的配置，以读写方式挂载根文件系统及其它文件系统，到此系统算是基本运行起来了，后面需要进行运行级别的确定及相应服务的启动
### /etc/rc.d/rc ###
执行/etc/rc.d/rc脚本。该文件定义了服务启动的顺序是先K后S，而具体的每个运行级别的服务状态是放在/etc/rc.d/rc\*.d（\*=0~6）目录下，所有的文件均是指向/etc/init.d下相应文件的符号链接。rc.sysinit通过分析/etc/inittab文件来确定系统的启动级别，然后才去执行/etc/rc.d/rc\*.d下的文件。
###　/etc/rc.d/rc.local ###
执行用户自定义引导程序/etc/rc.d/rc.local
### /sbin/mingetty ###
完成了系统所有的启动任务后，linux会启动终端或X-Window来等待用户登录。tty1,tty2,tty3...这表示在运行等级1，2，3，4的时候，都会执行"/sbin/mingetty"，而且执行了6个，所以linux会有6个纯文本终端，mingetty就是启动终端的命令。

到此，系统完成启动