---
title: bash中的函数
date: 2015-07-12 18:08:02
categories: shell
tags:
    - 脚本编程
    - bash
---


![](http://p7wcdketk.bkt.clouddn.com/18-5-2/3137327.jpg)

<!-- more -->

## 函数 ##

### 定义 ###

bash 也有自定义函数的功能，当脚本变得很大时，可将脚本文件中常用的功能写成函数，这样可以提高程序的复用性，更易维护。定义函数语法如下：

```bash
函数名(){
  函数体
}

或者

function 函数名(){
  函数体
}

或者

function 函数名 {
  函数体
}
```


对于函数定义有一些注意项，简单列举如下

- 函数定义要在执行之前
- bash 执行顺序是：系统别名-》函数-》系统命令-》可执行文件
- 如果将函数放在独立的文件中，脚本使用的时候，需要使用 `source` 或 `. `来加载

### 位置参数 ###

函数的调用方式为 : `函数名 参数列表`

对于函数的参数与参数之间，函数名和参数之间都使用空格进行分隔。因此，在函数之中，我们引用函数的参数也是使用位置参数的方式进行使用。在函数中使用位置参数指代的是函数的参数，而不是脚本执行时使用的位置参数。

如下示例， test3.sh 脚本文件中的内容：

```bash
#!/bin/bash

# 定义函数
func1(){
    # 打印函数的参数
    echo $*
}

# 调用函数，参数为 a b c
func1 a b c

# 打印脚本的参数
echo $*
```


### 作用域 ###

对于 bash 中函数中变量的作用域与其它高级编程语言不同。

如果我们在函数中有一行定义变量的语句，在我们调用函数后，该定义被执行，这时定义的变量并不属于函数内部的局部变量，在函数外部也可以使用。而定义属于函数内部的变量我们需要使用 `local` 命令。

如下示例 test4.sh 文件中的内容：

```bash
#!/bin/bash

func(){
    # 此时定义一个变量
    var1=shiyanlou001

    # 使用 local 定义一个局部变量
    local var2=shiyanlou002
}

# 执行函数
func

# 执行后，我们可以使用 var1 的值，但是不能使用 var2 的值
echo $var1 $var2
```



### 返回值 ###

每一个脚本或者命令执行完成之后都会有一个退出的状态，系统中一般只会记录上一个指令执行之后的退出状态，而同执行脚本一样，函数执行完成后也会有一个返回的状态值，默认为函数体中最后一行命令执行的状态。

除此之外，也可以显示的使用 `return` 语句给函数返回一个状态值。`return` 语句的使用方式如下：
    return [n]

该语句会导致函数停止执行，并且将 n 值返回给调用者，如果未提供 n，则返回值是在该函数中执行的最后一个命令的退出状态。

如下示例 test5.sh 文件中的内容：

```bash

#!/bin/bash

func(){
    echo "start function"
    return 123
    echo "function end"
}

func

# 查看函数的返回值，使用 $?
echo "return: $?"
```
![](http://p7wcdketk.bkt.clouddn.com/18-5-2/16749451.jpg)

### 几个例子： ###

**1.求输入的数值中最大的数**

新建 maxvalue.sh

```bash
#!/bin/bash

max(){
    while test $1 #这里$1代表传进函数的第一个参数
    do
        if test $maxvalue;then
            if test $1 -gt $maxvalue;then
                maxvalue=$1
            fi
        else
            maxvalue=$1
        fi
        shift     # 函数参数左移一位的意思
    done
    return $maxvalue  #返回值
}

max $*    #调用函数
echo "max value is $maxvalue"
```


![](http://p7wcdketk.bkt.clouddn.com/18-5-2/92849614.jpg)


**2. 调用其它脚本**

新建一个 beidy.sh，被调用的脚本

```bash
#!/bin/bash

echo "who you are: $USER"
```

新建一个 dy.sh。调用的脚本

```bash
#!/bin/bash

echo "you locate $0"

./beidy.sh
echo "first arg is $1"
```

执行

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/25462283.jpg)

**3. 测试uRL**

对一组 url 测试能否访问成功

```bash
#!/bin/bash


url_list=(  #定义一个包含三个网址的数组 url_list
http://www.qq.com
http://www.taobao.com
http://www.google.com
)

wait(){   #定义倒计时函数 wait
    echo -n 'wait 3 second...'
    for ((i=0;i<3;i++))
    do
        echo -n ".";sleep 1  #每隔1秒打印一个点
    done
    echo
}

check_url(){
    wait   # 调用已经定义的 wait 函数

    #循环遍历 url_list 中的地址
    for ((i=0;i<`echo ${#url_list[*]}`;i++))
    do

        #检测是否可以访问数组元素中的地址
        wget -o /dev/null -T 3 --tries=1 --spider ${url_list[$i]} >/dev/null 2>&1  #--tries是设置尝试次数，--spider检查网址，后面的>/dev/null 2>&1是不保留任何输出

        if [ $? -eq 0 ];then   #如果返回值为0则表示访问成功
            echo "${url_list[$i]} success"
        else
            echo "${url_list[$i]} false"
        fi
    done
}

main(){  #定义主函数，即入口函数，应用程序运行时首先执行的代码
        check_url   #调用定义的 check_url 函数
}


main    #调用主函数 

```
   
执行

![](http://p7wcdketk.bkt.clouddn.com/18-5-2/27145150.jpg)