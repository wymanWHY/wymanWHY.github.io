---
title: sed中级使用教程
date: 2016-04-06 18:42:07
categories: shell
tags:
    - sed
    - shell
    - bash
---


在上一章节中我们简单的使用了 sed，帮助我们对文本文件做添加、删除、修改等操作。但都是单一命令的简单操作，若我们需要多重操作同时存在的时候，我们便需要利用 sed 的其他特性了。

---
## 多重命令 ##

顾名思义，多重命令就是多个命令，我们将使用下面的几种方式实现为大家呈现同时多个命令进行处理。

新建一个test2.txt文本作为实验：

```bash
hello
This is a brand new test file for sed
Please be gentle!
```

### 使用分号分隔命令 ###

我们想同时将 hello 修改成 Hi，将 gentle 修改成 rude。使用`;`**分号**分隔命令的语句如下：

`$ sed 's/hello/Hi/; s/gentle/rude/' text2.txt`

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/98854982.jpg)

如果不用分号，使用`-e`参数也是可以的：

`$ sed -e 's/hello/Hi/' -e 's/gentle/rude/' text2.txt`

> 要注意格式，别漏了单引号`'`和最后的斜杠`/`
> 每次操作对应每一个`-e`

### 分行操作 ###

除了使用分号，以及 `-e` 参数之外，还可以通过分行来编写多个命令，来达到多重命令的效果，如下所示：

```bash
$ sed '                  #按enter键                                         
quote> s/hello/Hi/
quote> s/gentle/rude/' text2.txt
```

## 脚本文件 ##

在命令行输入较长的命令是不切实际的，我们可以通过在脚本文件中写入多个 sed 命令。最后再使用 sed 的 -f 参数去执行脚本文件中的命令，对目标文件进行处理。这其实也属于多重命令的一种应用方式。

### 语法 ###

```bash
$ sed -f <scriptfile> <file>
scriptfile ：脚本文件的名称。
file ：待处理文件的名称。
```

还是上面的例子，我们可以把sed语句写入文件sedfile中：

```bash
s/hello/Hi/
s/gentle/rude/
```

执行命令
```
sed -f sedfile text2.txt
```
,效果与之前一致：

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/31945997.jpg)

---

## 寻址 ##

sed 修改文件，有时候我们并不会对全文的内容进行操作，只对符合条件的行或者内容进行操作，这时候我们就会用到寻址了。而这里所谓的条件有两种：

- 指定的某一行
- 利用正则表达式，符合规则的内容

### 使用寻址 ###

对于修改文件的内容，可以只对符合条件的行或者内容进行操作，也可以针对全文，大致的分类如下：

- 没有指定地址：应用于每一行
- 只有一个地址：应用于这个地址匹配的任意行
- 指定了由逗号分隔的两个地址：匹配第一个地址的第一行于匹配第二个地址之间的行
- 地址后跟有感叹号：应用于不匹配该地址的所有的行

新建文件 xunzhi ，输入如下内容：

```bash
a b
b
c

d b

e

f
```

把文件中的 b 字符替换成 new，使用如下操作：

`$ nl xunzhi | sed 's/b/new/'`

在下面的运行截图中可以看到所有的 b 都被替换成 new，这就是没有指定地址的方式。

如果我们想要将字符 a 所在行的字符 b 替换为 new，就需要使用寻址：

```
$ nl xunzhi | sed '/a/s/b/new/'
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/36620495.jpg)

若是我们想指定删除最后一行：

```
$ nl xunzhi | sed '$d'
```

> 注意：d 是 sed 中的删除命令

利用正则表达式删除空行：

```
$ cat xunzhi | sed '/^$/d'
```

指定某个范围内的行，例如删除从第3行到最后一行的所有行：

```
$ nl xunzhi | sed '3,$d'
```
> 注：`$`标识为最后一行

当然指定区间范围时，具体行数于正则也可以混合使用，例如删除第一行直到第一个空行的所有行：

```
$ cat xunzhi | sed '1,/^$/d'
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/48378091.jpg)

### 分组命令 ###

在上面的操作中，我们只是对字符 a 所在的行执行一个操作，当我们想对 a 所在的行执行多个操作的时候就需要使用到分组命令。

sed 使用大括号`{}` 将一个地址嵌套在另一个地址中，或者在相同的地址上应用多个命令。我们可以根据如下实践来更好的理解。

修改我们的 xunzhi 文件内容，如下所示：

```python
a b hello world
b world
c b d hello world

d b world good

e hello

f
```



**需求：要把包含字符 b 又包含 hello 的行中的 world 改成 wyman，再把第 5 行的 world 替换成 very。**

**解答：**我们发现第 5 行包含 b 和 d。我们**把 b 作为一个共同的地址**。找到有 b 的行，在有 b 的行中找到有 hello 的行，然后替换 world 为 wyman，再在有 b 的行中找到有 d 的行，然后替换包含 world 的为 very。**注意这个顺序，这里有两行都同时包含 b, d, world 三个字符串，如果先去替换成 very 的话，那么第三行的 world 也会被替换成 very，而后执行替换成 wyman 的时候，第三行就没有 world 那个字符串了。**

新建一个文件 fenzu，输入如下内容:

```python
# this is fenzu
/b/{
/hello/s/world/wyman/
/d /s/world/very/
}
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/57281771.jpg)

** 注意:** `大括号`后面不能有空格，并且右括号要单独在一行。 注意 `d` 后面有空格，**如果没有空格的话，所有含有 `b` 字符的行的字符 world 都会被替换成 very**。因为 sed 会把 `d` 认成一个特殊字符。采用转义符去转义 `d` 是不行的。比如，修改xunzhi2为：

```python

a b hello world
b world
c b d hello world

d b world good

e hello

f

b world
```

将刚才的`d`后的空格删除：

```python
# this is fenzu
/b/{
/hello/s/world/wyman/
/d/s/world/very/
}
```

执行后，会发现成了这样：

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/73630508.jpg)

而如果是存在空格的情况：

![](http://p7wcdketk.bkt.clouddn.com/18-5-3/98409241.jpg)

**注意两者的差别**


---

参考文档：

- [SED单行脚本快速参考](http://sed.sourceforge.net/sed1line_zh-CN.html)

- [sed简明教程](https://coolshell.cn/articles/9104.html)

- [官方文档](https://www.gnu.org/software/sed/manual/sed.html)
