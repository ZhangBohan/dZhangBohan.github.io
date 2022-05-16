title: 数据库版本管理工具Liquibase
date: 2019-12-24 23:34:26
tags: database, migrate, java
---

![](https://image.1programmer.com/2019-12-24-15772015720671.jpg)

研发过程中经常涉及到数据库变更，对表结构的修复及对数据的修改，为了保证各环境都能正确的进行变更我们可能需要维护一个数据库升级文档来保存这些记录，有需要升级的环境按文档进行升级。

这样手工维护有几个缺点：

1. 无法保证每个环境都按要求执行
2. 遇到问题不一定有相对的回滚语句
3. 无法自动化

为了解决这些问题，我们进行了一些调研，主要调研对象是Liquibase和Flyway，我们希望通过数据库版本管理工具实现以下几个目标：

1. 数据库升级
2. 数据库回滚
3. 版本标记

调研过程中发现Flyway据库回滚功能是增值功能且实现逻辑是通过我们的升级脚本来进行“智能”降级，不符合我们目前的使用场景，Flyway相关的介绍可以看我早期的另一篇介绍：https://segmentfault.com/a/1190000000422670

## Liquibase

> Liquibase帮助团队跟踪、版本化及部署数据库架构和逻辑修改

### 安装

#### 检查JRE

```
$ java -version
java version "1.8.0_231"
Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
```
如果没有安装Java请自行安装

#### 安装Liquibase

下载[Liquibase-Version#-bin.tar.gz 文件](https://download.liquibase.org/download/)，
解压压缩包，将目录添加到环境变量中

```
$ export PATH="/opt/liquibase-3.8.2:$PATH"
```

这个命令重新启动命令行就不会生效了，如果要保证一直可用需要将这个领先设置到`.bashrc`或者`.zshrc`中

通过运行帮助命令验证安装

```
$ liquibase --help
17:12:10.389 [main] DEBUG liquibase.resource.ClassLoaderResourceAccessor - Opening jar:file:/opt/liquibase-3.8.2/liquibase.jar!/liquibase.build.properties as liquibase.build.properties
Starting Liquibase at 星期三, 04 十二月 2019 17:12:10 CST (version 3.8.2 #26 built at Tue Nov 26 04:53:39 UTC 2019)


Usage: java -jar liquibase.jar [options] [command]

Standard Commands:
...
```

#### 配置文件

下面是以mysql为例的配置文件

```
$ cat liquibase.properties
driver: com.mysql.cj.jdbc.Driver
classpath: ./mysql-connector-java-8.0.18.jar
url: jdbc:mysql://127.0.0.1/test
username: root
password: 123456
changeLogFile: myChangeLog.xml
```

### 数据库升级

创建`myChangeLog.xml`文件，这个文件用来记录升级记录升级信息，初始化的内容

```
$ cat myChangeLog.xml
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

</databaseChangeLog>
```

Liquibase支持通过SQL描述的方式来创建数据库

```
$ cat myChangeLog.xml
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

    <changeSet id="1.0" author="bohan">
        <sql>
        CREATE TABLE `deparment` (
        `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
        `name` varchar(100) COLLATE utf8mb4_bin DEFAULT NULL,
        PRIMARY KEY (`id`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
        </sql>
    </changeSet>
</databaseChangeLog>
$ liquibase update
Liquibase Community 3.8.2 by Datical
Liquibase: Update has been successful.
```

通过执行`liquibase update`进行升级，升级后的数据库如下，已经为我们创建了数据库，同时Liquibase生成了两个表用来管理数据库升级记录

```
$ mysql -h 127.0.0.1 -uroot -p123456 test -e "show tables;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+-----------------------+
| Tables_in_test        |
+-----------------------+
| DATABASECHANGELOG     |
| DATABASECHANGELOGLOCK |
| deparment             |
+-----------------------+
```

继续执行升级

```
$ cat myChangeLog.xml
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

    <changeSet id="1.0" author="bohan">
        <sql>
        CREATE TABLE `deparment` (
        `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
        `name` varchar(100) COLLATE utf8mb4_bin DEFAULT NULL,
        PRIMARY KEY (`id`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
        </sql>
    </changeSet>
    <changeSet id="1.1" author="bohan">
        <sql>
        insert into deparment values(1, "test");
        </sql>
    </changeSet>
</databaseChangeLog>
$ liquibase update
Liquibase Community 3.8.2 by Datical
Liquibase: Update has been successful.
$ mysql -h 127.0.0.1 -uroot -p test -e "select * from deparment;"
Enter password:
+----+------+
| id | name |
+----+------+
|  1 | test |
+----+------+
```

数据如预期被添加

#### 通过SQL文件
数据库变更也可以通过sql文件形式引用，避免`myChangeLog.xml`文件过大

```
<changeSet id="1.1" author="bohan">
    <sqlFile path="./update_deparment_name.sql"></sqlFile>
</changeSet>
```

### 数据库回滚

```
liquibase --help
Usage: java -jar liquibase.jar [options] [command]

Standard Commands:
 rollbackCount <value>          Rolls back the last <value> change sets
                                applied to the database
```

我们来执行`rollbackCount`进行回滚

```
$ liquibase rollbackCount 1
Liquibase Community 3.8.2 by Datical
Rolling Back Changeset:myChangeLog.xml::1.0::bohan
Unexpected error running Liquibase: No inverse to liquibase.change.core.RawSQLChange created
For more information, please use the --logLevel flag
```

提示没有回滚SQL，修改我们的`myChangeLog.xml`

```
$ cat myChangeLog.xml
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

    <changeSet id="1.0" author="bohan">
        <sql>
        CREATE TABLE `deparment` (
        `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
        `name` varchar(100) COLLATE utf8mb4_bin DEFAULT NULL,
        PRIMARY KEY (`id`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
        </sql>
        <rollback>
        DROP TABLE deparment;
        </rollback>
    </changeSet>
    <changeSet id="1.1" author="bohan">
        <sql>
        insert into deparment values(1, "test");
        </sql>
        <rollback>
        DELETE FROM deparment WHERE id = 1;
        </rollback>
    </changeSet>
</databaseChangeLog>
```

执行回滚，发现已经没有新增的记录了

```
liquibase rollbackCount 1
Liquibase Community 3.8.2 by Datical
Rolling Back Changeset:myChangeLog.xml::1.1::bohan
Liquibase: Rollback has been successful.
$ mysql -h 127.0.0.1 -uroot -p test -e "select * from deparment;"
Enter password:
```

再次执行，数据库也如预期被删除

```
$ liquibase rollbackCount 1
Liquibase Community 3.8.2 by Datical
Rolling Back Changeset:myChangeLog.xml::1.0::bohan
Liquibase: Rollback has been successful.
$ mysql -h 127.0.0.1 -uroot -p test -e "show tables;"
Enter password:
+-----------------------+
| Tables_in_test        |
+-----------------------+
| DATABASECHANGELOG     |
| DATABASECHANGELOGLOCK |
+-----------------------+
```

### 版本标记

Liquibase提供了完善的标签功能，经过刚刚的回滚到上一次操作后我们目前只执行了ID为1.0的变更

```
mysql> select * from DATABASECHANGELOG;
+-----+--------+-----------------+---------------------+---------------+----------+------------------------------------+-------------+----------+------+-----------+----------+--------+---------------+
| ID  | AUTHOR | FILENAME        | DATEEXECUTED        | ORDEREXECUTED | EXECTYPE | MD5SUM                             | DESCRIPTION | COMMENTS | TAG  | LIQUIBASE | CONTEXTS | LABELS | DEPLOYMENT_ID |
+-----+--------+-----------------+---------------------+---------------+----------+------------------------------------+-------------+----------+------+-----------+----------+--------+---------------+
| 1.0 | bohan  | myChangeLog.xml | 2019-12-05 03:15:18 |             1 | EXECUTED | 8:fe52f094e795797c89459e8f22483482 | sql         |          | NULL | 3.8.2     | NULL     | NULL   | 5515718387    |
+-----+--------+-----------------+---------------------+---------------+----------+------------------------------------+-------------+----------+------+-----------+----------+--------+---------------+
1 row in set (0.00 sec)
```

在实际开发中，我们升级版本时通常会需要同时执行多个变更，如果变量存在问题需要回滚时按数量回滚就比较麻烦了，我们需要对我们的变更进行标签标记，下面可能用到的命令如下：

```
liquibase --help
11:21:31.994 [main] DEBUG liquibase.resource.ClassLoaderResourceAccessor - Opening jar:file:/opt/liquibase-3.8.2/liquibase.jar!/liquibase.build.properties as liquibase.build.properties
Starting Liquibase at 星期四, 05 十二月 2019 11:21:31 CST (version 3.8.2 #26 built at Tue Nov 26 04:53:39 UTC 2019)


Usage: java -jar liquibase.jar [options] [command]

Standard Commands:
 rollback <tag>                 Rolls back the database to the the state is was
Maintenance Commands
 tag <tag string>          'Tags' the current database state for future rollback
 tagExists <tag string>    Checks whether the given tag is already existing
```

针对当前数据库，我们通过`liquibase tag`进行打标签操作

```
$ liquibase tag v1.0
Liquibase Community 3.8.2 by Datical
Successfully tagged 'root@172.17.0.1@jdbc:mysql://127.0.0.1/test'
Liquibase command 'tag' was executed successfully.
```

查看记录发现ID为`1.0`的记录TAG中已设置为`v1.0`，符合我们的预期

```
mysql> select * from DATABASECHANGELOG;
+-----+--------+-----------------+---------------------+---------------+----------+------------------------------------+-------------+----------+------+-----------+----------+--------+---------------+
| ID  | AUTHOR | FILENAME        | DATEEXECUTED        | ORDEREXECUTED | EXECTYPE | MD5SUM                             | DESCRIPTION | COMMENTS | TAG  | LIQUIBASE | CONTEXTS | LABELS | DEPLOYMENT_ID |
+-----+--------+-----------------+---------------------+---------------+----------+------------------------------------+-------------+----------+------+-----------+----------+--------+---------------+
| 1.0 | bohan  | myChangeLog.xml | 2019-12-05 03:15:18 |             1 | EXECUTED | 8:fe52f094e795797c89459e8f22483482 | sql         |          | v1.0 | 3.8.2     | NULL     | NULL   | 5515718387    |
+-----+--------+-----------------+---------------------+---------------+----------+------------------------------------+-------------+----------+------+-----------+----------+--------+---------------+
1 row in set (0.00 sec)
```

执行更新后如果需要回滚通过`liquibase rollback v1.0`即可

```
$ liquibase update
Liquibase Community 3.8.2 by Datical
Liquibase: Update has been successful.

mysql> select * from DATABASECHANGELOG;
+-----+--------+-----------------+---------------------+---------------+----------+------------------------------------+-------------+----------+------+-----------+----------+--------+---------------+
| ID  | AUTHOR | FILENAME        | DATEEXECUTED        | ORDEREXECUTED | EXECTYPE | MD5SUM                             | DESCRIPTION | COMMENTS | TAG  | LIQUIBASE | CONTEXTS | LABELS | DEPLOYMENT_ID |
+-----+--------+-----------------+---------------------+---------------+----------+------------------------------------+-------------+----------+------+-----------+----------+--------+---------------+
| 1.0 | bohan  | myChangeLog.xml | 2019-12-05 03:15:18 |             1 | EXECUTED | 8:fe52f094e795797c89459e8f22483482 | sql         |          | v1.0 | 3.8.2     | NULL     | NULL   | 5515718387    |
| 1.1 | bohan  | myChangeLog.xml | 2019-12-05 03:28:06 |             2 | EXECUTED | 8:695a5ec0b2b3ddc4a9beeeca530adebc | sql         |          | NULL | 3.8.2     | NULL     | NULL   | 5516486105    |
+-----+--------+-----------------+---------------------+---------------+----------+------------------------------------+-------------+----------+------+-----------+----------+--------+---------------+
2 rows in set (0.00 sec)


$ liquibase rollback v1.0
Liquibase Community 3.8.2 by Datical
Rolling Back Changeset:myChangeLog.xml::1.1::bohan
Liquibase: Rollback has been successful.

mysql> select * from DATABASECHANGELOG;
+-----+--------+-----------------+---------------------+---------------+----------+------------------------------------+-------------+----------+------+-----------+----------+--------+---------------+
| ID  | AUTHOR | FILENAME        | DATEEXECUTED        | ORDEREXECUTED | EXECTYPE | MD5SUM                             | DESCRIPTION | COMMENTS | TAG  | LIQUIBASE | CONTEXTS | LABELS | DEPLOYMENT_ID |
+-----+--------+-----------------+---------------------+---------------+----------+------------------------------------+-------------+----------+------+-----------+----------+--------+---------------+
| 1.0 | bohan  | myChangeLog.xml | 2019-12-05 03:15:18 |             1 | EXECUTED | 8:fe52f094e795797c89459e8f22483482 | sql         |          | v1.0 | 3.8.2     | NULL     | NULL   | 5515718387    |
+-----+--------+-----------------+---------------------+---------------+----------+------------------------------------+-------------+----------+------+-----------+----------+--------+---------------+
1 row in set (0.00 sec)
```
