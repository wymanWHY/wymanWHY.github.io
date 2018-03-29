---
title: docker中的一些技巧命令
date: 2017-02-19 18:26:54
categories: Linux OPS, docker
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
