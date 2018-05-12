---
title: awk的流程控制
date: 2016-08-11 8:49:33
categories: shell
tags:
    - awk
    - 脚本编程
---

# awk中的流程控制 #

## 1、条件语句 ##

### 语法 ###

awk 中的流程控制语句与其他高级编程语言（C、Java 等）相差不大，其格式为：

```bash
if(expression)
  action1
[else action2]
如果条件表达式 expression 的值为真，就执行 action1 。当存在 else 语句时，如果条件表达式为假，就执行 action2 。
```

如果操作是多个语句组成，就要用大括号括起来：

```bash
if(expression){
  statement1
  statement2
    ...
}
else if(expression){
  statement1
  statement2
    ...
}
else{
  statement1
  statement2
    ...
}
```

### 例子 ###

新建一个测试文本 testavg，输入如下内容：

```
john 85 92 78 94 88
andrea 89 90 75 90 86
jasper 84 88 80 92 84
tom 60 55 70 65 60
bob 99 90 87 93 96
jim 76 75 83 65 66
```

我们将通过脚本来计算学生平均成绩：

新建一个脚本文件 avg ，输入如下内容：

```bash
{
    total=$2+$3+$4+$5+$6
    avg=total/5
    print $1,avg
}
```

运行

```
$ awk -f avg testavg
```
![](http://p7wcdketk.bkt.clouddn.com/18-5-10/55876956.jpg)

如果平均分为大于等于**65**就为及格的话，显示及格不及格情况。我们来修改 avg 如下:

```bash
{
    total=$2+$3+$4+$5+$6
    avg=total/5
    if(avg >= 65)
        grade="Pass"
    else
        grade="Fail"
    print $1,avg,grade
}
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-10/34380376.jpg)

如果对成绩划分等级呢，大于等于 90 的为 A ，大于等于 80 的为 B ，大于等于 70 的为 C ，大于等于 60 的为 D ，其他的就是不及格 F 。

修改 avg 为如下：

```bash
{
    total=$2+$3+$4+$5+$6
    avg=total/5
    if(avg >= 90) grade="A"
    else if (avg >= 80) grade="B"
    else if (avg >= 70) grade="C"
    else if (avg >= 60) grade="D"
    else grade="F"
    print $1,avg,grade
}
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-10/3001838.jpg)

## 2、循环 ##

### while循环 ###

```bash
while(表达式){
  操作
}
#如果括号里的表达式为真，就执行花括号里面的操作，一直循环，直到表达式为假。
```

#### 例子 ####

如下计算1到100的和
```bash
$ awk '                                                            
> BEGIN{
> i=1;
> test=100;
> total=0;
> while(i<=test){
> total+=i;
> i++;
> }print total;}'
```


### do-while 循环 ###

#### 语法 ####
```bash
do{
  操作
}
while(表达式)
#与 while 循环的不同之处：循环体至少执行一次无论表达式是否成立。
```

#### 例子 ####
```bash
BEGIN{
    do{
        print x=x+1

    }while(x<0)
}
```

执行之后我们会发现结果为 1，这就是因为：

awk 中的变量被初始化为0

在判断表达式之前会执行其中的语句块 print x=x+1

此时打印出 x 的值之后在做判断，发现不成立，所以不再执行
这就是 do/while 循环与 while 的区别。

### for 循环 ###

#### 语法 ####
```bash
for(set_counter;test_counter;increment_counter){
  操作
}
```

- set_counter：设置计数器变量的初值


- test_counter：描述在循环开始时要测试的条件


- increment_counter：每次在循环的数值变化器，且在重新测试测试条件之前

#### 例子 ####

下面我们来用 for 循环的方式来求上面的分数平均值，修改 avg 为如下内容：

```bash
{
    total=0
    for(i=2;i<=NF;i++)
        total+=$i
    avg=total/(NF-1)
    print $1,avg
}
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-10/97643659.jpg)

### 综合实例：求阶乘 ###

阶乘：`n!=nx(n-1)x(n-2)x...x1`

新建文件jc，输入：

```bash
awk '
BEGIN{
    printf("Enter number:")
}

$1 ~ /^[0-9]+$/{     //获取输入，匹配是不是为数字
    num=$1         //num是我们要计算阶乘的数
    if(num==0)
        fact=1    
    else
        fact=num
    for(x=num-1;x>1;x--)
        fact*=x     //计算阶乘
    printf("The factorial of %d is %g\n",num,fact)
    exit
}
{printf("\nInvalid entry.Enter a number:")}'

```
> **重点关注获取输入的用法，和判断是否为数字的正则表达式语句**

![](http://p7wcdketk.bkt.clouddn.com/18-5-10/14615810.jpg)

## 3、数组 ##

数组可以用来存储一组数据的变量。数组中的每一个元素通过它们在数组中的下标访问。每个下标都用方括号括起来。

### 语法 ###

#### 1. 建立数组 ####

语法：
```bash
array[index]=value
```

#### 2.读取数组值 ####

语法：

```bash
{for(item in array) print array[item]}   #输出的顺序是随机的

{for(i=1;i<=len;i++) print array[i]}   #Len是数组的长度
```


#### 3. 多维数组 ####

awk 支持线性数组，在这种数组中的每个元素的下标是单个下标。如果将线性数组看成是一行数组，那么两位数组将表示数据的行和列，这样的数组就是多维数组中的两维数组，依此类推。

例如二维数组语法：
```bash
file_array[NR,i]=$i
```


> file_array 就是数组名。



> NR 是记录数，也就是行， i 是字段数，也就是列。下标是两个。



> 下标分隔符，默认是 `\034` ，可以用 `SUBSEP` 设置分隔符。
```bash
$ awk 'BEGIN{array["a","b"]=1;for(i in array) print i}'           
$ awk 'BEGIN{SUBSEP=":";array["a","b"]=1;for(i in array) print i}' 
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-10/57480508.jpg)

怎么向多维数组写入或读出元素?我们通过这样一个例子来了解：

我们新建一个宽高都为 6 的二维数组，每个位置上先用 O 填充，遍历 testdw 中的坐标，把相应坐标上的元素变为 X。

新建一个 testdw 文件，输入如下内容：
```
1,1
2,2
3,3
4,4
5,5
6,6
1,6
2,5
3,4
4,3
5,2
6,1

```

新建一个脚本文件 dw.awk ，输入如下内容：

```bash
BEGIN{
FS=","
WIDTH=6
HEIGHT=6
for(i=1;i<=WIDTH;++i){
    for(j=1;j<=HEIGHT;++j){
        dw[i,j]="O"
    }
}
}
{
    dw[$1,$2]="X"
}
END{
for(i=1;i<=WIDTH;++i){
    for(j=1;j<=HEIGHT;++j)
        printf("%s",dw[i,j])
    printf("\n")
}
}
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-11/71035683.jpg)

### 删除数组 ###

#### 语法： ####

```
delete array  #删除整个数组
delete array[item]  #删除某个数组元素（item）
```

### 应用实例 ###

#### 1. 遍历 ####

 
**（1）for循环**

新建一个 youxu1，输入如下内容：

```bash
#!/bin/awk -f

BEGIN{
    a[1]="a"
    a[2]="b"
    a[3]="c"
    a[4]="d"
    a[5]="e"

    for(i=1;i<=length(a);i++){
        print i,a[i]
    }
}
```

> 用 length(a) 函数得到数组 `a` 的长度



> 数组的下标是从`1`开始计算。注意和其他语言区分。



**（2）使用 in**

新建一个 youxu2，输入如下内容：

```bash
#!/bin/awk -f

BEGIN{
    a[1]="a"
    a[2]="b"
    a[3]="c"
    a[4]="d"
    a[5]="e"

    for(i in a){
        print i,a[i]
    }
}
```

执行

```bash
$ awk -f youxu2
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-11/24353822.jpg)

#### 2. 顺序排列 ####

我们先来新建一个测试文本 `testbasic`:

测试文本的每行都由一个行号开始，行号不是按照顺序排列的。

```bash
$ vim testbasic
2 hello world
1 hello wyman
3 hello you
```

下面的程序实现把测试文件内容按行号顺序打印出来。此程序通过使用行号作为下标创建数组来排序行，然后打印按数字顺序排序的行。

新建一个脚本文件 basic：
```bash
$ vim basic
{
    //用行号作为数组arr的下标，每行的内容作为对应的值
    if ($1 > max) 
        max = $1
    arr[$1] = $0
}

END {
      //用for 循环遍历，打印出数组元素
    for (x = 1; x <= max; x++)
        print arr[x]
}
```


![](http://p7wcdketk.bkt.clouddn.com/18-5-11/37074759.jpg)

#### 3.成绩例子 ####

记得上面的成绩例子吗，我们按照平均成绩分了等级。现在我们要知道获得 A 的学生是多少，获得 B 的学生是多少怎么做。

我们可以为每个字母等级设置不同的变量并判断哪个计数器可以递增。我们可以定义一个数组 `class_grade`，用字母等级作为数组的下标。如果遇到该等级，就给对应的值加`1` 。在 END 中用 for 循环遍历 `letter_grade` 制定数组 `class_grade` 的一个下标。输出被传送 `sort` 中，用于按正确的顺序输出等级。

我们修改 avg 内容为如下：

```bash
BEGIN{OFS="\t"}
{
total=0
for(i=2;i<=NF;++i)
    total+=$i
avg=total/(NF-1)
student_avg[NR]=avg
if(avg >= 90) grade="A"
else if (avg >= 80) grade="B"
else if (avg >= 70) grade="C"
else if (avg >= 60) grade="D"
else grade="F"
++class_grade[grade]
print $1,avg,grade
}
END{
for(x=1;x<=NR;x++)
    class_avg_total+=student_avg[x]
class_average=class_avg_total/NR
for(x=1;x<=NR;x++)
    if(student_avg[x]>=class_average)
        ++above_average
    else
        ++below_average
print ""
print "class average: ",class_average
print "at or above average: ",above_average
print "below average: ",below_average
for(letter_grade in class_grade)
    print letter_grade ":",class_grade[letter_grade]|"sort"
}
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-11/87545569.jpg)




