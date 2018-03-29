---
title: Dockerfile 基本格式
date: 2018-03-29 18:54:19
categories: Docker
tags:
    - Docker
    - Dockerfile
---


### [参考官方文档](https://docs.docker.com/engine/reference/builder/#usage) ###


对于一个 Dockerfile 文件内容来说，基本语法格式如下所示：


    INSTRUCTION arguments

使用 # 号作为注释，指令（INSTRUCTION）不区分大小写，但是为了可读性，一般将其大写。而 Dockerfile 的指令一般包含下面几个部分：


- 基础镜像：以哪个镜像为基础进行制作，使用 FROM 指令来指定基础镜像，一个 Dockerfile 必须以 FROM 指令启动。

- 维护者信息：可以指定该 Dockerfile 编写人的姓名及邮箱，使用 MAINTAINER 指令。

- 镜像操作命令：对基础镜像要进行的改造命令，比如安装新的软件，进行哪些特殊配置等，常见的是 RUN 命令。

- 容器启动命令：基于该镜像的容器启动时需要执行哪些命令，常见的是 CMD 命令或 ENTRYPOINT

**举个栗子**：

```
# 指定基础镜像
FROM ubuntu:14.04

# 维护者信息
MAINTAINER shiyanlou/shiyanlou001@simplecloud.cn

# 镜像操作命令
RUN apt-get -yqq update && apt-get install -yqq apache2

# 容器启动命令
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

通过阅读上述内容中我们熟悉的一些 linux 指令，可以很容易的得出该命令创建了一个 apache 的镜像。包含了最基本的四项信息。

其中 **FROM** 指定基础镜像。

**RUN** 命令默认使用 /bin/sh，并使用 root 权限执行。

**CMD** 命令也是默认在 /bin/sh 中执行，但是只能有一条 CMD 指令，如果有多条则只有最后一条会被执行。

下面我们创建一个空目录，并在其中编辑 Dockerfile 文件，并基于此构建一个新的镜像，使用如下操作：

```
# 首先创建目录并切换目录
$ mkdir /home/wyman/test && cd /home/wyman/test

# 编辑 Dockerfile 文件，默认文件名为 `Dockerfile`，也可以使用其它值，使用其它值需要在构建时通过 `-f` 参数指定，这里我们使用默认值。并在其中添加上述示例的内容
$ vim Dockerfile

# 使用 build 命令，`-t` 参数指定新的镜像,注意语法，后面有个“.”，表示在此文件夹中创建容器
$ docker image build -t wyman_test:1.0 .
```
```
...
$ docker image build -t wyman:1.0 .                       [19:06:13]
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM ubuntu:14.04
14.04: Pulling from library/ubuntu
99ad4e3ced4d: Downloading  32.33MB/73MB
ec5a723f4e2a: Download complete 
2a175e11567c: Download complete 
8d26426e95e0: Download complete 
46e451596b7c: Download complete
...
```