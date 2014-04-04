---
layout: post
title: "Mysql忘记root密码解决方案"
description: ""
category: IT在路上-Mysql
tags: [mysql]
---

有时候很长时间没有用到自己的mysql数据库，很容易就把登录密码忘记了，想要重装，又怕之前有什么需要用到的数据库，又不好轻易重装，这个时候就想到要找回密码了，
这得用到下面的方法，不是找回密码，而是重置密码。具体步骤如下：  


　　1.如果打开了mysql，那么首先关闭它。

　　2.按CTRL+R进入DOS窗口，进入mysql\bin目录。

　　3.输入mysqld --skip-grant-tables回车，这个命令意思是：启动mysql时不启动grant-tables，授权表，这样就可以避过管理员登录

　　4.此时再开一个DOS窗口(因为刚才那个DOS窗口已经不能动了)，同样转到mysql\bin目录。

　　5.输入mysql回车，如果成功，将出现MySQL提示符 >

　　6.输入命令use mysql; 切换数据库。

　　6.改密码：> update user set password=password("yourpassword") where user="root"; 

　　7.刷新权限(必须的步骤)>flush privileges;

　　8.退出 > \q
　　
　大功告成！此方法参考了网上的解决方案。
