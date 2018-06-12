---
title: nagios的安装及nrpe的配置
date: 2016-09-17 19:37:55
categories: Linux OPS
tags:
    - nagios
---


> Nagios是一款开源的监控系统，它利用各种插件完成所需要监控的主机和服务，一套完整的nagios系统，依赖于下面三个套件：



> - nagios 核心程序：nagios Core


> - nagios 插件： nagios-plugin


> - 被监控主机的扩展程序：nrpe



> Nagios 系统提供了一个插件NRPE，Nagios 通过周期性的运行它来获得被监控服务器的各种状态信息

调用关系如下图所示：

![](http://p7wcdketk.bkt.clouddn.com/18-5-18/77859954.jpg)

nagios本身并不需要其他的依赖包，只是在为了直观地显示监控结果的web界面会用到httpd，一套完整的nagios监控系统，包含了nagios Core、nagios-pulugin、nrpe、apache等套件。我们分为三个部分来进行叙述：

***

# 一、安装并配置需要的环境 #

操作系统：`CentOS 7`

所用到的nagios套件安装包：
<a id="download" href="https://pan.baidu.com/s/1qaxc1He32dc_ciD1JW1uCw/"><i class="fa fa-download"></i><span> 下载地址</span>
</a>
（包含nagios-4.3.4、nagios-plugin、nrpe-3.2.1）

## 1、安装依赖工具 ##

刚装完的系统空空如也，为了能正常安装nagios，首先得把依赖工具装好，这里，使用光盘自带的yum源完成工作安装，主要包含了gcc编译环境、ssl等软件包的安装：

```bash
# yum install gcc glibc glibc-common gd gd-devel xinetd openssl-devel -y
```

## 2、安装apache和PHP环境 ##

nagios的web界面，依赖apache来实现，LAMP的安装应该都烂熟于心了，这里，我们直接用光盘完成apache安装：

```bash
yum install httpd.x86_64 php-5.4.16 -y

```

## 3、创建nagios用户和组 ##

为便于管理，nagios程序的相关权限统一用nagios用户进行维护，将nagios的所有属组都划为nagios，创建nagios用户及组,这一步骤需要在监控端与被监控端都完成：

```bash
useradd nagios -s /sbin/nologin
```

可以验证一下是否创建成功：
```bash
id nagios
```

没问题则第一部分到此结束

***

# 二、服务器端的安装 #

## 1、安装nagios主程序 ##

nagios分服务端与客户端，为便于理解，**将监控主机称为服务端**，**被监控主机称为客户端**，在服务端的安装，主要是完成`nagios Core`的安装.

这里使用源码编译安装的方式完成：

```bash
# tar zxvf nagios-4.3.4.tar.gz
# cd nagios-4.3.4
# ./configure --with-nagios-user=nagios  --with-nagios-group=nagios
# make all
# make install
# make install-init
# make install-commandmode
# make install-config
# make install-webconf

```

完成后，在`/usr/local/nagios`文件夹下可以看到bin,etc,sbin,share,var等文件夹



## 2、配置nagios web界面 ##

nagios使用apache提供直观的显示界面，需要为apache配置nagios界面，执行了`make install-webconf`后，在apache的配置路径`/etc/httpd/conf.d/`下生成了nagios的httpd配置文件`nagios.conf`，将其中的内容添加到`/etc/httpd/conf/httpd.cond`文件末尾即可。



> `nagios.conf`文件内容：

```
# SAMPLE CONFIG SNIPPETS FOR APACHE WEB SERVER
#
# This file contains examples of entries that need
# to be incorporated into your Apache web server
# configuration file.  Customize the paths, etc. as
# needed to fit your system.

ScriptAlias /nagios/cgi-bin "/usr/local/nagios/sbin"

<Directory "/usr/local/nagios/sbin">
#  SSLRequireSSL
   Options ExecCGI
   AllowOverride None
   <IfVersion >= 2.3>
      <RequireAll>
         Require all granted
#        Require host 127.0.0.1

         AuthName "Nagios Access"
         AuthType Basic
         AuthUserFile /usr/local/nagios/etc/htpasswd.users
         Require valid-user
      </RequireAll>
   </IfVersion>
   <IfVersion < 2.3>
      Order allow,deny
      Allow from all
#     Order deny,allow
#     Deny from all
#     Allow from 127.0.0.1

      AuthName "Nagios Access"
      AuthType Basic
      AuthUserFile /usr/local/nagios/etc/htpasswd.users
      Require valid-user
   </IfVersion>
</Directory>

Alias /nagios "/usr/local/nagios/share"

<Directory "/usr/local/nagios/share">
#  SSLRequireSSL
   Options None
   AllowOverride None
   <IfVersion >= 2.3>
      <RequireAll>
         Require all granted
#        Require host 127.0.0.1

         AuthName "Nagios Access"
         AuthType Basic
         AuthUserFile /usr/local/nagios/etc/htpasswd.users
         Require valid-user
      </RequireAll>
   </IfVersion>
   <IfVersion < 2.3>
      Order allow,deny
      Allow from all
#     Order deny,allow
#     Deny from all
#     Allow from 127.0.0.1

      AuthName "Nagios Access"
      AuthType Basic
      AuthUserFile /usr/local/nagios/etc/htpasswd.users
      Require valid-user
   </IfVersion>
</Directory>

```

`nagios.conf`文件中定义了认证用户的认证文件名`htpasswd.users`和存放路径`/usr/local/nagios/etc/`

```
AuthUserFile /usr/local/nagios/etc/htpasswd.users
```
用`htpasswd`在对应的路径下生成对应的认证密码文件：

```bash
 htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
# 回车后提示输入密码，两次确认后密码生效
#认证用户名为nagiosadmin
```


> htpasswd.users文件中的内容为nagiosadmin密码的加密串

## 3、安装nagios插件 ##

前面说到，nagios是通过各类插件实现对各个主机及服务的监控，这里要完成的nagios-plugin是nagios官方提供的基础插件包。

同样，也是采用编译安装方式：

```bash
 tar zxvf nagios-plugins-2.1.2.tar.gz
 cd nagios-plugins-2.1.2
 ./configure --with-nagios-user=nagios --with-nagios-group=nagios
 make && make install
```

完成安装后的nagios主目录包括以下文件夹：

|文件夹|说明|
|:-|:-|
|bin|Nagios 可执行程序所在目录|
|etc|agios 配置文件所在目录|
|sbin|Nagios CGI 文件所在目录，也就是执行外部命令所需文件所在的目录|
|share|Nagios网页文件所在的目录|
|libexec|Nagios 外部插件所在目录|
|var|Nagios 日志文件、lock 等文件所在的目录|

## 4、验证 ##

启动apache服务与nagios服务，看监控服务端能否正常启动：

```bash
 service httpd start
 /etc/init.d/nagios start
```

打开浏览器，输入地址http://IP/nagios，输入安装时提供的认证用户名和密码，成功后可以看到nagios界面，完成服务端安装：

![](http://p7wcdketk.bkt.clouddn.com/18-5-18/501105.jpg)

***

# 三、客户端的安装 #

客户端指的是被监控的主机端，在被监控端，主要是进行nrpe的安装，nagios调用自身的check_nrpe与远程被监控主机端的nrpe进程通信，完成远程被监控主机上的监控数据收集。



> **在进行安装之前，和前面一样，记得先完成依赖工具的安装（gcc、openssl-devel等），和nagios用户和数组的创建**

## 1、nagios插件的安装 ##

nagios插件与上述一样，使用编译安装完成：

```bash
 tar zxvf nagios-plugins-2.1.2.tar.gz
 cd nagios-plugins-2.1.2
 ./configure --with-nagios-user=nagios --with-nagios-group=nagios
 make && make install
```

## 2、nrpe的安装 ##

```bash
 tar zxvf nrpe-3.2.1.tar.gz
 cd nrpe-3.2.1
 ./configure --with-nagios-user=nagios --with-nagios-group=nagios
 make all && make install
 make install-plugin
 make install-daemon
 make install-daemon-config
 cp sample-config/nrpe.cfg /usr/local/nagios/etc/nrpe.cfg

```

## 3、验证 ##

安装完成后，可以通过服务端的check_nrpe命令检查客户端的nrpe是否正常工作。

### 首先，在客户端启动nrpe进程： ###

启动之前，需要修改客户端的两处配置：

**1：先在客户端的`nrpe.cfg`配置文件中将服务端的ip地址写入`allowed_hosts`字段中：**

![](http://p7wcdketk.bkt.clouddn.com/18-5-18/54620624.jpg)

**2：修改/etc/xinet.d/nrpe文件：**

```bash
service nrpe {
	disable		=	no
	...
	...
	only_from	=	10.51.17.78  #nagios服务端的IP地址，如上面一致
} 
```


启动`nrpe`：
```bash
/usr/local/nagios/bin/nrpe -d -c /usr/local/nagios/etc/nrpe.cfg
```

可以通过检验**5666**端口是否打开来检查nrpe运行与否：

```bash
netstat -lunpt |grep 5666
```

### 其次，在服务端运行check_nrpe检查客户端通信正常与否： ###

```bash
/usr/local/nagios/libexec/check_nrpe -H 10.51.17.62 
```

> `check_nrpe`的用法：

> check_nrpe `–H` 被监控的主机 `-c` 要执行的监控命令

> 可以用`check_nrpe -h `来查看帮助

如果能正常返回对端的 `nrpe` 版本号，则说明通信没问题

![](http://p7wcdketk.bkt.clouddn.com/18-5-18/94068327.jpg)

否则：

1. 检查防火墙、SELinux的状态和设置
2. 检查nrpe.cfg中是否已经添加服务端地址
3. 检查服务端nagios服务的状态
4. 检查客户端nrpe进程状态



## 4、使用 ##

nagios定义了`check_nrpe`所能使用的命令，在`nrpe.cfg`文件中出现的命令定义：

```bash
// ===============================================
command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
command[check_load]=/usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,25,20
command[check_sda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/sda1
command[check_zombie_procs]=/usr/local/nagios/libexec/check_procs -w 5 -c 10 -s Z
command[check_total_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200
```

**这些命令就是服务端可以使用check_nrpe来调用查看客户端的监控命令**,等号后面所接的，就是该命令所真正执行的插件的全路径，路径后面的参数，指定了不同的具体阈值来作为“警告”的严重程度：

- `-w`表示**worning**
- `-c`表示**critical**

通过`checkk_nrpe -h`知道，check_nrpe使用 `-c`参数来进行这些命令调用

可以在服务端测试一下：

```bash
/usr/local/nagios/libexec/check_nrpe -H 10.51.17.62 -c check_load
# 查看10.51.17.62这台机子的负载情况
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-18/13700839.jpg)

当然，还可以通过上面的例子查看它的`登录用户数`、`sda1分区使用情况`、`僵尸进程数`、`总进程数`等指标

同理，我们也可以根据自己的需要增加自定义监控插件，只需要三个步骤：**【在被监控主机（客户端）】**

1. 在`/usr/local/nagios/libexec`中添加插件的执行文件。
2. 在`nrpe.cfg`文件中按照模板增加一句定义语句，指向上一步自定义插件所在的全路径，并配置好参数。
3. 重启nrpe。

即可实现对客户端的自定义监控需求