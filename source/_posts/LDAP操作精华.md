---
title: LDAP的一些操作精髓
date: 2017-04-6 20:06:09
categories: LinuxOPS
tags:
   - LDAP
---

最近在做LDAP大整改，实现LDAP用户通过ssh访问服务器，总结一下LDAP配置中常见的一些操作

### 1、客户端的操作 ###

需要安装的包：

    nss-pam-ldapd
    pam_ldap

配置文件的修改：（将原pam_sss认证改为ldap认证）

    vi /etc/pam.d/system-auth
	vi /etc/pam.d/password-auth

```bash
auth        required      pam_env.so
auth        sufficient    pam_fprintd.so
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 500 quiet
auth        sufficient    pam_ldap.so
auth        required      pam_deny.so


account     required      pam_unix.so
account     sufficient    pam_localuser.so
account     sufficient    pam_succeed_if.so uid < 500 quiet
account     [default=bad success=ok user_unknown=ignore] pam_ldap.so
account     required      pam_permit.so


password    requisite     pam_cracklib.so try_first_pass retry=3 type=
password    sufficient    pam_unix.so md5 shadow nullok try_first_pass use_authtok
password    sufficient    pam_ldap.so use_authtok
password    required      pam_deny.so


session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
session     [success=ok default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
session     required      pam_mkhomedir.so skel=/etc/skel/ umask=0022
session     optional      pam_ldap.so
```

### 2、服务器端的操作 ###

1）修改密码

```
ldappasswd -x -D "cn=Manager,dc=domain,dc=com" "uid=username,dc=domain,dc=com" -s $passwd
# 以用户在LDAP中的dn为标识“uid=username,dc=domain,dc=com”
```

2)删除操作

删除时，给出dn即可：
```
ldapdelete -x -D 'cn=Manager,dc=domain,dc=com' -w yourpasswd -r 'dc=domain,dc=com'
-r 表示以递归模式删除，即删除该节点下面的所有子节点。
```

