---
layout: post
title: "CentOS下安装mysql及配置Tomcat+Nginx"
description: ""
category: 工作生涯
tags: [Linux]
---
![Linux](http://www.mojiaqin.cn/images/2015/nginx.jpg)


接上，这回开始安装数据库以及web服务器了，Apache和Nginx都不熟练，现在nginx用于反向代理非常厉害，同时对于处理并发的能力也很强，所以既然都是新学，就选择使用Nginx，首先安装Mysql和Nginx，之后配置Nginx和Tomcat的代理，用于解析Java web项目。  


### 一、Mysql 安装

直接通过yum形式安装MYSQL，无需独立下载

先查看系统中有没有安装好的mysql

	rpm -qa | grep mysql

一般都是会有的，有则进行卸载

	rpm -e mysql 或 rpm -e --nodeps mysql

可以通过yum形式安装的MySQL版本

	yum list | grep mysql

我的是显示了下面的版本信息：  
![mysql](http://www.mojiaqin.cn/images/2015/0324/mysql.png)

一般需要安装 mysql、mysql server和mysql devel三个版本  
通过如下命令安装

	yum install -y mysql mysq-server mysql-devel

安装成功后通过

	rpm -qi mysql -server

查看mysql server的版本

通常安装好的都不是最新版哦。  
可以通过下面的命令

	service mysqlld start

来启动mysql服务

想要查看是否是开机启动的服务，通过下面命令实现

	chkconfig --list | grep mysqld

如果不是开机启动，则使用如下命令设置开机启动

	chkconfig mysqld on

为mysql的默认root用户设置密码为'admin'

	mysqladmin -u root password 'admin'

通过如下命令登录mysql

	mysql -u root -p

mysql数据库文件安装地址

	/var/lib/mysql

数据库日志地址

	/var/log

数据库监听端口查看

	netstat -anp



### 二、安装Nginx

CentOS6.6操作系统，直接通过如下命令

	yum install nginx

来安装是不行的，要先处理安装源，下面是安装完整流程，十分简单：  
1、先执行源的处理：

	rpm -ivh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm

2，查看yum安装的nginx信息

	yum info nginx

3，安装

	yum install -y nginx

4，启动服务

	services nginx start

5，停止服务

	services nginx stop

### 三、配置Nginx和Tomcat

1，进入配置文件目录  
cd /etc/nginx

2，编辑文件  
vi nginx.conf


3，配置信息：

	
	//nginx用户  
	user  nginx;
	
	//系统处理器数目
	worker_processes  2;
	
	//错误信息存放地址，默认
	error_log  /var/log/nginx/error.log warn;
	
	
	
	
	events {
	    use epoll;
	    worker_connections  1024;
	}

	
	http {
	    include       /etc/nginx/mime.types;
	    default_type  application/octet-stream;
	
	    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
	                      '$status $body_bytes_sent "$http_referer" '
	                      '"$http_user_agent" "$http_x_forwarded_for"';

	access_log  /var/log/nginx/access.log main;

    sendfile        on;

    keepalive_timeout  65;

    upstream backend{
      server localhost:8080;
      ip_hash;
    }

	server{
       listen 80;
       server_name localhost;

        root /usr/java/apache-tomcat-7.0.59/webapps/ROOT;

        location / {
                index index.html index.htm index.shtml index.jsp;
               proxy_redirect off;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                client_body_buffer_size 128k;
                client_max_body_size 10m;

                proxy_connect_timeout 90;
                proxy_send_timeout 30;
                proxy_read_timeout 30;
                proxy_pass http://backend;

        }

        location ~\.(html|shtml|jsp|jspx|do|action)?$
        {
                proxy_set_header Host $host;
                proxy_set_header X-FOrwarded-FOr $remote_addr;
                proxy_pass http://backend;

        }

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)?$
        {
                expires 30d;
        }
        location ~ .*\.(js|css)?$
        {
                expires 1h;
        }
	}


4，查看配置文件是否正确

	nginx -t

如果有问题对应进行修改既可，不断查看直到出现如下信息，表示没有问题了  
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok  
nginx: configuration file /etc/nginx/nginx.conf test is successful



配置好之后，输入

	service nginx start 

启动nginx，浏览器输入localhost，如果调到了tomcat原有的默认页面说明已经配置成功了。



### 遇到问题及解决
出现502 Bad Gateway 错误  
搜了一下无头绪，查看/var/log/nginx下的error.log文件，看到了如下信息提示：
 *1 connect() to [::1]:8080 failed (13: Permission denied) while connecting to upstream, client: 127.0.0.1,  

通过搜索这个信息终于找到了解决方法：  
这是SELinux安全子系统造成的问题，解决办法如下：  
1 临时方法 – 设置系统参数  
使用命令setenforce 0  
附：  
setenforce 1 设置SELinux 成为enforcing模式  
setenforce 0 设置SELinux 成为permissive模式  

2 永久方法 – 需要重启  
vi /etc/selinux/config  
设置SELINUX=disabled ，重启服务器。  
我是通过第二个方式解决了，将原来的SELINUX=enforcing该为SELINUX=disabled.  


重启完成后重新启动nginx和tomcat，输入localhost,发现已经能跳转到tomcat的主页面了，说明已经配置成功。


### 完工
基本上配置了整个CentOS下的Java Web开发环境，接下来就是开发工作了。

PS:Vim编辑器好像还不错的样子，咩哈哈！

