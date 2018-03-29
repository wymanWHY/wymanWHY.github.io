---
title: 故障：nfs挂载出现nobody
date: 2015-06-09 18:55:56
categories: Linux OPS
tags:
    - nfs
---


### 在客户端清除idmap的缓存，然后重启rpcidmap，并重新挂载。大部分情况下可以解决 ###

```
[root@ha1 ~]# nfsidmap -c
[root@ha1 ~]# /etc/init.d/rpcidmapd restart
正在启动 RPC idmapd：                                      [确定]
正在启动 RPC idmapd：                                      [确定]
```

### 如果上面的办法没有解决，可以用下面的办法 ###

nfs服务器端，修改/etc/idmapd.conf，给Domain指定一个值，然后重启rpcidmap服务。

```
[root@ha2 ~]# vi /etc/idmapd.conf
[General]
#Verbosity = 0
# The following should be set to the local NFSv4 domain name
# The default is the host's DNS domain name.
#Domain = local.domain.edu
Domain = mydomain.com
```

重启服务

```
[root@ha2 ~]# /etc/init.d/rpcidmapd restart
Shutting down RPC idmapd:                                  [确定]
正在启动 RPC idmapd：                                      [确定]
```
nfs客户端，同样修改/etc/idmapd.conf，给Domain指定一个值，要与服务器端指定的域名相同，然后重启rpcidmap服务

```
[root@ha1 ~]# vi /etc/idmapd.conf
[General]
#Verbosity = 0
# The following should be set to the local NFSv4 domain name
# The default is the host's DNS domain name.
#Domain = local.domain.edu
Domain = mydomain.com
```

重启服务

```
[root@ha1 ~]# /etc/init.d/rpcidmapd restart

正在启动 RPC idmapd：                                      [确定]
正在启动 RPC idmapd：                                      [确定]
```

重新挂载后，发现目录属主正常，为www

```
[root@ha1 ~]# df -h
文件系统              容量  已用  可用 已用%% 挂载点
/dev/sda2             9.9G  2.3G  7.1G  25% /
tmpfs                 244M     0  244M   0% /dev/shm
/dev/sda1             194M   28M  157M  15% /boot
/dev/sda5             8.7G  148M  8.1G   2% /data
192.168.1.109:/data/nfsshare
                       19G  1.2G   17G   7% /mnt
[root@ha1 ~]# ll /mnt/
总用量 0
-rw-rw-r-- 1 www www 0 3月  25 14:15 a
-rw-rw-r-- 1 www www 0 3月  25 15:32 bb
```
