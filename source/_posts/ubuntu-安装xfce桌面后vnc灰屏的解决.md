---
title: ubuntu 安装xfce桌面后vnc灰屏的解决
date: 2018-05-07 18:38:17
categories: Linux OPS
tags:
    - VNC
    - xfce
---

修改配置文件~/.vnc/xstartup为:

```bash
#!/bin/sh  
  
# Uncomment the following two lines for normal desktop:  
unset SESSION_MANAGER  #取消原有注释
#exec /etc/X11/xinit/xinitrc  
unset DBUS_SESSION_BUS_ADDRESS   #新增部分


[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup  
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources  
xsetroot -solid grey  
vncconfig -iconic &  
x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" & 
 

#x-window-manager &  #注释 
#新增如下针对xfce的环境配置  
x-session-manager & xfdesktop & xfce4-panel &  
xfce4-menu-plugin &  
xfsettingsd &  
xfconfd &  
xfwm4 &  
```

ubuntu server 18.04 root测试可用，解决灰屏问题

但是...

普通用户没有效果啊

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1525701274428&di=d76d4168fa24b2bb66c09c38c6b474f3&imgtype=0&src=http%3A%2F%2Fwanzao2.b0.upaiyun.com%2Fsystem%2Fpictures%2F33163314%2Foriginal%2F1455497617_500x500.png)
