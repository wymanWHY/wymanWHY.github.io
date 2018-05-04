---
title: shc-将shell脚本封装为二进制程序
date: 2018-05-03 19:36:29
categories: shell
tags:
    - shell
---


脚本的好处是便捷、高效，拿起来就可以写，写完就能跑，都不用编译

但坏处也显而易见，一些敏感的、不想让外人知道的东西都是明文写在里面的，所以，在这推荐一款**神奇的脚本封装程序——shc** ： 



![](http://p7wcdketk.bkt.clouddn.com/18-5-3/43639958.jpg)



作者是位长得有几分像[皮克](https://baike.baidu.com/item/%E6%9D%B0%E6%8B%89%E5%BE%B7%C2%B7%E7%9A%AE%E5%85%8B/6479448?fr=aladdin&fromid=15433506&fromtitle=%E7%9A%AE%E5%85%8B)的帅气西班牙人

<a id="download" href="http://www.datsi.fi.upm.es/~frosal/"><i class="fa fa-download"></i><span> 下载地址</span>
</a>

下载后解压，一顿简单操作即可使用：

```bash
tar xzvf shc.tar.tgz 
cd shr/
make && make install 
```

贴出官方给出的参数和解释：

```bash
OPTIONS

     The command line options are:

     -e date
          Expiration date in dd/mm/yyyy format [none]

     -m message
          message to display  upon  expiration  ["Please  contact
          your provider"]

     -f script_name
          File name of the script to compile

     -i inline_option
          Inline option for the shell interpreter i.e: -e

     -x comand
          eXec    command,    as    a    printf    format    i.e:
          exec(\\'%s\\',@ARGV);

     -l last_option
          Last shell option i.e: --

     -r   Relax security. Make  a  redistributable  binary  which
          executes  on different systems running the same operat-
          ing system.

     -v   Verbose compilation

     -D   Switch on debug exec calls

     -T   Allow binary to be  traceable  (using  strace,  ptrace,
          truss, etc.)

     -C   Display license and exit

     -A   Display abstract and exit

     -h   Display help and exit

```

> 就不翻译了...

**日常用法：**

```bash
shc -r -f /shellfile.sh
```
运行成功后会在当前目录下生成两个文件：

- shellfile.sh.x
- shellfile.sh.c

`shellfile.sh.x`是脚本所对应的可执行程序

`shellfile.sh.c`是`shellfile.sh.x`对应的c语言实现的源码

shc根据脚本文件的第一行`#!/bin/bash`或其他shell将脚本翻译成相应的c源码并生成可执行程序。


但shc似乎无法识别expect

**封装后的脚本安全性会有所提高，但这也仅能防个君子，通过gdb或其他调试工具仍然能获得最初的源码**
