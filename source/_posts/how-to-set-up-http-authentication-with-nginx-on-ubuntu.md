title: 如何在Ubuntu上通过Nginx设置HTTP认证
date: 2015-09-28 18:03:43
tags:
- Ubuntu
- Nginx
- HTTP
---

## Apache Utils
我们需要htpassword来创建和生成加密的用户用于基础认证（Basic Authentication）。通过以下命令安装`apache2-utils`。

```
sudo apt-get install apache2-utils
```

## 创建用户名和密码

在Nginx托管的网站目录下生成一个`.htpasswd`文件。如下的命令可以创建文件同步增加用户和加密的密码到文件中

```
sudo htpasswd -c /etc/nginx/.htpasswd exampleuser
```
命令行为提示你输入密码
```
New password:
Re-type new password:
Adding password for user exampleuser
```

htpaswd的文件格式如下：
```
login:password
```
**注意：**htpasswd需要对nginx运行用户可访问

## 更新Nginx配置
在你的网站的Nginx配置文件增加如下两行：

```
auth_basic "Restricted";
auth_basic_user_file /etc/nginx/.htpasswd;
```
第二行是你的htpasswd文件位置。

举个例子，假如你的文件是`/etc/nginx/sites-available/website_nginx.conf`，通过vi或者其它编辑器打开该文件
```
sudo vi /etc/nginx/sites-available/website_nginx.conf
```

增加代码：

```
server {
  listen       portnumber;
  server_name  ip_address;
  location / {
      root   /var/www/mywebsite.com;
      index  index.html index.htm;
      auth_basic "Restricted";                                #For Basic Auth
      auth_basic_user_file /etc/nginx/.htpasswd;  #For Basic Auth
  }
}
```


## 刷新Nginx
为了使配置生效，需要刷新nginx配置，然后再访问
```
$ sudo /etc/init.d/nginx reload
* Reloading nginx configuration...  
```

原文：https://www.digitalocean.com/community/tutorials/how-to-set-up-http-authentication-with-nginx-on-ubuntu-12-10
