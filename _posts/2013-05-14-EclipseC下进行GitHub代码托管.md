---
layout: post
title: "Eclipse C下进行GitHub代码托管"
description: ""
category: IT在路上
tags: [Github，代码同步]
---
##如何管理代码
如今的IT人士都习惯了多地点写代码的工作方式，因此代码的转移是个问题，不断的代码拷贝转移不是代码管理的上策。只有托管到版本控制服务上才是最好的管理代码的方式。因此我们需要选择一个代码托管的服务，毫无疑问，GitHub是现在最流行的服务之一。下面对eclipse下的GitHub代码托管的环境进行简单的搭建说明。

##创建代码仓库
在[GitHub](www.github.com)上创建一个代码仓库(repository)（当然，前提肯定是要有github账号的），比如我的代码仓库是“Clearning”。  
####本地SSH文件  
GitHub在进行代码同步时需要验证一个SSH key，如果没有设置这个的话会报无权限的错误，所以需要进行的设置，本地ssh文件一般在安装github客户端时就已经自动生成了，默认路径是：*C:\Users\username\ .ssh*，其中的*username*是你的机器的用户。找到其中的*id_rsa.pub*文件。
####添加GitHub账户的SSH Key
需要将本地的SSH key添加到远端服务器的账号上，才能进行代码同步。进入Github官网，进行如下步骤设置：
*Account settings*->*SSH Keys*->*Add SSH key*,如下图的1,2,3,4所示  
![SSH Key](http://oldmo.github.io/images/0514/egit6.jpg)   
在Key的输入框里输入*id_rsa.pub*里面的内容，点击*Add Key*按钮即可成功添加SSH认证。
##下载EGit
EGit是流行的分布式版本控制工具，也是eclipse的一款插件。在eclipse上进行代码管理需要安装这个插件。安装方法有两种：
#####方法一  
*Help*->*Install New Software*->*Add*按钮，在弹出的对话框中，Name可以随便起个名字，Location填如下的地址    `http://download.eclipse.org/egit/updates` 确定后会出现下面的两个选项    
![Egit](http://oldmo.github.io/images/0514/egit.jpg)  
两个都选择进行安装就可以了。  
#####方法二
*Help*->*Eclipse Marketplace*,在*find*中搜索“git”，确定后选择第一个安装即可，如图  
![Egit](http://oldmo.github.io/images/0514/egit1.jpg)  

##在Eclipse中*Import*代码仓库
可以直接通过导入的方式将GitHub上的仓库以项目的形式导入到本地。
*File*->*Import*->*Git*->*Projects from Git*,如下图  
![导入git](http://oldmo.github.io/images/0514/egit2.jpg)  
进入后选择*URI*会出现下面的页面  
![URI](http://oldmo.github.io/images/0514/egit3.jpg)   
这时需要进入到www.github.com中登录你的用户，找到你创建的仓库（repository），复制如下图中红色的部分  
![SSH](http://oldmo.github.io/images/0514/egit4.jpg)  
这个地址就是从客户端访问你的远端服务器的地址，将其填入Eclipse的*URI*后，下面的部分空会自动填充好，差一个*Password*，如下图  
![URI](http://oldmo.github.io/images/0514/egit5.jpg)   
找到*id_rsa.pub*文件，将里面的所有内容复制到*Password*输入框，点击*Next*即可连接到代码服务器，选中*master*并继续*Next*，到了*Local Destination*这一步在*Directory*中选择你要将代码放到本地的路径，我的是选择了一个GitHub的专门的文件夹存放，如下  
![本地路径](http://oldmo.github.io/images/0514/egit7.jpg)  
在下一步可以这样选择，以普通项目形式导入：  
![导入形式](http://oldmo.github.io/images/0514/egit8.jpg)  
至此，代码仓库就导入成功了。可以在eclipse看到这个刚刚导入的代码仓库，项目下加了一个仓库的标志  
![代码仓库](http://oldmo.github.io/images/0514/egit9.jpg)  

##代码同步
在项目下创建一个Hello.c文件，未同步时该文件下会有'?'，通过如下方法同步代码：  
选中*Hello.c*->右键->*Team*->*Commit* 会出现一个提交的对话框，在*Commit message*中输入这次提交代码的说明  
![提交说明](http://oldmo.github.io/images/0514/egit10.jpg)  
点击*Commit and Push*进行提交和推送，这是会发现*Hello.c*下的'?'变成了一个仓库的形式，说明已经同步成功。  
整个项目和整个文件夹的同步也是如此进行。基本的功能这样子的够用了的，当然，团队的代码管理还是得进行其他方面的设置，以后碰到再续了。