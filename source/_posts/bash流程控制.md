---
title: bash流程控制
date: 2015-07-04 23:17:10
categories: shell
tags:
    - 脚本编程
    - bash
---

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/3137327.jpg)

<!-- more -->

## 条件判断 ##

条件判断是在程序执行时，根据不同的条件，选择执行不同的程序语句。通常包括单分支结构和多分支结构。单分支结构就是只判断一种情况，多分支结构就是判断多种情况。

### 单分支 ###

针对单分支的情况，我们可以使用两种方式进行条件判断。

**1. if 语句**

最常见的条件判断语句就是：`if...then`，当满足条件就执行操作命令，`fi` 表示结束。

语法：

```bash
if 表达式 ;then

操作

fi
```

比如我们要实现判断从控制台输入的数字是否大于 7 ，如果大于 7 就在控制台打印出 you're right! 。

```bash
#!/bin/bash

read -p "Input num :" a 

if [ $a -gt 7 ];then
   echo "you're right!"
fi
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/69103202.jpg)

**2. case 语句**

通常单分支的情况都是使用 if 语句，但我们也可以使用 case 语句来判断单分支的情况。

单分支的 case 语句语法结构如下：

```bash
case 变量值 in
值)
操作
esac
```

> 判断当变量值等于某值的时候，就执行包含在结构中的语句。 用 `esac` 结束。
比如我们要实现当从控制台输入的数字等于 7 时，输出 you're right ，除了使用 `if` 语句还可以使用 `case` 语句。

```bash

#!/bin/bash

read -p "Input num :" a 

case $a in 
7)
   echo "you're right!"
esac
```

运行结果如下：

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/45508066.jpg)

### 多分支 ###

上面的例子只是当我们输入的数字大于 7 才会输出一些内容，当我们想要在输入的数字等于 7 或者小于 7 时，也能给出一些提示信息的话，就需要对多种情况进行判断，此时我们就需要使用下面讲解的两种结构。

1. elif/else

多分支的 `if` 语句结构如下：

```bash
if 表达式1 ;then
操作1
elif 表达式2 ;then
操作2
elif 表达式3 ;then
操作3
...
else
操作4
fi
```



> 如果表达式1的结果为真，则执行操作1，如果表达式2的结果为真，则执行操作2，依次类推，如果所有条件都不符合，则执行 `else` 后面的操作4 。同样也是以 `fi` 结束。

我们想要实现在等于 7 和 小于 7 的时候也能打印出一些提示，就可以按照如下操作:

```bash

#!/bin/bash

read -p "Input num :" a 

if [ $a -gt 7 ];then
   echo "you're right!"
elif [ $a -eq 7 ];then
    echo "This number is equal to 7"
else
    echo "This number is less than 7"
fi
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/86126700.jpg)

**2. case**

条件分支中如果需要判断的条件过多，那么使用 `if/elif` 来处理就很显得很麻烦，因此在处理选择条件较多的情况时就可以利用 `case` 来完成，它类似于编程语言中的 `switch/case` ，但是在用法上有所不同。

语法：

```bash
case 变量值 in
值1）
操作1
;;  
值2）
操作2
;;
...
*）
操作3
;;
esac
```


> 如果变量值等于值1，则执行操作1，如果变量值为值2，则执行操作2，依次类推，如果所有的条件都不满足，则执行 *） 后的操作3 。



> - 每个判断条件需要使用 `;;`双分号来结束。
- 值不一定是数字，也可以为字符串。
- 最后需要用 `esac` 来结束本次条件判断。

如下实例实现判断输入月份属于哪个季度。

```bash

#!/bin/bash

read -p "Input the month :" a

case $a in
1|2|3)
        echo "This is the first quarter";;
4|5|6)
        echo "This is the second quarter";;
7|8|9)
    echo "This is the third quarter";;
10|11|12)
    echo "This is the fourth quarter";;
*)
        echo "Input error";;
esac
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/53488650.jpg)

## 循环结构 ##

在编写脚本时会遇到要重复执行一段代码很多次或上百上千次，这种时候设定成循环语句来实现就会简单很多了。

常用的循环结构有：

- 当型结构：for 循环、while 循环
- 直到型结构：until 循环


> 当型结构：先对控制条件进行判断，当条件满足时，再重复执行一些操作，直到条件不再满足。


> 直到型循环：先在执行了一次循环体之后，再对控制条件进行判断，当条件不满足时执行循环体，满足时则停止。

###  for 循环 ###

for循环流程图：

```flow
st=>star:开始
e=>end:结束
op=>operation:操作
cond=>condition: Yes or No?

st->cond
cond(yes)->op->cond
cond(no)->e
```

for 循环有三种结构：

- 列表 for 循环
- 不带列表的 for 循环
- 类似 C 语言风格的 for 循环

**1. 列表 for 循环**

语法：
```bash
for 循环变量 in 列表
do
    操作
done
```

实例1：

用 for 循环的方式输出 1 到 10 ：

新建一个 loop.sh
```bash
#!/bin/bash

for i in {1..10}
do
echo $i
done
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/62732378.jpg)

实例2：

我们使用 for 循环进行数值运算，计算出 1~10 的和:

```bash
#!/bin/bash

sum=0

for n in {1..10};do

  let "sum+=n" # 进行数值运算

done

echo "sum is : $sum"
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/14946772.jpg)

**2. 不带列表的 for 循环**

语法：

```bash
for 循环变量
do
操作
done
```

实例：

使用 for 循环打印出从控制台输入的参数。

```bash
#!/bin/bash

for a
do
echo $a
done
```

执行

```bash
$ bash nolist.sh 1 2 3
1
2
3
```

类似 C 语言风格的 for 循环

也被称为计数循环。

实例：

实现计算 1 到 10 的和。

```bash
#!/bin/bash

sum=0

for ((n=1;n<=10;n++));
do

  let "sum+=n"

done

echo "sum is : $sum"
```

> sum 代表和，先给 sum 赋了一个初值 0 。

> 在 for 后面的双括号中，给循环变量 n 赋了一个初值 1，当 n 小于或者等于 10 的时候，就把 sum 和 n 的和赋值给变量 sum ，然后 n 再加一。刚开始的时候，`n=1`，`sum=0`，这个时候 n 的值是小于 10 的，所以进入循环体，执行 `sum+=n` ，这个时候 sum 的值就变成了 1 ，然后执行 `n++` ，这个时候 n 的值变为了 2，此时 n 的值还是小于 10 的，所以再次进入循环体，依次类推。直到 n 的值大于 10 时，就不再进入循环体。


### while 循环 ###

while 循环利用一个条件来控制是否继续重复执行某个操作。为了避免死循环，必须保证循环体中包含退出循环的条件。

语法：
```bash
while 判断表达式
do
    操作
done
```

实例：

使用 while 循环计算出 1~10 的和。
```bash
#!/bin/bash

sum=0
num=1
while (($num <= 10))
do
    let "sum+=num"
    let "num++"
done
echo "the sum is $sum"
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/99780367.jpg)

### until 循环 ###

`until` 与 `while` 的不同是，`until` 在条件表达式不成立时，进入循环，条件成立，终止循环，与 `while` 刚好相反。

语法：
```bash
until 判断表达式
do
操作
done
```

实例：

使用 until 循环计算出 `1~10` 的和。

```bash
#!/bin/bash

sum=0
num=1
until (( $num > 10 ))
do
    let "sum+=num"
    let "num++"
done
echo "the sum is $sum"
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/97322352.jpg)


### select 循环 ###

除了前面三种比较常见的循环控制以外，还有一种叫 select 循环的控制语句。它主要是提供一种创建具有编号的菜单的方法。就像我们平时出去吃饭一样，店家制定一个菜单供我们点菜。

语法：
```bash
select 变量名 in [ 菜单取值列表 ]
do
    操作
done
```

实例1

通过 select 简单创建一个菜单选项，实现打印出我们选择的字符串。

新建一个 select.sh 脚本文件：

```bash
#!/bin/bash

select str in hello nihao Bonjour 
do
    echo $str
done
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/50556256.jpg)

实例2：

打印出我们选择的在 /home/shiyanlou 下的文件名或目录名。

新建一个 select2.sh

```bash
#!/bin/bash

PS3="select a num:"
select files in `ls /root/Desktop`
do
    echo -e "you seleted:\n  $dir"
done
```
> `PS3` 是 select 循环的提示符，就是上个实例提到的默认提示符（`#？`），这里我们对其进行了修改


![](http://p7wcdketk.bkt.clouddn.com/18-5-2/44378860.jpg)

## 循环控制 ##

像其他编程语言一样，shell 有时也需要在没有到达循环条件结束时就强制跳出循环。常见的强制跳出循环的操作有：break、continue 等。

break

break 强制跳出所有循环，终止整个循环的执行。它可以跳出 for、while、until 等循环。
```
# 退出循环
break  

# 在嵌套循环中，退出第 n 层循环
break n 
```

举例:

```bash
#!/bin/sh

num=0

while [ $num -lt 10 ]
do
   echo $num

   if [ $num -eq 7 ]
   then
      break
   fi
   let num++
done
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/48842063.jpg)

可以看到 break 执行后，7 以后的数字就没有再输出了，也就是说跳出了循环。

### continue ###

continue 也是用于强制跳出循环，不过它主要是跳出当前循环，不会跳出所有循环，也就是后面的循环在正常情况下依旧能执行。

举例

```bash
#!/bin/sh

for num in {1..10}
do

   if [ $num -eq 7 ]
   then
      continue
   fi

   echo $num
done
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/99568551.jpg)

运行到 continue 的时候就跳出了当前的循环，因此在输出时就没有输出数字 7

