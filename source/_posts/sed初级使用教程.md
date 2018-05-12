---
title: sed初级使用教程
date: 2016-04-03 10:12:27
categories: shell
tags:
    - sed
    - shell
    - bash
---


SED的英文全称是 Stream EDitor，是一个简单而强大的流编辑器，用程序的方式来编辑文本，sed中往往使用了大量的正则表达式，所以它的执行效率非常高，几行代码即可完成复杂的任务，如替换、增加内容、删除文本内容等等。当然，在弄懂正则表达式之前，sed中火星文一般的语法看着也非常容易头大。

## sed 命令基本格式： ##

`sed [参数]... [执行命令] [输入文件]...`



### 常用的参数： ###

|参数|	说明|
|:-|:-|
|-n|	安静模式，只打印受影响的行，默认打印输入数据的全部内容|
|-e|	用于在脚本中添加多个执行命令一次执行，在命令行中执行多个命令通常不需要加该参数|
|-f filename|	指定执行 filename 文件中的命令|
|-r|	使用扩展正则表达式，默认为标准正则表达式|
|-i|	将直接修改输入文件内容，而不是打印到标准输出设备|

### 执行命令的格式： ###
```bash
[n1][,n2]command

[n1][~step]command
```

其中 n1,n2 表示输入内容的行号，它们之间为 ,，如果为～波浪号则表示从 n1 开始以 step 为步进的所有行；command为执行动作，下面为一些常用动作指令：

|命令|	说明|
|:-|:-|
|s	|行内替换|
|c	|整行替换|
|a	|插入到指定行的后面|
|i	|插入到指定行的前面|
|p	|打印指定行，通常与 `-n` 参数配合使用|
|d	|删除指定行|

### sed 使用实例 ###

#### 删除指定行 ####

为了让效果明显的看到，我们使用 `nl` 命令将 test.txt 的内容与打印行号同时打印出来，同时利用 sed 将 2-5 行删除显示：

    $ nl text.txt | sed '2,5d'

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/46174925.jpg)

> 命令解释：'2,5d' 表示 2~5 行，d 表示删除。

删除第三行到最后一行：

$ nl text.txt |sed '3,$d'

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/67388428.jpg)
> 命令解释：`$`定位到最后一行

因为没有使用`-i`参数，所以原text.txt文件内容并未被覆盖，若是要在原文件中删除第 1 行，可以这样：

    $ sed -i '1d' text.txt

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/27361420.jpg)
> 可以发现，text.txt文件被改写了


#### 添加字符串 ####

在第二行后添加 test 字符串，`a`(append)表示在行后添加一行加上字符串，`i`(insert) 表示在行前加一行添加字符串

例如，在第2行后添加字符串test：

    $ nl text.txt | sed '2a test'

在第2行前添加字符串test：

    $ nl text.txt | sed '2i test'

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/58744424.jpg)


#### 替换字符串 ####

将 2-5 行内容取代为 gogogo

c 为替换内容选项。

    $ nl text.txt | sed '2,5c gogogo'

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/50473770.jpg)

#### 列出匹配字符 ####

列出 text.txt 内第 5-7 行

sed 命令中 `-n` 为安静模式选项。以下两条命令执行结束后可对比结果。

```bash
$ nl text.txt |sed -n '5,7p'

$ nl text.txt |sed  '5,7p'

```

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/81111235.jpg)



---

参考文档：

- [SED单行脚本快速参考](http://sed.sourceforge.net/sed1line_zh-CN.html)

- [sed简明教程](https://coolshell.cn/articles/9104.html)

- [官方文档](https://www.gnu.org/software/sed/manual/sed.html)
