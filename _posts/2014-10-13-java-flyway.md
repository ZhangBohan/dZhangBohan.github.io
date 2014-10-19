---
layout: post
title:  "java敏捷数据库迁移框架——flyway"
date:   2014-10-13 01:30:00
categories: 有码
tags: DB JAVA
---

![image](/images/flyway-logo.png)

看看自己的项目的那些SQL文件或者干脆连个建表语句都没有的同学是否会有想法把他们管理起来呢？向大家推荐一款非常轻量级的敏捷数据库迁移框架——[Flyway](http://flywaydb.org/)。想知道她有什么魅力吗？

Flyway为大家提供了如下的实现方式：

* Java API
* 命令行
* Maven
* Gradle
* Ant
* SBT

为了减少描述难度在这里使用了**Java API**，项目构建方式为**Maven**，数据库为**MySQL**

## 需要环境
* [Java 6+](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
* [Maven 2+](http://maven.apache.org/)
* [MySQL](http://dev.mysql.com/downloads/)

## 创建项目
首先我们要在命令行中使用Maven原型插件执行如下命令

	mvn archetype:generate -B \
		        -DarchetypeGroupId=org.apache.maven.archetypes \
		        -DarchetypeArtifactId=maven-archetype-quickstart \
		        -DarchetypeVersion=1.1 \
		        -DgroupId=foo \
		        -DartifactId=bar \
		        -Dversion=1.0-SNAPSHOT \
		        -Dpackage=foobar

我们已经准备好开始了。当前项目的结构如下


	.
	|-- pom.xml
	`-- src
	    |-- main
	    |   `-- java
	    |       `-- foobar
	    |           `-- App.java
	    `-- test
	        `-- java
	            `-- foobar
	                `-- AppTest.java


进入创建的项目

	cd bar

## 增加Flyway依赖
编辑当前目录下的`pom.xml`，增加Flyway和MySQL的依赖

	<project ...>
	    ...
	    <dependencies>
	        <dependency>
	            <groupId>com.googlecode.flyway</groupId>
	            <artifactId>flyway-core</artifactId>
	            <version>2.3</version>
	        </dependency>
	        <dependency>
	            <groupId>mysql</groupId>
	            <artifactId>mysql-connector-java</artifactId>
	            <version>5.1.6</version>
	        </dependency>
	        ...
	    </dependencies>
	    ...
	</project>

## 整合Flyway
现在我们可以将Flyway的代码放入项目中，并配置数据库，例如增加到默认生成的：`src/main/java/foobar/App.java`

	package foobar;

	import com.googlecode.flyway.core.Flyway;

	public class App {
	    public static void main(String[] args) {
	        // 创建Flyway实例
	        Flyway flyway = new Flyway();

	        // 设置数据库
	        flyway.setDataSource("jdbc:mysql://localhost:3306/foobar", "user", "pass");

	        // 开始迁移
	        flyway.migrate();
	    }
	}

## 让我们创建第一个数据迁移吧
创建数据迁移目录`src/main/resources/db/migration`，执行命令

`mkdir -p src/main/resources/db/migration`

创建我们的第一个数据迁移`src/main/resources/db/migration/V1__Create_person_table.sql`

	CRETE TABLE person (
	    id INT,
	    name VARCHAR(100)
	);

## 执行程序
执行`App.java`（也可以直接在IDE中执行main方法）

`mvn package exec:java -Dexec.mainClass=foobar.App -Dmaven.test.skip=true`[^1]

如果你成功了，应该会得到如下信息

	INFO: Creating Metadata table: `foobar`.`schema_version`
	Feb 27, 2014 12:20:18 AM com.googlecode.flyway.core.command.DbMigrate migrate
	INFO: Current version of schema `foobar`: << Empty Schema >>
	Feb 27, 2014 12:20:18 AM com.googlecode.flyway.core.command.DbMigrate applyMigration
	INFO: Migrating schema `foobar` to version 1
	Feb 27, 2014 12:20:18 AM com.googlecode.flyway.core.command.DbMigrate logSummary
	INFO: Successfully applied 1 migration to schema `foobar` (execution time 00:00.194s).

## 持续增加数据迁移吧
假如我们现在需要增加第二个数据迁移，命名为：`src/main/resources/db/migration/V2__Add_people.sql`

	INSERT INTO person (id, name) VALUES (1, 'Axel');
	INSERT INTO person (id, name) VALUES (2, 'Mr. Foo');
	INSERT INTO person (id, name) VALUES (3, 'Ms. Bar');

执行命令

`mvn package exec:java -Dexec.mainClass=foobar.App -Dmaven.test.skip=true`

输出如下

	Feb 27, 2014 12:25:00 AM com.googlecode.flyway.core.command.DbMigrate migrate
	INFO: Current version of schema `foobar`: 1
	Feb 27, 2014 12:25:00 AM com.googlecode.flyway.core.command.DbMigrate applyMigration
	INFO: Migrating schema `foobar` to version 2
	Feb 27, 2014 12:25:00 AM com.googlecode.flyway.core.command.DbMigrate logSummary
	INFO: Successfully applied 1 migration to schema `foobar` (execution time 00:00.047s).

##总结
通过Flyway让我们能很方便的管理数据库文件，并进行版本控制。[文档地址](http://flywaydb.org/documentation/api.html)

[^1]: `-Dmaven.test.skip=true`跳过Test