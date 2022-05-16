title: "MySQL错误 ERROR 2006 (HY000): MySQL server has gone away"
date: 2015-04-2 10:33:46
tags:
- MySQL
- DEBUG
---
MySQL下当我导入一个比较大的SQL文件时出现了`ERROR 2006 (HY000): MySQL server has gone away`错误，具体情况如下：

```
>  ll *.sql
-rwxr-xr-x@ 1 bohan  staff    27M Mar 26 18:08 91620_all.sql

> mysql test < 91620_all.sql
ERROR 2006 (HY000) at line 17128: MySQL server has gone away
```

上面可以看到，文件大小为27M导入的时候会报这个错误。

## 错误原因
> If you are using the mysql client program, its default max_allowed_packet variable is 16MB. To set a larger value, start mysql like this:

```
shell> mysql --max_allowed_packet=32M
```
> That sets the packet size to 32MB.

我们通过MySQL[相关文档](https://dev.mysql.com/doc/refman/5.1/en/packet-too-large.html)可以发现默认大小是16M。

## 解决方法
所有大于16M的SQL文件都会报这个错误。

不过我们可以直接通过命令后增加`--max_allowed_packet=32M`解决

或者登录MySQL客户端，修改[系统变量](http://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_max_allowed_packet)：

```
> ssh mysql
mysql> set GLOBAL max_allowed_packet=32*1024*1024;
```

我们也可以通过修改MySQL配置`my.cnf`文件，在最后一行增加`max_allowed_packet=32M`就可以了

MySQL配置文件的位置：

* Windows下 `C:\ProgamData\MySQL\MySQL Server5.6`
* Linux下 `/etc/mysql`
* Mac下通过brew安装 `/usr/local/Cellar/mysql/5.6.23`
