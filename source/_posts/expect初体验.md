---
title: expect初体验
date: 2018-04-11 09:07:48
categories: 
   - Linux OPS
   - shell
tags:
   - expect
   - 自动交互
   - bash
---

### 前言 ###
使用命令行与机器打交道，难免会出现需要交互的时候，比如用passwd修改密码，会提示你输入两遍密码确认；比如用ssh访问远程机器，会要求输入密码，这些都需要人工手动参与执行操作。但很多无人值守的情况下如何完成交互，这里，我们可以使用expect。

通过expect，可以让我们将预先需要准备手工干预的部分提前备好，等到出现相应提示信息（或返回信号），将我们准备的参数与要执行的语句以send命令交给机器。所以在使用expect前，你需要详细了解你要完成的任务的具体流程，详细到什么时候会出现什么提示。

---

### 语法 ###

expect遵循的是tcl语言的语法，一条Tcl命令由空格分割的单词组成. 其中, 第一个单词是命令名称, 其余的是命令参数:

    cmd 参数1 参数2 参数3...

**1、$符号**

$代表变量的值

**2、[]**

[]方括号执行了一个嵌套的命令，类似shell中的 \`cmd\`或(())或$()

**3、“”**

“blablabla” 双引号标记为命令的一个参数

**4、{}**

{blablabla} 花括号也标记为命令的一个参数，但是花括号内的其他符号不被解释

---

### 常用命令 ###

**1、spawn** 

spawn后接要执行的语句，也就是你需要向linux系统发出的shell命令

**2、expect**

expect为等待的信息，也是系统向用户反馈的提示

**3、send**

向系统发送交互的值，可以是一个shell命令，也可以是为expect等待的提示所输出的答复，比如ssh host 之后，第一步应该是先回答yes/no，此处可以用send回复yes

**4、set**

通过set定义变量并赋值

**5、interact**

执行了interact后，终端会把操作权限交给控制台，可以手动进行剩余部分的操作；比如我们通过ssh到远程主机后，在ssh、输密码阶段可以用expect完成，当连接成功并登录远程主机后，剩余的部分我们希望自己手动进行其他的操作，这时可以在expect脚本后边加上interact

**6、expect eof**

expect eof表示捕获终端输出信息的终止，通常一个自动化任务往往以interact或expect eof作为结尾

**7、exp_continue**

使用exp_continue后，会重新从当前expect块的开始重新执行，可以简单理解问while循环的continue

---

### 举几个栗子 ###


1、切换用户

```python
#!/usr/bin/expect  -f   //这个expect的路径就是用which expect 查看的结果
 
spawn su - nginx       //切换用户
expect "password:"      //提示让输入密码
send "testr"       //输入nginx的密码
interact                //操作完成
```

2、ssh到远程机器

```python
#!/usr/bin/expect  
set timeout 5
set server [lindex $argv 0]  #传递参数1
set user [lindex $argv 1]	#传递参数2
set passwd [lindex $argv 2] #传递参数3
spawn ssh -l $user $server
expect {
&quot;(yes/no)&quot; { send &quot;yesr&quot;; exp_continue }
&quot;password:&quot; { send &quot;$passwdr&quot; }
}
expect &quot;*Last login*&quot; interact
```
