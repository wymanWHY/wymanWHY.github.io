---
title: 构建nagios性能图表：pnp4nagios的配置
date: 2016-10-28 18:47:02
categories: Linux OPS
tags:
    - nagios
    - pnp
---


> `PNP` 是 `nagios` 的一个插件，它可以分析 `plugins` 提供的性能数据，并将它们自动存储到` RRD `数据库。并
用` rrdtool `将` nagios `采集的数据绘制图表的工具。

---

# 一、安装pnp4nagios #
`PNP4nagios`基于 `php` 和 `perl` ，数据的处理和存储依赖`RRDtool`，`RRDtool`是一个高性能数据记录和制图系统，可以很容易地集成在 shell 脚本，perl，python，ruby 等应用程序中。所以在安装前，需要进行完成各依赖工具的安装

## 1、安装rrdtool ##


> 所用到的安装包：
<a id="download" href="https://pan.baidu.com/s/1qaxc1He32dc_ciD1JW1uCw/"><i class="fa fa-download"></i><span> 下载地址</span>
</a>
（包含`pnp4nagios`、`rrdtool`、`nagios`等）

我用的是系统光盘自带的`rrdtool`，当然也可以通过源码安装

```bash
yum install rrdtool.x86_64
```

## 2、安装pnp4nagios ##

使用编译安装：

```bash
tar xzvf pnp4nagios-0.6.25.tar.gz
cd pnp4nagios-0.6.25
./configure --prefix=/usr/local/pnp4nagios \
--with-nagios-user=nagios --with-nagios-group=nagios \
make all
make install
make install-init
make install-config
make install-webconf
make install-config
```

---

# 二、配置pnp4nagios #

## 1、修改httpd中关于pnp4nagios的配置 ##

安装完成后，`/etc/httpd/conf.d/`下生成pnp的web配置文件`pnp4nagios.conf`，将里面的内容追加到`httpd.conf`中

```bash
cd /etc/httpd/conf.d/
cat pnp4nagios.conf >> /etc/httpd/conf/httpd.conf
```


## 2、修改pnp4nagios中的配置文件 ##

让完成安装后的`pnp4nagios`目录下的默认配置文件生效

```bash
cd /usr/local/pnp4nagios/etc
cp misccommands.cfg-sample misccommands.cfg
cp nagios.cfg-sample nagios.cfg
cp rra.cfg-sample rra.cfg
cd pages
cp web_traffic.cfg-sample web_traffic.cfg
cd ../check_commands/
cp check_all_local_disks.cfg-sample   check_all_local_disks.cfg
cp check_nrpe.cfg-sample check_nrpe.cfg
cp check_nwstat.cfg-sample check_nwstat.cfg
cp /usr/local/pnp4nagios/libexec/* /usr/local/nagios/libexec/
```

## 3、开启 debug 模式 ##

```bash
cd /usr/local/pnp4nagios/etc
sudo vim process_perfdata.cfg
```

将里面的 `LOG_LEVEL` 修改成如下：

```bahs
LOG_LEVEL = 2
```

> 开启了这个模式后会在 `/usr/local/pnp4nagios/var/perfdata.log` 显示错误日志，方便查找原因。 

## 4、配置 Nagios 主配置文件 ##

```bash
cd /usr/local/nagios/etc
sudo vim nagios.cfg
```

修改 `process_performance_data` 的值为1
```bash
process_performance_data=1
```

并且添加如下内容： 

```
host_perfdata_file=/usr/local/pnp4nagios/var/host-perfdata
service_perfdata_file=/usr/local/pnp4nagios/var/service-perfdata

host_perfdata_file_template=DATATYPE::HOSTPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tHOSTPERFDATA::$HOSTPERFDATA$\tHOSTCHECKCOMMAND::$HOSTCHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$

service_perfdata_file_template=DATATYPE::SERVICEPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tSERVICEDESC::$SERVICEDESC$\tSERVICEPERFDATA::$SERVICEPERFDATA$\tSERVICECHECKCOMMAND::$SERVICECHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$\tSERVICESTATE::$SERVICESTATE$\tSERVICESTATETYPE::$SERVICESTATETYPE$\tSERVICEOUTPUT::$SERVICEOUTPUT$

host_perfdata_file_mode=a
service_perfdata_file_mode=a

host_perfdata_file_processing_interval=15
service_perfdata_file_processing_interval=15

host_perfdata_file_processing_command=process-host-perfdata-file
service_perfdata_file_processing_command=process-service-perfdata-file
```

> 这部分内容不用手敲，在`/usr/local/pnp4nagios/etc/`下的`nagios.conf`文件中有定义，这份文件定义了`pnp`关于数据处理5种模式中的3种，只需要选择所对应的部分内容复制下来即可。

> 这里我用的是`Buld Mode with NPCD`模式。

![](http://p7wcdketk.bkt.clouddn.com/18-5-21/14445880.jpg)

> <i class="fa fa-cloud-upload"></i>更多的描述请参考[官方文档](https://docs.pnp4nagios.org/pnp-0.6/modes#bulk_mode_with_npcd)

## 5、添加画图命令 ##

为了支持 `BULK + NPCD` 的运行模式，需要对模板中的 `process-service-perfdata-file` 与 `process-host-perfdata-file` 命令进行定义：

```bash
$ sudo vim /usr/local/nagios/etc/objects/commands.cfg 
...
#添加如下：
define command{
       command_name    process-service-perfdata-file
       command_line    /bin/mv /usr/local/pnp4nagios/var/service-perfdata /usr/local/pnp4nagios/var/spool/service-perfdata.$TIMET$
}

define command{
       command_name    process-host-perfdata-file
       command_line    /bin/mv /usr/local/pnp4nagios/var/host-perfdata /usr/local/pnp4nagios/var/spool/host-perfdata.$TIMET$
}
```

> <i class="fa fa-cloud-upload"></i>参见[官方文档](https://docs.pnp4nagios.org/pnp-0.6/config#bulk_mode_with_npcd)

## 6、将图形关联至需要监控的服务中 ##

在配置文件中，通过添加`action_url`来把图形关联到需要监控的服务或主机中：

**比如在需要监控ssh服务：**

```bash
sudo vim /usr/local/nagios/servers/ssh_SERVICE.cfg
...
define service{
	service_description   check ssh
	check_command		  check_ssh
	action_url            /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=$SERVICEDESC$
...
}
```

**比如在需要监控的主机中：**
```bash
sudo vim /usr/local/nagios/servers/host.cfg
...
define host{
	use				      linux-server
	host_name		      my server
	address		  		  10.10.10.10
	action_url            /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=_HOST_
...
}
```

> **注意：**主机和服务的微小差别


当然，如果每次新建一个监控服务或主机都得这么写会比较麻烦，我们可以把它定义到nagios的模板中：

```bash
vim /usr/local/nagios/etc/objects/templates.cfg

...
define host {
  name       host-pnp
  action_url /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=_HOST_
  register  0
}
define service {
  name       service-pnp
  action_url /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=$SERVICEDESC$
  register  0
}
```
![](http://p7wcdketk.bkt.clouddn.com/18-5-21/45718630.jpg)


使用时，在配置文件中的`use`中直接引用即可：

![](http://p7wcdketk.bkt.clouddn.com/18-5-21/12975852.jpg)

## 7、验证 ##

重启`apache`：

```bash
serivce httpd restart
```

重新加载nagios服务：

```bash
service nagios reload
```

刷新浏览器：**报错了**



![](http://p7wcdketk.bkt.clouddn.com/18-5-21/7935413.jpg)

原因在于`/var/lib/php`的权限问题，`nagios`由nagios用户运行，`php`文件夹的权限为`root`，修改权限给`nagios`用户：

```bash
chown -R nagios:nagios php
```

再运行成功。与配置Pnp之前不同的是，在被监控的边上出现了一个小框

![](http://p7wcdketk.bkt.clouddn.com/18-5-21/32899631.jpg)

点击小方框，发现提示：

![](http://p7wcdketk.bkt.clouddn.com/18-5-21/81099072.jpg)

页面提示 `pnp4nagios` 检测当前的运行文件，所有条件都满足，并在最后提示我们只需要把 `/usr/local/pnp4nagios/share/install.php` 文件删除或重命名即可正常工作，所以使用该命令：

```bash
sudo mv /usr/local/pnp4nagios/share/install.php /usr/local/pnp4nagios/share/install.php.bak
```

再进去：

![](http://p7wcdketk.bkt.clouddn.com/18-5-21/80280610.jpg)

图形效果出来了。
