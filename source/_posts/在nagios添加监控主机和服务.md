---
title: 在nagios中添加监控主机和服务
date: 2016-10-06 15:10:29
categories: Linux OPS
tags:
    - nagios
    - 监控
---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1525600863413&di=5d8248fbc0f22c8f87dcc2710789e3d3&imgtype=0&src=http%3A%2F%2Foss.aliyuncs.com%2Fnetmarket%2Fimage%2F53375ac4-69f0-40e1-ad38-c8b270c7eb40.png)

<!--more-->

##  Nagios 添加监控主机 ##

操作步骤：

1、修改 Nagios 的主配置文件，将服务器的配置与监控项的配置独立出来
2、添加监控主机相关信息配置文件
3、检查配置文件语法正确性
4、重新加载配置使新增配置文件生效
5、查看 Web 界面验证配置成功

### 1、修改主配置文件 ###
在装好nagios后，可以通过web界面看到一个名为localhost的本地监控事例，这是由于 Nagios 配置文件生成的模版，该配置位于 /usr/local/nagios/etc/objects/localhost.cfg：
```bash
$ tree /usr/local/nagios/etc/    # Nagios 配置文件目录
/usr/local/nagios/etc/
|-- cgi.cfg                 # web接口配置文件
|-- htpasswd.users          # 登录 Nagios Web 页面时的用户名密码认证文件
|-- nagios.cfg                 # 主配置文件
|-- nrpe.cfg                   # 客户端配置文件
|-- objects                    # 包含其他配置文件的目录
|   |-- localhost.cfg          # 用于定义对主机监控
```

不建议将新增要监控的主机配置写在里面，因为当所要监控的主机太多，这部分会很受伤，所以，可以在`/usr/local/nagios/etc/`下通过建立专用文件夹，存放主机和服务的配置信息，并且可以通过不同的文件夹，来区分不同功能的服务、或不同平台的机器。比如：

```bash
# 以不同的云平台与功能管理
tree /usr/local/nagios/etc/
|-- cgi.cfg                 
|-- htpasswd.users          
|-- nagios.cfg              
|-- nrpe.cfg                
|-- objects                 
|-- servers         # 新增存放服务与机器目录
    |-- aliyun      # 存放 aliyun 平台相关的机器
    |   |-- product_Host.cfg
    |   |-- product_Service.cfg
    |   |-- staging_Host.cfg
    |   `-- staging_Service.cfg
    `-- aws
        |-- product_Host.cfg
        |-- product_Service.cfg
        |-- staging_Host.cfg
        `-- staging_Service.cfg
```

我们在/usr/local/nagios/etc下新建一个servers的文件夹：
```
mkdir /usr/local/nagios/etc/servers
```
为了让 Nagios 在启动的时候会读取该配置文件中我们的配置文件，我们需要修改主配置文件：` /usr/local/nagios/etc/nagios.cfg`

在里面新增一条我们新建的路径信息：`cfg_dir=/usr/local/nagios/etc/servers`

 如此一来，在这个目录下的文件只要符合 *.cfg 命名的文件就会被 nagios 加载。

### 主配置文件 nagios.cfg 介绍 ###

```
# Nagios 检索监控命令配置文件的路径
cfg_file=/etc/nagios/objects/commands.cfg                        

# Nagios 针对全局的变量配置文件，也称为资源(宏)定义文件
resource_file=/etc/nagios/resource.cfg                           

# Nagios 的状态存储数据文件，获取到的监控信息存在该文件中
status_file=/usr/local/nagios/var/status.dat                     

# Nagios 的监控状态间隔配置，默认多少秒执行一次监控命令
status_update_interval=10        

# Nagios 服务的运行用户 
nagios_user=nagios

# Nagios 服务的运行组
nagios_group=nagios                                             

# Nagios 若需要在 Web 界面中发送命令需开启该配置项
check_external_commands=1                                        

# Nagios 设置检查时间单位间隔长度，默认为 60s
interval_length=60                                               

# Nagios 是否启用通知功能，1表示启用通知，0表示关闭通知。
enable_notifications={1|0}                                       

# Nagios 设置服务检测命令的执行超时时长，若是超过该时间未能获取返回数据则判定为超时
service_check_timeout=60

# Nagios 设置主机检测命令的执行超时时长，若是超过该时间未能获取返回数据则判定机器挂掉了
host_check_timeout=30                                            

# Nagios 设置是否启用主动服务检测机制，1表示启用，0表示关闭。
execute_service_checks={1|0}

# Nagios 设置是否接受被动服务检测的结果，1表示接受，0表示拒绝。
accept_passive_service_checks={1|0}                              

# Nagios 设置是否启用主动主机检测机制，1表示启用，0表示关闭。
execute_host_checks={1|0}                                        

# Nagios 设置是否接受被动主机检测的结果，1表示接受，0表示拒绝。
accept_passive_host_checks={1|0}                                

# Nagios 设置是否启用事件处理，1表示启用，0表示关闭，也就是在服务出错时执行设置好的处理机制。
enable_event_handlers=1                                         

# Nagios 产生日志文件的路径
log_file=/usr/local/nagios/var/nagios.log 

# Nagios 指定日志转储的方法，n代表不做日志回滚
log_rotation_method={n|h|d|w|m}                                 
```
### 添加监控主机相关信息配置文件 ###

添加需要监控的主机test，在我们之前定义的主机信息路径中增加它的配置信息：`vim /usr/local/nagios/etc/servers/test.cfg`

```
define host {
        use                             linux-server
        host_name                       test
        alias                           My first Apache server
        address                         127.0.0.1
        max_check_attempts              3
        check_period                    24x7
        notification_interval           30
        notification_period             24x7
}
```

给大家解释一下都写了啥：

```
# 通过 define host 关键字来让系统识别中间的内容块是用于设置设备信息的
define host {
        # use 关键字表示使用的模版，模版将在后续讲解，此处使用的是 linux-server 模版
        use                             linux-server

        # host_name 关键字表示机器的名字，也是在 Web 界面中显示的名字
        host_name                       test

        # alias 表示机器的别名，一般用作机器别名的描述
        alias                           My first Apache server

        # address 设置该机器的 IP 地址，以便与数据的获取与被动监控的请求
        address                         127.0.0.1

        # 最大的尝试次数，也就是在某服务监控出错再次运行监控命令获取数据的次数
        max_check_attempts              3

        # 检测的时间段
        check_period                    24x7

        # 发送消息提醒的时间间隔
        notification_interval           30

        # 发送消息提醒的时间段
        notification_period             24x7
}
```
这份配置文件基本阐述了一台设备的基本信息与个别的定制化配置，当然还会有这样的一些配置项：

- hostgroups web # 所属主机组，通常可以使用主机组来管理监控内容
- check_command check-host-alive # 检测命令
- check_interval 5 # 检测间隔
- retry_interval 1 # 重试间隔，一次检测失败后，重试的间隔
- contacts shiyanlou # 联系人
- contact_groups linux-admins # 联系人组，在机器出现问题的时候告警信息发送的人员组
- notification_options d,u,r # 哪种状态进行通知；d(Down)，u(UNREACHABLE)，r- (recovery)，f，s } ，各状态及其表示符如下：
|参数|含义|
|:-|:-|
|d —— DOWN |                      #挂了|
|u —— UNREACHABLE      |          #不可达|
|r —— UP(host recovery)    |      #重新恢复态|
|f —— flapping                |   #异常|
|s |调试宕机时间开始或结束|

### 检查配置文件 ###

每次修改配置文件之后我们需要通过这样的命令来检查我们的配置文件是否正确，因为若是直接重启或者重新加载使得配置文件生效，但配置中却有错误的话会导致我们的监控服务直接不可用，所以每次在重启之前我们都需要先检查一遍：

```bash
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

![](http://p7wcdketk.bkt.clouddn.com/18-5-6/18314253.jpg)

检查发现配置无误的情况下，我们重新加载一下nagios服务:

```bash
service nagios reload
```

打开web登录界面，点击左边`Hosts`标签栏，可以看到我们新增的那台主机test（目前也就是本机）：

![](http://p7wcdketk.bkt.clouddn.com/18-5-6/52130009.jpg)

总结一下，总的来说，增加被监控主机可以分为两个步骤：

- 1、 添加/修改主机配置文件
- 2、 重新加载nagios服务


## 添加监控服务 ##


监控主机只能看到主机的开关机状态，但我们往往更关心主机上所运行的业务是否正常，所以，我们还需要在nagios上添加我们需要监控的服务。

### 创建监控服务配置文件 ###

一般把**监控主机**和**监控服务**分开放置在两个配置文件中，比如下面的文件结构：

```bash
|-- servers         
    |-- test_Host.cfg
    `-- test_Service.cfg 
```

`test_Service.cfg`文件就是监控服务的配置文件：

```bash
define service {
      host_name                       test
      service_description             Check SSH
      check_command                   check_ssh
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}
```

是不是和添加主机的配置文件很相似？解释一下不同之处吧：

```bash
host_name：该配置项告知 Nagios 该监控服务针对哪台设备
use：这里没有使用 use 关键字，但是在 service 中也有模版的概念，用法与 host 中的一样
check_command：该配置项指定此监控服务使用的命令（使用的命令必须在 commands.cfg 配置文件中有所指定）
max_check_attempts：该配置项指定当服务检查出现问题是最大的尝试次数
```
> 只罗列了常用的几项，具体可以参考[官方手册](https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/3/en/objectdefinitions.html#service)

### 命令配置文件 ###

在服务的配置文件中有一项名为`check_command`的配置项，它指定了监控服务所用的命令，而这个命令，就是由commands.cfg所定义的，它存在于:

```bash
/usr/local/nagios/etc/objects/commands.cfg
```

查看一下这份文件，可以看到里面定义了一系列的命令：
![](http://p7wcdketk.bkt.clouddn.com/18-5-6/1884406.jpg)

其中:

- command_name：也就是命令的名字，在 service 中我们所使用的命令
- command_line：也就是命令所使用的脚本/插件，与使用该脚本时的相关参数

比如我们上面举例要监控的ssh服务的命令check_ssh：
![](http://p7wcdketk.bkt.clouddn.com/18-5-6/81610197.jpg)

**它所用到的脚本(或称为插件)，存在于路径$USER1下的名为check_ssh的脚本**

而$USER1变量的定义，是在`usr/local/nagios/etc/resource.cfg`这份配置文件里定义的。

好奇的话可以打开看一下：

`vi usr/local/nagios/etc/resource.cfg`

![](http://p7wcdketk.bkt.clouddn.com/18-5-6/96160418.jpg)

可以看到，USER1变量的值为路径`/usr/local/nagios/libexec`,也就是说check_ssh这份真正在背后执行的脚本，存放在:

```bash
/usr/local/nagios/libexec
```

那我们看看这个路径下还都有些啥：

![](http://p7wcdketk.bkt.clouddn.com/18-5-6/57031443.jpg)

原来，这个路径是用于存放nagios监控过程中真正执行的脚本/插件，因此，我们也可以根据实际需要自己开发相应的脚本/插件，将他们放在这个路径下，实现自己的监控需求，该路径下的脚本/插件只需要有可执行权限即可，无论是C、shell还是PHP、python编写的，都可以。

### 检查配置文件 ###

同样，在新增了一个监控服务之后，我们试着检查一下配置文件是否有误：

```bash
nagios /usr/local/nagios/etc/nagios.cfg
```

确认无误后重新加载nagios服务：

```bash

service nagios reload

```

打开web界面，点击左边的`Services`标签，可以看到新增加的check_ssh服务：

![](http://p7wcdketk.bkt.clouddn.com/18-5-6/57031443.jpg)

> 若是看到尾pending状态，请再等会，监控的轮询时间间隔没到


## 添加联系人 ##

在监控的默认联系人为nagiosadmin，如果需要更改，请在`/usr/local/nagios/etc/objects/contact.cfg`文件中指定：
![](http://p7wcdketk.bkt.clouddn.com/18-5-6/51505824.jpg)

> 可以根据默认的模板进行修改，不做解释了



## 添加组 ##

当遇到类似功能或具有相同业务的服务时，我们可以分组管理，定义组的语法如下：


```bash
define hostgroup{
    hostgroup_name        test_servers
    alias                Servers For Test
    members                T1,T2,T3
    }
```

- hostgroup_name：设置 group 的名字
- alias：对该组的一个描述
- members：该组中所拥有的成员

定义组成员的方式，如果不想把`members`字段写得老长老长，还可以在每个主机的配置文件中添加下面的配置项：

```bash
hostgroups test_servers
```

当我们需要对一组服务器，统一监控每台服务器上的某个服务（假设是ssh服务），可以在service的定义中加入`hostgroup_name`，于是对应组的成员便都会监控该服务，这样就可以省下了许多重复性的配置工作。

比如，我们新加一台主机名为test2，分组为test_group，用于监控ssh服务。实现的步骤如下：

**1、首先在两台主机的配置文件中定义其所在组**:

```bash
hostgrous test_group
``` 

**2、其次在建立监控服务配置文件，在配置文件中：**

```bash
# 删除内容
host_name              test

# 添加内容
hostgroup_name         test_group

```

并且在空白处新增一块关于组的定义：

```bash

define hostgroup{
    hostgroup_name          test_group
    alias                   For test
}

```

**3、检查配置，并重新加载nagios服务：**

```bash
略
```

完成后我们打款web界面查看一下

![](http://p7wcdketk.bkt.clouddn.com/18-5-6/85896236.jpg)

![](http://p7wcdketk.bkt.clouddn.com/18-5-6/73925537.jpg)

可以看到，两台主机的信息都生效了，且监控的ssh服务也出现在了列表中。

---

OK，nagios的基本操作就先介绍这么多

后面会介绍到自开发的脚本如何应用、创建Nagios自定义模板，以及如何构建图表