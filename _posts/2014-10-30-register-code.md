---
layout: post
title:  "手机短信验证"
date:   2014-10-30 10:50:00
categories: 有码
tags: Java Redis Coding
---
手机的普及程度在中国远高于邮箱，以手机作为密码找回也非常的简单和易用，尤其是手机应用更是可以直接读取用户的手机短信实现免输入就可以完成验证，所以现在的应用中越来越多的使用到了短信验证。

当用户注册成大概的流程如下：

1. 输入手机号，点击发送验证码
1. 用户接收到验证码后输入到验证框

用户要做的事情就是输入手机号、点击发送验证码按钮、输入验证码，就完成了手机绑定。

那么在短信验证中服务器做了什么呢？

1. 向服务器发送`Ajax`请求，把用户输入的手机号发送回服务器（用户点击发送验证码后）
1. 服务器验证手机号的合法性
1. 生成随机码、同时将随机码以手机号为键保存到服务器中,并设置过期时间
1. 发送短信给用户
1. 用户收到短信后会输入到验证框
1. 用户提交表单时验证随机码是否正确

## 向服务器发送Ajax请求

发送`Ajax`请求这步比较简单，直接直接用`Jquery`发送请求并取得结果，代码样例如下：

	$.ajax('/auth/verify/13888888888').done(function (code) {
    	alert(code);
	});

## 服务器验证手机号的合法性

这一步通常使用正则表达式验证是否是有效的11位的手机号：

	/1[3458]\d{9}/g
	
上面的规则是第一位必需是1，第二位必需在3、4、5、8中，后面带9位数字，这样就确认用户输入一定是11位的手机号了。这些验证码一般都是根据国情而定的，例如4G手机就开始启用`170`的号段，那这样的验证就不合适了，**解决办法：放松验证条件或与时俱进保持更新**

## 生成随机码、同时将随机码以手机号为键保存到服务器中,并设置过期时间

生成的随机码一般为了对用户友好可以直接生成四位的数字随机数，例如：1234等。我使用的是`Apache`的`commons.lang`包进行生成：

	RandomStringUtils.random(4, false, true);


将随机码以手机号为键保存到服务器中,并设置过期时间这个看个人喜好，推荐直接存入缓存服务中，**项目中的内存缓存除外，如`ehcahe`**。不推荐把随机码直接存到项目内存中，如果期间进行服务器部署会导致验证码全部失效。

我使用的方法是使用`Redis`进行保存

    String phone = "138123456789";
    Jedis jedis = new Jedis("localhost");

    String key = "verifies:" + phone;

    String verifyCode = RandomStringUtils.random(4, false, true);

    jedis.set(key, verifyCode);
    jedis.expire(key, 60 * 5); // 五分钟内有效

##发送短信给用户

如何在程序中发送短信给用户呢？我们的程序并没有发送短信到用户手机中的能力，通过需要接入第三方的短信服务提供商，通过调用第三方接口来实现发送短信的功能：

**首先要找到短信服务提供商，接入短信服务**

开发人员通过短信服务提供商提供的接口调通调用服务

发送验证码的过程如下：

1. 网站（手机）请求发送信息
2. 服务器向短信服务提供商通信，提交发送请求
3. 短信服务提供商通过运营商将信息发送到用户的手机中

##用户收到短信后会输入到验证框

以下就没有什么难度了，用户收到短信后会输入到验证框，这时可以等待用户提交表单后验证


## 需要注意的问题

1. 手机正则过期，不能配置最新的手机号码。保证正则一直可用
2. 生成的随机码不要直接在请求时返回到浏览器中，这样会导致非普通用户可以简单获取到验证码批量注册用户
3. 发短信的频率一定要做限制，服务提供商通常都会每条信息收取几分钱