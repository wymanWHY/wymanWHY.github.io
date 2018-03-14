---
title: 在gitbash中使用markdown
date: 2014-12-07 18:48:30
categories: git
tags:
	- gitbash
	- markdownpad
	- notepad
---

习惯了通过命令行的方式调用带图形界面的文本编辑器，比如gedit、kwrite
在windows的gitbash环境下调取系统下的文本编辑器，也可以通过类似Linux中的修改配置bashrc文件的方式解决：

## windows下的gitbash配置文件如何找？

通过df -kh
```bash
df -kh
```
发现，/目录被挂载在了“C:/Program Files/Git”下，进入“C:/Program Files/Git/etc”目录，发现bash.bashrc文件，
添加notepad++可执行程序所在路径：
```bash
alias np="'C:/Program Files (x86)/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```
保存后，source一下即可使用np命令使用notepad++


同理，markdownpad也一样：
```bash
alias md="'C:\Program Files (x86)\MarkdownPad 2\MarkdownPad2.exe' -multiInst -notabbar -nosession -noPlugin"
```
