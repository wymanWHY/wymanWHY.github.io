---
title: docker中的一些技巧命令
date: 2017-02-19 18:26:54
categories: docker
tags:
 - docker
---
 
### 获取日志 ###
---

获取容器的输出信息可以使用如下命令：

> docker  logs [OPTIONS] CONTAINER

常用的配置项有：


> -t 或 --timestamps 显示时间戳


> -f 实时输出，类似于 tail -f

如下所示，我们查看刚刚创建的容器的日志，使用如下命令：

    $ docker container logs -tf test



### 显示进程 ###
---

除了获取日志之外，还可以显示运行中的容器的进程信息，例如查看刚刚创建的容器的进程信息：

    $ docker container top test


> **需要注意的是，该命令对于并未运行的容器是无效的**



### 查看修改 ###
---

查看相对于镜像的文件系统来说，容器中做了哪些改变，可以使用如下命令：

    docker container diff test

例如我们在 test 容器中创建一个文件123,然后删除了一个文件2，就可以使用 diff 命令查看到相应的修改：

```
$ docker diff test                                            [18:33:52]
C /root
A /root/.bash_history
A /root/123
A /root/2
```

> **它只记录了上一回的修改操作**


最后，附上docker run 的一些常用参数，参考[lynk-coder的博文《4.1 docker-1.12 run命令运行以及参数详解》](https://blog.csdn.net/kunloz520/article/details/53839237)
``` python
Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]    
  
  -d, --detach=false         指定容器运行于前台还是后台，默认为false     
  -i, --interactive=false   打开STDIN，用于控制台交互    
  -t, --tty=false            分配tty设备，该可以支持终端登录，默认为false    
  -u, --user=""              指定容器的用户    
  -a, --attach=[]            登录容器（必须是以docker run -d启动的容器）  
  -w, --workdir=""           指定容器的工作目录   
  -c, --cpu-shares=0        设置容器CPU权重，在CPU共享场景使用    
  -e, --env=[]               指定环境变量，容器中可以使用该环境变量    
  -m, --memory=""            指定容器的内存上限    
  -P, --publish-all=false    指定容器暴露的端口    
  -p, --publish=[]           指定容器暴露的端口   
  -h, --hostname=""          指定容器的主机名    
  -v, --volume=[]            给容器挂载存储卷，挂载到容器的某个目录    
  --volumes-from=[]          给容器挂载其他容器上的卷，挂载到容器的某个目录  
  --cap-add=[]               添加权限，权限清单详见：http://linux.die.net/man/7/capabilities    
  --cap-drop=[]              删除权限，权限清单详见：http://linux.die.net/man/7/capabilities    
  --cidfile=""               运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法    
  --cpuset=""                设置容器可以使用哪些CPU，此参数可以用来容器独占CPU    
  --device=[]                添加主机设备给容器，相当于设备直通    
  --dns=[]                   指定容器的dns服务器    
  --dns-search=[]            指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件    
  --entrypoint=""            覆盖image的入口点    
  --env-file=[]              指定环境变量文件，文件格式为每行一个环境变量    
  --expose=[]                指定容器暴露的端口，即修改镜像的暴露端口    
  --link=[]                  指定容器间的关联，使用其他容器的IP、env等信息    
  --lxc-conf=[]              指定容器的配置文件，只有在指定--exec-driver=lxc时使用    
  --name=""                  指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字    
  --net="bridge"             容器网络设置:  
                                bridge 使用docker daemon指定的网桥       
                                host    //容器使用主机的网络    
                                container:NAME_or_ID  >//使用其他容器的网路，共享IP和PORT等网络资源    
                                none 容器使用自己的网络（类似--net=bridge），但是不进行配置   
  --privileged=false         指定容器是否为特权容器，特权容器拥有所有的capabilities    
  --restart="no"             指定容器停止后的重启策略:  
                                no：容器退出时不重启    
                                on-failure：容器故障退出（返回值非零）时重启   
                                always：容器退出时总是重启    
  --rm=false                 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)    
  --sig-proxy=true           设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理 
```
