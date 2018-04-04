---
title: 故障：ldapadd 报错
date: 2018-04-03 15:01:14
categories: Linux OPS
tags:
    - LDAP
---

### Can't contact LDAP server ###

使用LDAPADD配置LDAP用户的时候，突然报错：

```
ldap_sasl_bind(SIMPLE): Can't contact LDAP server (-1)
```

**原因在于：**使用的主机名称与dc名不一致导致的

**解决办法：**用 -h 参数指定主机名，-p 参数指定端口号**389**

```
ldapadd -x -D cn=Manager,dc=my-domain,dc=com -W -f XXX/XX.ldif -h MYHOSTNAME -p 389
```


---

### Invalid credentials  ###

另一个报错：

```
ldapadd ldap_bind: Invalid credentials (49)
```

**原因：**可能在于配置目录信息不一致

**解决办法：**

- 清空/etc/openldap/slapd.d/下的所有文件
 
 `rm -rf /etc/openldap/slapd.d/*`

- 重新进行一次slaptest测试，写入/etc/openldap/slapd.d/

    slaptest -f /etc/openldap/slapd.conf -F /etc/openldap/slapd.d
