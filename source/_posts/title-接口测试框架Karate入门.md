title: 接口测试框架Karate入门
date: 2019-11-23 23:37:55
tags: api-test, test, bdd
---

# 接口测试框架Karate入门

![](https://image.1programmer.com/2019-12-24-15772018260521.jpg)


有效的接口测试可以为我们的接口质量保驾护航。Karate的中文翻译是空手道，是一款接口测试框架，BDD类型，非程序员也能上手。

## 你能学到什么

你可以通过这篇教程学会如何通过jar包方式使用Karate进行接口测试

## 你需要什么

- 大约10分钟
- JDK 1.8+

## 环境准备

本篇教程通过命令行执行jar文件的方式来进行接口测试编写。

首先创建工作目录，进入这个目录

```
$ mkdir KarateTest
$ cd KarateTest
```
下载执行文件。

通过[https://dl.bintray.com/ptrthomas/karate/](https://dl.bintray.com/ptrthomas/karate/)下载最新jar包，下载后保存到项目根目录中，现在我们的项目有了第一个文件，结构如下：

```
$ tree
.
└── karate-0.9.4.jar

0 directories, 1 file
```

编写第一个接口用例文件`HttpbinGet.feature`，内容如下

```
Feature: Httbin.org get method test

   Scenario: Get should be ok
    Given url 'http://httpbin.org/get'
    When method get
    Then status 200
```

添加完用例后目录结构为

```
$ tree
.
├── HttpbinGet.feature
└── karate-0.9.4.jar

0 directories, 2 files
```

执行用例

```
$ java -jar karate-0.9.4.jar HttpbinGet.feature
23:07:19.174 [main] INFO  com.intuit.karate.Main - Karate version: 0.9.4
23:07:19.617 [ForkJoinPool-1-worker-1] WARN  com.intuit.karate - skipping bootstrap configuration: could not find or read file: classpath:karate-config.js
23:07:19.843 [ForkJoinPool-1-worker-1] DEBUG com.intuit.karate - request:
1 > GET http://httpbin.org/get
1 > Accept-Encoding: gzip,deflate
1 > Connection: Keep-Alive
1 > Host: httpbin.org
1 > User-Agent: Apache-HttpClient/4.5.5 (Java/1.8.0_231)

23:07:20.634 [ForkJoinPool-1-worker-1] DEBUG com.intuit.karate - response time in milliseconds: 788.74
1 < 200
1 < Access-Control-Allow-Credentials: true
1 < Access-Control-Allow-Origin: *
1 < Connection: keep-alive
1 < Content-Type: application/json
1 < Date: Sat, 23 Nov 2019 15:07:20 GMT
1 < Referrer-Policy: no-referrer-when-downgrade
1 < Server: nginx
1 < X-Content-Type-Options: nosniff
1 < X-Frame-Options: DENY
1 < X-XSS-Protection: 1; mode=block
{
  "args": {},
  "headers": {
    "Accept-Encoding": "gzip,deflate",
    "Host": "httpbin.org",
    "User-Agent": "Apache-HttpClient/4.5.5 (Java/1.8.0_231)"
  },
  "origin": "111.192.170.27, 111.192.170.27",
  "url": "https://httpbin.org/get"
}


23:07:20.684 [pool-1-thread-1] INFO  com.intuit.karate.Runner - <<pass>> feature 1 of 1: HttpbinGet.feature
---------------------------------------------------------
feature: HttpbinGet.feature
report: target/surefire-reports/HttpbinGet.json
scenarios:  1 | passed:  1 | failed:  0 | time: 1.0301
---------------------------------------------------------
Karate version: 0.9.4
======================================================
elapsed:   1.41 | threads:    1 | thread time: 1.03
features:     1 | ignored:    0 | efficiency: 0.73
scenarios:    1 | passed:     1 | failed: 0
======================================================
```
可视化报告地址：target/cucumber-html-reports/overview-features.html

![](https://image.1programmer.com/2019-12-24-15772018371845.jpg)



