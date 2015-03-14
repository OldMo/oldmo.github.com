---
layout: post
title: "CentOS下配置JDK+Tomcat+Eclipse"
description: ""
category: 工作生涯
tags:Linux
---
![Linux](http://www.mojiaqin.cn/images/2015/linux.jpg)

---

工作需要，以后要开始转向Linux下的开发了，其实早就应该做的工作，终于可以正式的开始在Linux下做开发工作了，首先把开发环境搭好。  


###一、JDK 安装

#####1.1、下载JDK

下载地址:http://www.oracle.com/technetwork/cn/java/javase/downloads/jdk7-downloads-1880260-zhs.html 

根据linux的版本选择下载注:查看linux位数的命令是

	uname -a 
x86_64则说明是64位内核, 跑的是64位的系统；  
i386, i586说明是32位的内核, 跑的是32位的系统

可供下载的有下列形式的JDK，根据自己要求下载

	* jdk-7u75-linux-x64.tar.gz（64位）
	* jdk-7u75-linux-i586.tar.gz（32位）
	* jdk-7u75-linux-x64.rpm

#####1.2、安装JDK
tar.gz和rpm方式安装有些许不同： 
 
######（1）：tar.gz 方式安装JDK

//建立/usr/lib/jvm 目录

	sudo mkdir /usr/lib/jvm   

//切换到下载目录，输入命令解压

	sudo tar -xzf jdk-7u75-linux-x64.tar.gz

//将解压后的文件移到/usr/lib/jvm目录。命令为

	sudo mv jdk1.7.0_75 /usr/lib/jvm


######（2）：rpm 方式安装JDK

首先卸载系统自带的JDK

	rpm -qa|grep jdk    查看自带的jdk

	rpm -e --nodes  jdk...   卸载jdk

安装JDK
  
	chmod u+x jdk-7u75-linux-x64.rpm  赋执行权限

	rpm -ivh jdk-7u75-linux-x64.rpm  安装

####1.3、配置环境变量
打开系统启动配置文件

	vi /etc/profile

在文件末尾处加入如下代码段，将JDK启动信息进行配置

	export JAVA_HOME=/usr/lib/jvm/jdk1.7.0_75   
	export JRE_HOME=${JAVA_HOME}/jre
	export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
	export PATH=${JAVA_HOME}/bin:$PATH

保存退出 

	 :wq
回车

	Enter

使得配置文件生效  

	source /etc/profile


查看java版本

	java  -version

输出如下为:

	java version "1.7.0_75"
	Java(TM) SE Runtime Environment (build 1.7.0_75-b13)
	Java HotSpot(TM) 64-Bit Server VM (build 24.75-b04, mixed mode)


####1.4、编写测试

建立代码文件夹

	mkdir /home/admin/code  
创建测试文件
 
	vi  test.java  

在vim编辑器中输入如下测试代码：

	public class test{
	     public static void main(String[] args){
	          System.out.println("Hello World");
	     }
	}

<b>*注意类型和文件名需要保持一致，否则会报错 或者也可以不加 public</b>

编译java文件

	javac test.java  

执行文件

	java test  

输入结果

	Hello World

<b>JDK安装成功！</b>

###二、安装Tomcat7.0

1、下载地址 ![地址](http://tomcat.apache.org/download-70.cgi)
下载其中的Core部分下的tar.gz文件

解压缩

	tar -xzvf apache-tomcat-7.0.59.tar.gz  /usr/java

2、在/etc/profile下添加路径

	export CATALINA_HOME=/usr/java/apache-tomcat-7.0.59

3、使修改生效

	source /etc/profile  

4、执行启动

	./startup.sh

5、查看效果

浏览器中输入：http://localhost:8080


###三、安装Eclipse

由于安装环境主要是用于WEB开发，所以下载的是jee版的Eclipse

1、下载地址：![下载](http://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/luna/SR2/eclipse-jee-luna-SR2-linux-gtk-x86_64.tar.gz)

解压缩

	tar -xzvf eclipse-jee-luna-SR2-linux-gtk-x86_64.tar.gz  /usr/java

进入解压缩后的目录

	cd eclipse   

启动eclipse

	./eclipse    


安装Eclipse期间遇到一个问题    

<b style color="red"> Error: The Eclipse executable launcher was unable to locate its companion launcher jar</b>

根据网上提供的方式，重新解压缩一下可以解决这个问题，如果重新解压缩还不行，最好就重新下载一份，基本上是不会有这个问题的了。


###安装Nginx
待续