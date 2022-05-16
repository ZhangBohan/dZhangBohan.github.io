title: 如何打开MySQL查询日志？
date: 2015-07-29 10:30:55
tags:
- MySQL
- LOG
---

首先找到MySQL的配置文件`my.cnf`，在`[mysqld]`下添加

```
general_log_file=~/query.log
```

同时，登录MySQL console中设置打开log

```
mysql -uroot

> SET global general_log = 1;
```

重启MySQL之后就可以在当前用户的HOME目录中通过`query.log`查看SQL日志了。

例如，当你执行`use mysql; select * from user;`

```
150729 11:51:43	   43 Connect	root@localhost on
		   43 Query	select @@version_comment limit 1
150729 11:51:47	   43 Query	SELECT DATABASE()
		   43 Init DB	mysql
		   43 Query	show databases
		   43 Query	show tables
		   43 Field List	columns_priv
		   43 Field List	db
		   43 Field List	event
		   43 Field List	func
		   43 Field List	general_log
		   43 Field List	help_category
		   43 Field List	help_keyword
		   43 Field List	help_relation
		   43 Field List	help_topic
		   43 Field List	innodb_index_stats
		   43 Field List	innodb_table_stats
		   43 Field List	ndb_binlog_index
		   43 Field List	plugin
		   43 Field List	proc
		   43 Field List	procs_priv
		   43 Field List	proxies_priv
		   43 Field List	servers
		   43 Field List	slave_master_info
		   43 Field List	slave_relay_log_info
		   43 Field List	slave_worker_info
		   43 Field List	slow_log
		   43 Field List	tables_priv
		   43 Field List	time_zone
		   43 Field List	time_zone_leap_second
		   43 Field List	time_zone_name
		   43 Field List	time_zone_transition
		   43 Field List	time_zone_transition_type
		   43 Field List	user
		   43 Query	select * from user
```



参考：http://stackoverflow.com/questions/6479107/how-to-enable-mysql-query-log
