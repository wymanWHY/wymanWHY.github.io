---
title: sed高级使用教程
date: 2016-04-18 17:39:07
categories: shell
tags:
    - sed
    - shell
    - bash
---


sed中的一些其他概念

## 缓冲空间 ##

sed 中维护着两个缓冲空间：

- pattern space：模式空间，用于存放读取到的内容
- hold space：保持空间，用于暂时存储模式空间传过来的内容

sed 简单的操作流程是这样处理文本文件的：

- 首先将文本内容的第一行读入模式空间中；
- 然后执行相关的处理命令；
- 紧接着将模式空间处理后的内容输出；
- 最后删除模式空间中的内容；

就这样不断的执行这样四个步骤直至将文本文件中所有的行都读取并操作完成

![](http://p7wcdketk.bkt.clouddn.com/18-5-8/52863728.jpg)

而当我们需要做一些复杂一点的操作时，仅仅一个模式空间并不够，所以出现了保持空间：

- 首先将文本内容的第一行读入模式空间中；
- 然后执行相关的处理命令：
  - 将内容放置保持空间，暂存；
  - 执行相关处理；
- 紧接着将模式空间处理后的内容输出；
- 最后删除模式空间中的内容；

![](http://p7wcdketk.bkt.clouddn.com/18-5-8/33545906.jpg)

> 注意：两个空间在只能存储一行的内容，读取下一行内容时，若是空间中有内容只会被覆盖掉，要想存储两行的内容必须使用追加操作。追加操作的实质是在已有的内容后添加换行符，然后添加内容。

模式空间很好理解，在之前使用 sed 的例子中都是使用的模式空间处理相关的问题，那么什么时候我们会用到保持空间呢？

例如我们想将文章中的语句倒叙，也就是本是第一句的内容最后输出，本该最后一句输出的第一句输出.



### 多行空间 ###

多行空间，顾名思义，就是可以存储多行内容，可以通过换行符`\n`来区分。

`sed` 允许匹配扩展到多行。包含 3 个多行命令 `(N,D,P)`

|命令|意思|
|:-|:-|
|N|追加下一行|
|D|删除多行模式空间的第一行|
|P|打印多行模式空间中的第一行|

#### 追加下一行 ####

之前我们处理的字符串都在一行，当我们想要处理的字符串被断开了，比如我们想要将下面名叫 `zuijia` 的实验文本中的 `modify that document` 替换成 `modify this document` ，但字符串不在一行上，`that` 被放在了下一行，这个时候我们就需要读取下一行的内容。

我们通过如下实践来理解，我们首先新建一个名叫 `sedn` 的脚本文件，实现把 `zuijia` 这个文本中的 `modify that document` 替换成 `modify this document`。

新建一个 zuijia ，内容如下：

```
Permission is granted to copy, distribute and/or modify
that document under the terms of the GNU Free Documentation License, Version 1.3 or any later version published by the Free Software Foundation; 
with no Invariant Sections, 
no Front-Cover Texts, and no Back-Cover Texts. A copy of the license is included in the section entitled “GNU Free Documentation License
```

新建一个 `sedn` ,内容如下：
```
N
s/modify\nthat/modify this/
```
它的实现流程为：
- 首先 sed 读取第一行内容放在模式空间中
- 然后 N 命令将下一行内容追加在模式空间中，而第一句与第二句之间就间隔了一个换行符
- 然后执行替换的命令

若是没有追加的话，这里正则表达式将无法匹配，也就没有办法做替换了

#### 删除多行中第一行 ####

通过 D 命令我们可以删除多行中的第一行

#### 多行打印 ####

打印命令 p 会打印模式空间中的所有内容，而打印命令 P 则输出多行模式空间的第一部分，也就是第一个换行符之前的内容。

我们通过以下实践来验证 `P` 的功能：

**我们将匹配 UNIX 结尾的行，并且如果下一行是以 `System` 开始，那么在 `UNIX` 之后加上 `Operating`。**

首先我们新建一个文本 `testprin`，内容如下：

```
Here are examples of the UNIX
System. Where UNIX
System appears, it should be the UNIX
Operating System.
```

我们新建一个脚本文件 `prin` 来替换 `System` 为 `Operating System`，内容如下：

```
/UNIX$/{
  N
  /\nSystem/{
    s// Operating &/
    P
    D
  }
}
```


> & 意思是用分组中正则表达式的内容替换掉匹配的内容。


> // 之间不加任何内容，表示的是上一次的匹配。在这例子中就是 \nSystem

执行过程如下：

首先读入一行。如果结尾匹配 UNIX，那么执行 N 命令读入下一行。

1、如果匹配到 \nSystem，那么将 `System` 替换成 `Operating \nSystem`，然后打印出替换之后的第一行，并把模式空间中的第一行删除。
2、如果下一行不是 System 开头，那么完成匹配 UNIX 分组的操作，继续读取下一行进行匹配。

![](http://p7wcdketk.bkt.clouddn.com/18-5-9/76581209.jpg)

### 保持空间 ###

|命令|缩写|功能|
|:-|:-|:-|
|Hold|h 或 H|将模式空间的内容复制或追加到保持空间|
|Get|g 或 G|	将保持空间内容复制或追加到模式空间|
|Exchange|x|交换保持空间和模式空间的内容|

> !，表示的是指定行不执行后续的操作，其他所有行都要执行

我们通过上述的倒序实践来深入理解：

新建一个名为 reverse.sed 的脚本：

```bash
#!/bin/sed -nf

# 从第二行开始将保持空间的内容追加到模式空间中
1! G

# 直到最后一行才打印
$ p

# 将模式空间的内容复制到保持空间
h
```

然后为该脚本添加执行权限，然后我们执行：

    ./reverse.sed words


![](http://p7wcdketk.bkt.clouddn.com/18-5-9/83600156.jpg)

我们来看其运行的流程：

- 首先 sed 读取第一行数据 `hahaha`
- 第一个命令 1! G 表示不是第一行的内容才会执行 G 命令，所以此时不操作
- 第二个命令 $ p 表示最后一行才打印，所以此时依旧不操作
- 第三个命令 h 表示将模式空间的内容复制到到保持空间中
  - 此时模式空间内容是：`hahaha`
  - 此时保持空间内容是：`hahaha`
- 第一行处理完毕
- 读取第二行内容 `yeyeye`
  - 此时模式空间内容是：`yeyeye`
  - 此时保持空间内容是：`hahaha`
- 第一个命令 1! G 表示不是第一行的内容才会执行 G 命令，所以此时会把保持空间的内容追加到模式空间
 - 此时模式空间内容是：`yeyeye\nhahaha`
 -此时保持空间内容是：`hahaha`
- 第二个命令 `$p` 表示最后一行才打印，所以此时便打印模式空间的内容
- 第三个命令 h 表示将模式空间的内容复制到到保持空间中
  - 此时模式空间内容是：`yeyeye\nhahaha`
  - 此时保持空间内容是：`yeyeye\nhahaha`
> 此例可以很好地帮助理解sed的行操作流程，通过读取文本的每一行，依次执行sed中的相关操作



---

参考文档：

- [SED单行脚本快速参考](http://sed.sourceforge.net/sed1line_zh-CN.html)

- [sed简明教程](https://coolshell.cn/articles/9104.html)

- [官方文档](https://www.gnu.org/software/sed/manual/sed.html)