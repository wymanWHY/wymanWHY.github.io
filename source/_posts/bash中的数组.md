---
title: bash中的数组
date: 2015-07-07 13:29:57
categories: shell
tags:
    - 脚本编程
    - bash
---


![](http://p7wcdketk.bkt.clouddn.com/18-5-2/3137327.jpg)

<!-- more -->

## 数组 ##

### 数组的定义和赋值 ###

bash 提供对于一维数组的支持，需要注意的是，它并不支持多维数组。通常情况下，数组的索引为一个整数，从 0 开始计算。但是我们也可以使用字符串作为数组的索引，这样的数组被称为关联数组。

在 bash 中，变量其实可以理解为只有一个元素的索引数组。如下示例：

```bash
# 定义一个变量 var1，此时 var1 为只有一个元素的数组
var1="hello-bash"

# 查看 var1 的值
echo $var1

# 查看 var1 数组中索引为 0 的值，即第一个元素，关于引用数组中元素的内容我们在下面的内容将会学习到
echo ${var1[0]}

# 查看 var1 数组中索引为 1 的值，结果为空
echo ${var1[1]}

```


![](http://p7wcdketk.bkt.clouddn.com/18-5-2/79550127.jpg)

上述定义的变量 var1 可以理解为一个只有一个元素的数组。

对于索引数组的定义，我们只需给对应的索引分配一个值，即会自动创建一个索引数组，如下所示：

```bash
# 给索引 0 分配一个值，即会自动创建索引数组 var2
var2[0]=hello-bash001
var2[1]=hello-bash002
# 查看数组 var2 索引 `0` 对应的值，使用下述两种方式
echo $var2
echo ${var2[0]}
# 查看索引为 `1` 的值
echo ${var2[1]}

```
![](http://p7wcdketk.bkt.clouddn.com/18-5-2/26776218.jpg)

除了上述介绍的定义索引数组的方式，我们还可以使用 var=(value1 value2 value3) 的方式

进行赋值，并且可以再赋值的时候指定索引。

如下示例：

```bash

# 定义数组 var3
var3=(a b c)

# 查看数组中所有的元素
echo ${var3[*]}

# 查看数组中指定索引的值
echo ${var3[0]}
echo ${var3[1]}

# 定义数组 var4，使用指定索引定义
var4=([0]=a b [1]=c [9]=d)

# 查看相应的值
echo ${var4[*]}
echo ${var4[1]}
echo ${var4[9]}

```

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/40944589.jpg)
> 在给索引数组分配值时，如果指定了索引，则将该索引分配给指定的值。

### 使用 read 定义 ###

使用 `read` 命令接受输入，并保存为一个数组，需要使用到 `-a` 参数，即array。

如下示例：

```bash

$ read -a var5

# 查看数组 var5 的所有值
echo ${var5[*]}

```


![](http://p7wcdketk.bkt.clouddn.com/18-5-2/9180583.jpg)


### 使用 declare 定义 ###

`declare` 命令除了可以定义索引数组之外（使用 `-a` 参数），还可以定义关联数组（使用 `-A`参数）。

如下示例，我们使用 `declare` 声明一个索引数组：

```bash

$ declare -a var6=(a b c d)

# 查看 var6 的全部元素
$ echo ${var6[*]}

```

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/1686537.jpg)

### 引用 ###

对于数组的引用来说，我们有以下几种方式：

| 方式 | 含义 |
| :- | :- |
| "${array_name[n]}" | 查看指定索引的元素，n 为数组的索引 |
| "${array_name[*]}" | 所有的数组元素的值 |
| "${array_name[@]}" | 所有的数组元素的值 |
| "${!array_name[@]}" | 所有的索引 |
| "${!array_name[*]}" |所有的索引 |
| "${array_name[*]:m}"  | 从索引m 开始，后面的所有元素 |
| "${array_name[*]:m:n}" | 从索引 m 开始，后面的 n 个元素 |
| "${""#array_name[*]}" |	显示数组元素个数 |


### 数组删除 ###

删除数组也是使用 `unset` 命令，但是我们可以仅删除数组中的某个元素，也可以删除整个数组

。

如下示例:

```bash

# 定义数组
$ array=(1 2 3)

# 打印数组中的所有元素
$ echo ${array[*]}
1 2 3
#
$ unset array[1]    # 删除索引为1的元素，注意是从0开始
$ echo $array[*]
1 3
$ unset array    # 删除整个数组
$ echo ${array[*]}

```