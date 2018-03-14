---
title: 利用crontab实现SVN的自动化备份
date: 2014-11-25 11:43:31
categories: 
	- Linux OPS
	- shell
tags:
	- svn 备份
	- crontab
	- shell
---




svn作为集中式的版本控制系统，由于数据的集中存储，备份是必不可少的;
 
> svn的备份机制大致有以下三种：
- svnadmin dump 
- svnadmin hotcopy 
- svnsync



> - 第一种svnadmin dump是官方推荐的备份方式，优点是比较灵活，可以全量备份也可以增量备份，并提供了版本恢复机制。 
缺点是：如果版本比较大，如版本数增长到数万、数十万，那么dump的过程将非常慢；备份耗时，恢复更耗时；不利于快速进行灾难恢复。 
个人建议在版本数比较小的情况下使用这种备份方式。 
- 第二种svnadmin hotcopy原设计目的估计不是用来备份的，只能进行全量拷贝，不能进行增量备份； 
优点是：备份过程较快，灾难恢复也很快；如果备份机上已经搭建了svn服务，甚至不需要恢复，只需要进行简单配置即可切换到备份库上工作。 
缺点是：比较耗费硬盘，需要有较大的硬盘支持（俺的备份机有1TB空间，呵呵）。 
- 第三种svnsync实际上是制作2个镜像库，当一个坏了的时候，可以迅速切换到另一个。不过，必须svn1.4版本以上才支持这个功能。 
优点是：当制作成2个镜像库的时候起到双机实时备份的作用； 
缺点是：当作为2个镜像库使用时，没办法做到“想完全抛弃今天的修改恢复到昨晚的样子”；而当作为普通备份机制每日备份时，操作又较前2种方法麻烦。
————引自[头痛不痛的博客](https://www.cnblogs.com/zydev/p/5370512.html)

当备份涉及的项目过多，人工备份耗时耗力，我们可以**利用系统的cron计划，编写脚本实现svn的自动备份**



 #### 完全备份 #### 
    ```bash
    #!/bin/bash
    
    BACKUP_CMD="/usr/bin/svnadmin hotcopy" #使用svn自带的hotcopy备份机制进行全备份
    SVN_LOOK="svnlook youngest"
    SVN_ROOT="/svndata"
    SVN_BACKUP_ROOT=/svn_backup
    BACKUP_DATE=$(date +%Y-%m-%d)
    
    cp -rf $SVN_ROOT/passwd $SVN_BACKUP_ROOT
    cp -rf $SVN_ROOT/authz_bak $SVN_BACKUP_ROOT
    cp -rf $SVN_ROOT/svnserve_bak $SVN_BACKUP_ROOT
    
    
    #检查目标路径是否存在!
    if [ ! -e $SVN_BACKUP_ROOT ]
    then
     mkdir $SVN_BACKUP_ROOT
    fi
    REPO_NAME_LIST=$(ls $SVN_ROOT) #记录版本库名称
    
    #定义logfile
    LOGFILE=$SVN_BACKUP_ROOT/full_backup.log
    echo "====================$BACKUP_DATE========================" >>$LOGFILE
    
    #通过创建的文件夹命名日期来判断备份与否，如果已备份，退出
    if [ -e $SVN_BACKUP_ROOT/Full_"$BACKUP_DATE" ]
    then
     echo "$(date +%T)==>$SVN_BACKUP_ROOT/Full_"$BACKUP_DATE" already exists!" >>$LOGFILE
     echo "Thers is no need backup again!" >>$LOGFILE
     exit 1
    else 
     mkdir $SVN_BACKUP_ROOT/Full_"$BACKUP_DATE"
    fi
    
	#记录备份项目的版本号信息  
    if [ ! -e $SVN_BACKUP_ROOT/full_repo_revision ]
    then
     mkdir $SVN_BACKUP_ROOT/full_repo_revision
    fi
    FULL_REPO_VERSION=$SVN_BACKUP_ROOT/full_repo_revision
    
    #启动备份
    for repo_name in $REPO_NAME_LIST
    do
     echo $repo_name
     SOURCE_DIR=$SVN_ROOT/$repo_name
     DES_DIR=$SVN_BACKUP_ROOT/Full_"$BACKUP_DATE"/$repo_name
     #get repo version
     get_repo_version=$($SVN_LOOK $SVN_ROOT/$repo_name)
     $BACKUP_CMD $SOURCE_DIR $DES_DIR
    	echo "$(date +%T)==> repository:[$repo_name] at version($get_repo_version) Successfull Full Backup !" >>$LOGFILE
    	echo "$get_repo_version" > $FULL_REPO_VERSION/$repo_name.number
    done
    
    echo "" >>$LOGFILE
	```

#### 增量备份 ####

与完成备份的脚本差不多
```bash
    #!/bin/bash

    SVN_ROOT="/file/svndata"
    SVN_BACKUP_ROOT=/svn_backup
    BK_DATE=$(date +%Y-%m-%d)
    DUMP="svnadmin dump"  #用dump进行增量备份
    LOOK="svnlook youngest" #最后用svnlook来判断备份的情况

    REPO_NAME_LIST=$(ls $SVN_ROOT)
    THIS_NUM_FILE=$SVN_BACKUP_ROOT/Increment_"$BK_DATE"/repo_version_number
    LAST_FULL_NUM=$SVN_BACKUP_ROOT/full_repo_revision

    LOGFILE=$SVN_BACKUP_ROOT/increment_backup.log

    echo "=============$(date +%Y-%m-%d_%T)==============" >>$LOGFILE

    if [ ! -e $SVN_BACKUP_ROOT/Increment_"$BK_DATE" ]
    then
     echo "no Increment Directory , We are making one..."
     mkdir $SVN_BACKUP_ROOT/Increment_"$BK_DATE"
     echo "Increment Directory Created!"
    else
     echo "$SVN_BACKUP_ROOT/Increment_"$BK_DATE" backup directory already exists!" >>$LOGFILE
     echo "There is no need to backup again!!" >>$LOGFILE
     exit 1
    fi
    

    echo " ">>$THIS_NUM_FILE
    echo "=============$(date +%Y-%m-%d_%T)==============" >>$THIS_NUM_FILE

    if [ -e "$LAST_FULL_NUM" ]
    then
      for repo_name in $REPO_NAME_LIST
    	SOURCE=$SVN_ROOT/$repo_name
    	DEST=$SVN_BACKUP_ROOT/Increment_"$BK_DATE"/$repo_name.dumpfile
      do
    thisnum=$($LOOK $SVN_ROOT/$repo_name)
    echo "The latest rivision of [$repo_name] is : $thisnum" >>$THIS_NUM_FILE
    lastnum=$(cat "$LAST_FULL_NUM"/"$repo_name".number)
    if [ $thisnum -eq $lastnum ]
    then
      echo "There is no new version for [$repo_name]!!" >>$LOGFILE
    else
      echo "Increment Backup working... "
      $DUMP $SVN_ROOT/$repo_name -r $lastnum:$thisnum --incremental >$SVN_BACKUP_ROOT/Increment_"$BK_DATE"/$repo_name.dumpfile
       echo "Backup for [$repo_name] from VERSION $lastnum to VERSION $thisnum succeed!" >>$LOGFILE
    fi
      done
	echo " " >>$LOGFILE
```


### 添加至cron计划 ###

我们定义备份策略为：每周进行一次增量备份，每周六启动备份；每月进行一次完成备份，每月1号启动备份
则crontab可以写为：

```bash
	01 02 01 * * /full_bk.sh #在每月第一天的凌晨2:01启动完全备份脚本；
	01 02 * * 6 /incr_bk.sh  #每周六凌晨2:01启动增量备份脚本；
```

（完全备份脚本名为full_bk.sh，增量备份脚本文件名为incr_bk.sh）



**PS:备份后注意空间的增长**