---
title: 来来来，SELinux了解一下
date: 2017-08-11 15:39:08
categories: Linux OPS
tags:
    - SELinux

---


{% asset_img 2.jpg %}

{% cq %}SELinux是个神奇的东西，这货幽灵般地存在内核中，经常让我们的程序挂得莫名其妙。{% endcq %}

<!-- more -->

---

> [参考鸟哥私房菜《第十六章、程序管理與 SELinux 初探》：](http://linux.vbird.org/linux_basic/0440processcontrol.php#selinux)
> 
## 来头 ##

SELinux 来头不小，它是由美国国家安全菊——NSA开发诞生的，它的诞生主要是用于解决**内部程序及资源误用而产生的事故，是一种系统内部控制手段，用于管理系统内应用的存取控制。**

{% asset_img 0.jpg %}

在之前，先了解并区分一下**自主式访问控制 (Discretionary Access Control, DAC)**与**强制式访问控制 (Mandatory Access Control, MAC)**

### 自主式访问控制 ###

通常情况下，我们对Linux文影件的权限可以分为rwx三种，DAC就是根据文件所带的rwx属性来决定其是否具有可读可写的能力，但DAC的缺点也很明显：
- root权限过大。如果某个程序的权限属于root，而如果一个普通用户获得了这么一个具有root权限的程序，他就有可能通过这个程序实施系统破坏
- 文件系统的权限可以任意更改。如果某个目录的权限设置为777，那么任何人都可以对这个目录进行任意操作。

### 强制式访问控制 ###

MAC将程序和路径的权限进行了管控，即是是root，在使用不同的程序时，你所能获取到的权限也不一定是root，而是要看该程序的设定来确定你对该路径具有怎样的权限。也就是说，**MAC将控制主体由用户转向了程序。**每个路径都有其所对应的主体程序，于是，在MAC的限制下，每个程序的活动范围变小了，比如，我们在开启SELinux的情况下会发现，apache只认可/var/www/html这个目录下的页面，所有设置到其他路径下的页面都无法打开。因为httpd程序只能在/var/www/下进行访问，而出了这个路径，其他目录也得要设置为httpd所能访问的类型才可以。MAC好比一个看不见的牢笼，将httpd圈在了我们所设定的范围内，架设apache真的被别有用心的人获取了控制权，他也没法访问我们系统内的其他文件。**这也是我前面说到的，SELinux强调内部程序访问管控，缩小程序的权限范围**。如图：

{% asset_img 1.jpg apache在DAC、MAC下的不同权限范围 %}

---


## security Context(安全上下文) ##

从字面意思很难理解这个概念，我们可以把它想象为一个文件的rwx属性，比如一个文件通过ls -l来查看它具有哪些rwx属性，我们可以使用ls -Z来查看一个文件具有怎样的安全上下文权限（很拗口）：

for了个example:

```bash
#查看/root目录下各个文件的安全上下文
[root@master ~]# ls -Z
-rw-------. root root  **system_u:object_r:admin_home_t:s0** anaconda-ks.cfg
drwxr-xr-x. root root  **system_u:object_r:admin_home_t:s0** Badduser
drwxr-xr-x. root root  **unconfined_u:object_r:admin_home_t:s0** Desktop
drwxr-xr-x. root root  **unconfined_u:object_r:admin_home_t:s0** Documents
drwxr-xr-x. root root  **unconfined_u:object_r:admin_home_t:s0** Downloads
drwxr-xr-x. root root  **unconfined_u:object_r:admin_home_t:s0** expect_test
drwxr-xr-x. root root  **system_u:object_r:admin_home_t:s0** GITLAB
-rw-------. root root  **system_u:object_r:admin_home_t:s0** initial-setup-ks.cfg
drwxr-xr-x. root root  **unconfined_u:object_r:admin_home_t:s0** Music
drwxr-xr-x. root root  **unconfined_u:object_r:admin_home_t:s0** Pictures
drwxr-xr-x. root root  **unconfined_u:object_r:admin_home_t:s0** Public
drwxr-xr-x. root root  **system_u:object_r:admin_home_t:s0** scripts_test
drwxr-xr-x. root root  **unconfined_u:object_r:admin_home_t:s0** Templates
drwsr-xr-x. root admin **system_u:object_r:admin_home_t:s0** test
drwxr-xr-x. root root  **unconfined_u:object_r:admin_home_t:s0** Videos

```

中间有三个冒号相连的，就是这个文件的上下文环境。大概可以解释为：

```
身份：角色：类型
```

前面两个我们不关心，其中的类型才是程序能否访问的重点。比如上面，我们在/root路径下的文件，都是admin_home_t这个类型。以鸟哥文章所举的例子为例，我们查看crond程序的上下文环境以及crond所相关的路径的上下文：

```
[root@study ~]# ps -eZ | grep cron
system_u:system_r:crond_t:s0-s0:c0.c1023 1338 ? 00:00:01 crond
system_u:system_r:crond_t:s0-s0:c0.c1023 1340 ? 00:00:00 atd
# 這個安全本文的類型名稱為 crond_t 格式！

# 2. 再來瞧瞧執行檔、設定檔等等的安全本文內容為何！
[root@study ~]# ll -Zd /usr/sbin/crond /etc/crontab /etc/cron.d
drwxr-xr-x. root root system_u:object_r:**system_cron_spool_t**:s0 /etc/cron.d
-rw-r--r--. root root system_u:object_r:**system_cron_spool_t**:s0 /etc/crontab
-rwxr-xr-x. root root system_u:object_r:**crond_exec_t**:s0 /usr/sbin/crond
```

可见，crond程序被设置为可以访问system_cron_spool_t这种类型的路径。

---

## SELinux type的查询和修改 ##

### 查看可以访问的上下文类型 ###

我们想知道某个程序的上下文类型能访问哪些对应的类型，该怎么操作，这里需要使用到setools中的sesearch命令

> setools可以通过yum完成安装，安装之后即可使用seinfo、sesearch等命令。

```
[root@study ~]# sesearch [-A] [-s 主體類別] [-t 目標類別] [-b 布林值]
選項與參數：
-A  ：列出後面資料中，允許『讀取或放行』的相關資料
-t  ：後面還要接類別，例如 -t httpd_t
-b  ：後面還要接SELinux的規則，例如 -b httpd_enable_ftp_server
```

for它个example:

```
#查看httpd主体程序的类型，发现为**httpd_exec_t**
[root@master www]# ls -Z /usr/sbin/httpd 
-rwxr-xr-x. root root system_u:object_r:httpd_exec_t:s0 /usr/sbin/httpd

...
#httpd_exec_t可以访问的类型如下：
[root@master www]# sesearch -A -s httpd_exec_t
Found 7 semantic av rules:
   allow file_type noxattrfs : filesystem associate ; 
   allow file_type fs_t : filesystem associate ; 
   allow file_type ramfs_t : filesystem associate ; 
   allow httpd_exec_t httpd_exec_t : filesystem associate ; 
   allow file_type tmpfs_t : filesystem associate ; 
   allow file_type tmp_t : filesystem associate ; 
   allow file_type hugetlbfs_t : filesystem associate ; 


```

### 修改路径的上下文类型 ###

1、chcon——修改目录的上下文类型

```
[root@study ~]# chcon [-R] --reference=範例檔 檔案
選項與參數：
-R  ：連同該目錄下的次目錄也同時修改；
-t  ：後面接安全性本文的類型欄位！例如 httpd_sys_content_t ；
-u  ：後面接身份識別，例如 system_u； (不重要)
-r  ：後面接角色，例如 system_r；     (不重要)
-v  ：若有變化成功，請將變動的結果列出來
--reference=範例檔：拿某個檔案當範例來修改後續接的檔案的類型！
```

2、restorecon——恢复默认的上下文类型

    restorecon [-Rv] 文件或目录

3、semanage——设置默认的上下文类型

restorecon是恢复至默认的上下文类型，那么通过semanage可以修改一个文件或目录的上下文类型，之后用restorecon即可设置为我们想要的类型。

```
# 1. 先建立 /srv/mycron 同時在內部放入設定檔，同時觀察 SELinux type
[root@study ~]# mkdir /srv/mycron
[root@study ~]# cp /etc/cron.d/checktime /srv/mycron
[root@study ~]# ll -dZ /srv/mycron /srv/mycron/checktime
drwxr-xr-x. root root unconfined_u:object_r:var_t:s0   /srv/mycron
-rw-r--r--. root root unconfined_u:object_r:var_t:s0   /srv/mycron/checktime

# 2. 觀察一下上層 /srv 的 SELinux type
[root@study ~]# semanage fcontext -l | grep '^/srv'
SELinux fcontext         type               Context
/srv                     all files          system_u:object_r:var_t:s0
# 怪不得 mycron 會是 var_t 囉！

# 3. 將 mycron 預設值改為 system_cron_spool_t 囉！
[root@study ~]# semanage fcontext -a -t system_cron_spool_t "/srv/mycron(/.*)?"
[root@study ~]# semanage fcontext -l | grep '^/srv/mycron'
SELinux fcontext         type               Context
/srv/mycron(/.*)?        all files          system_u:object_r:system_cron_spool_t:s0

# 4. 恢復 /srv/mycron 以及子目錄相關的 SELinux type 喔！
[root@study ~]# restorecon -Rv /srv/mycron
[root@study ~]# ll -dZ /srv/mycron /srv/mycron/*
drwxr-xr-x. root root unconfined_u:object_r:system_cron_spool_t:s0 /srv/mycron
-rw-r--r--. root root unconfined_u:object_r:system_cron_spool_t:s0 /srv/mycron/checktime
# 有了預設值，未來就不會不小心被亂改了！這樣比較妥當些～
```

