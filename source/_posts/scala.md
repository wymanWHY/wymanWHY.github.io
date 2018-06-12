---
title: scala
date: 2018-06-01 22:11:43
categories: 大数据
tags:
    - scala

---


> Scala 是一门多范式（multi-paradigm）的编程语言，设计初衷是要集成面向对象编程和函数式编程的各种特性。



> Scala 运行在Java虚拟机上，并兼容现有的Java程序


## 值与变量 ##

值(val)赋值后不可变

```scala
val 值名称：类型=XXX
```

变量（var）赋值后可变

```scala
var 变量名称：类型=xxx
```

### 常用变量 ###

```scala
Byte 
Char 
Short 
Int 
Long 
Float 
Double 
Boolean
```

## 方法 ##

### 定义 ###

```scala
def 方法名 （参数名：参数类型）：返回类型={
	//block内最后一行为返回值
}
```

当定义的方法没有参数时，调用这个方法时，可以直接用`方法名()`的方式调用，也可以不用加括号`()`，直接`方法名`，例如:


```scala
scala> def say(){println("ok")}
say: ()Unit

scala> say
ok

scala> say()
ok
```

但是，当定义时本身就没有使用括号的情况下，调用时不能再在方法名的后面加上括号，例如：

```scala
scala>  def say{println{"ok"}}
say: Unit

scala> say
ok

scala> say()
<console>:13: error: Unit does not take parameters
       say()
          ^
```

当返回值为Unit时可以定义为：

```scala
def 方法名（参数名：参数类型）{
}
```

### Tips ###

- 没有参数的方法可以不带括号访问

- scala没有静态方法，可以通过object来实现

- 互为伴生类/伴生对象的class、object，必须出现在同一源码文件内

## 条件表达式和循环 ##

### 条件表达式 ###

`if`:

```scala
scala> val i=1   //定义一个值，i=1
i: Int = 1
 
scala> if (i<0) false else true  //如果i<0,输出false；否则，输出true
res9: Boolean = true     //输出true
```



### 循环表达式 ###

`for,while,to,until,Range`:

`**to**`:

```scala
scala> 1 to 10   //输出10个数，从1到10，包含1和10，类似于[1,10]闭区间
res10: scala.collection.immutable.Range.Inclusive = Range(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
```

`**until**`:

```scala
scala> 1 until 10  //输出1到10-1个数，类似与[1,10)半开半闭区间
res11: scala.collection.immutable.Range = Range(1, 2, 3, 4, 5, 6, 7, 8, 9)
```

`**Range**`:

```scala
scala> Range(1,10)
res12: scala.collection.immutable.Range = Range(1, 2, 3, 4, 5, 6, 7, 8, 9)
```

```scala
scala> Range(1,10,2)    //设置步长为2
res13: scala.collection.immutable.Range = Range(1, 3, 5, 7, 9)
```

`**for**`:

```scala
scala> for(i<- 1 to 10) {println(i)}
1
2
3
4
5
6
7
8
9
10
```

## Lazy value ##

### 语法 ###

```java
lazy val val_name = val_value
```

lazy变量只有到要用到的时候，才会被初始化


## 参数 ##

### 默认参数 ###

`def say(name:String="null"){println{name}}`

```java
scala> def say(name:String="null"){println{name}}
say: (name: String)Unit

scala> say()   //当调用时没有参数传入时，会使用定义时设定的默认参数null
null

scala> say("yeah")  //如果调用带有参数，则使用该参数
yeah
```

### 带名参数 ###

带名参数允许赋值时的顺序可以和定义时的顺序不一致：

```java
scala> def add(x:Int,y:Int){println(x)}   //定义时，x在前，y在后，要求输出x
add: (x: Int, y: Int)Unit

scala> add(y=1,x=2)                       //调用时，y在前，x在后，依旧输出x
2 

```

> 并不以参数的位置决定



## 数组 ##

### 定长数组 ###

语法1：`val array_name = new Array[T](length)`

```java
scala> val a=new Array[String](3)   //定义一个数组，含3个String类型的元素
a: Array[String] = Array(null, null, null)  //元素的初始值为null

scala> for(i<-0 to 2){println(a(i))}    //遍历验证，确实为null，要注意下标从0开始
null
null
null

scala> a(0)=baidu;a(1)=ali;a(2)=tencent  //因为是string类型，赋值时需要用双引号标注，否则就会这样报错
<console>:14: error: not found: value tencent
       a(2)=tencent
            ^
<console>:12: error: not found: value baidu
       a(0)=baidu;a(1)=ali;;
            ^
<console>:12: error: not found: value ali
       a(0)=baidu;a(1)=ali;;
                       ^

scala> a(0)=baidu
<console>:13: error: not found: value baidu
       a(0)=baidu
            ^

scala> a(0)="baidu";a(1)="ali";a(2)="tencent"

scala> for(i<-0 to 2){println(a(i))}   //遍历验证，赋值有效
baidu
ali
tencent
```

语法2：`val array_name = Array(“”,””)`

```java
scala> val b=Array("baidu","ali","tencent")
b: Array[String] = Array(baidu, ali, tencent)

scala> for(j<- 0 to 2){println(b(j))}
baidu
ali
tencent
```


### 变长数组 ###

顾名思义，数组可以插入元素

