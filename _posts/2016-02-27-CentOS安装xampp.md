---
layout: post
title: "CentOS终端安装xampp及遇到问题解决"
description: ""
category: IT在路上
tags: ""
---

记录在远程服务器上部署xampp并进行管理的过程。

### 一、通过putty连接到远程服务器

### 二、下载xampp

<code>
wget http://sourceforge.net/projects/xampp/files/XAMPP%20Linux/1.8.3/xampp-linux-x64-1.8.3-1-installer.run/download
</code>

### 三、下载完以后，给该文件添加执行权限

<code>
chmod a+x xampp-linux-x64-1.8.3-a-installer.run
</code>

### 四、安装xampp过程

<code>
[root@job2016 src]# ./xampp-linux-x64-1.8.3-1-installer.run
</code>

### 五、安装后的文件在 

<code>
/opt/lamp
</code>

### 六、启动xampp

<pre><code>
[root@job2016 lampp]# /opt/lampp/lampp start
Starting XAMPP for Linux 1.8.3-1...
XAMPP: Starting Apache...fail.
XAMPP:  Another web server is already running.
XAMPP: Starting MySQL...ok.
XAMPP: Starting ProFTPD...fail.
XAMPP:  Another FTP daemon is already running.
</code></pre>

### 七、停止xampp

<pre><code>
[root@job2016 lampp]# /opt/lampp/lampp stop
Stopping XAMPP for Linux 1.8.3-1...
XAMPP: Stopping Apache...not running.
XAMPP: Stopping MySQL...ok.
XAMPP: Stopping ProFTPD...not running.
</code></pre>

### 八、添加开机启动

Ln命令：

<code>
	#ln –s /opt/lampp/xampp /etc/rc.d/init.d/xampp
</code>

### 九、如果执行完上面这条还不能开机自动启动，再执行下面3条语句

<code>
	#chkconfig --add xampp  
	#chkconfig --list | grep xampp  
	#chkconfig --level 3 xampp on
</code>

### 十、卸载xampp

<code>
	#/opt/lampp/xampp stop  
	#rm -rf /opt/lampp
</code>


### 部署可能出现问题

#### 问题1：访问网址时如果出现下面的画面


![forbidden](http://www.mojiaqin.cn/images/2016/0227/forbidden.jpg)

解决：先找到httpd-xampp.conf （find /opt/ -name httpd-xampp.conf ） 
如下图，注释一行，添加一行，重启即可 
![conf](http://www.mojiaqin.cn/images/2016/0227/conf.jpg)


#### 问题2：无法使用mysql命令

root@DB-02 ~]# mysql -u root  
-bash: mysql: command not found

解决:这是由于系统默认会查找/usr/bin下的命令，如果这个命令不在这个目录下，就会找不到命令，我们需要做的就是映射一个链接到/usr/bin目录下，相当于建立一个链接文件。
首先得知道mysql命令或mysqladmin命令的完整路径，比如mysql的路径是：/opt/lampp/bin/mysql，我们则可以这样执行命令：

<code>
	#ln -s /opt/lampp/bin/mysql /usr/bin
</code>


#### 问题3：修改mysql的root密码，因为默认为空 

解决：先找到mysql (find /opt/ -name mysql)如图 
 ![mysql](http://www.mojiaqin.cn/images/2016/0227/mysql.jpg)
这里看到有bin/mysql，执行命令/opt/lampp/bin/mysql -uroot -p不需要密码进入数据库。 
修改root密码：

<code>
		set password for 'root'@'localhost'=password('passwd');
</code>

输入quit退出，重新执行命令/opt/lampp/bin/mysql -uroot -p，此时需要输入密码才能进入数据库


#### 问题4：无法访问phpmyadmin，返回403错误

需要修改/opt/lampp/etc/extra下的文件httpd-xmapp.conf，找到如下内容：

`

	# since XAMPP 1.4.3
	<Directory "/opt/lampp/phpmyadmin">  
	AllowOverride AuthConfig Limit  
	Order allow,deny  
	Allow from all     
	</Directory>  
`

增加一句Require all granted 
变为：

`

		# since XAMPP 1.4.3
		<Directory "/opt/lampp/phpmyadmin">
			AllowOverride AuthConfig Limit
			Order allow,deny
			Allow from all
			Require all granted 
		</Directory>
`  

同时，找到：

`

		<LocationMatch "^/(?i:(?:xampp|security|licenses|phpmyadmin|webalizer|server-status|server-info))">
				Require local
			 ErrorDocument 403 /error/XAMPP_FORBIDDEN.html.var
		</LocationMatch>

`

将Require local注释掉，就可以从本地以外的地方访问了，注释如下:

`

		<LocationMatch "^/(?i:(?:xampp|security|licenses|phpmyadmin|webalizer|server-status|server-info))">
			 #Require local
			 ErrorDocument 403 /error/XAMPP_FORBIDDEN.html.var
		</LocationMatch>
`

#### 问题5：修改了mysql的root登陆密码后phpmyadmin不能登陆 


修改了root密码后不能登录phpmyadmin
 ![phpmyadmin](http://www.mojiaqin.cn/images/2016/0227/phpmyadmin.png)

解决：找到config.inc.php文件（find /opt/ -name config.inc.php）
 ![configinc](http://www.mojiaqin.cn/images/2016/0227/configinc.jpg)
 
然后vim /opt/lampp/phpmyadmin/config.inc.php如图修改密码   
 ![configinc2](http://www.mojiaqin.cn/images/2016/0227/configinc2.jpg)
 
之后就可以打开phpmyadmin了


#### 问题6：phpmyadmin管理数据库时出现问题  

总是提示一些表不存在的问题，如“MySQL error - #1932 - Table 'phpmyadmin.pma user config' doesn't exist in engine

解决：这是由于数据库xampp版本的问题导致初始化时phpmyadmin中的表格与配置文件有出入，需要根据以下几个步骤进行修改：
1、删除var/mysql/phpmyadmin/下的以 pma__开头的所有文件
2、删除phpmyadmin库中的所有表格，通过shell脚本删除数据库中所有表的语句
 
<code>
mysqldump -u root -ppassword  dbname|grep ^DROP|mysql -u root -ppassword dbname
</code>

 3、连接数据库，将 /opt/lampp/phpmyadmin/sql/create_tables.sql下的表导入phpmyadmin，导入语句
 
<code>
source /opt/lampp/phpmyadmin/sql/create_tables.sql;
</code>

4、将/opt/lampp/phpmyadmin/config.inc.php文件中下面的内容

 <pre><code>
 $cfg['Servers'][$i]['bookmark'] = 'pma__bookmark';
$cfg['Servers'][$i]['relation'] = 'pma__relation';
$cfg['Servers'][$i]['table_info'] = 'pma__table_info';
$cfg['Servers'][$i]['table_coords'] = 'pma__table_coords';
$cfg['Servers'][$i]['pdf_pages'] = 'pma__pdf_pages';
$cfg['Servers'][$i]['column_info'] = 'pma__column_info';
$cfg['Servers'][$i]['table_uiprefs'] = 'pma__history';
$cfg['Servers'][$i]['table_uiprefs'] = 'pma__table_uiprefs';
$cfg['Servers'][$i]['tracking'] = 'pma__tracking';
$cfg['Servers'][$i]['userconfig'] = 'pma__userconfig';
$cfg['Servers'][$i]['recent'] = 'pma__recent';
$cfg['Servers'][$i]['users'] = 'pma__users';
$cfg['Servers'][$i]['usergroups'] = 'pma__usergroups';
$cfg['Servers'][$i]['navigationhiding'] = 'pma__navigationhiding';
$cfg['Servers'][$i]['savedsearches'] = 'pma__savedsearches';
$cfg['Servers'][$i]['central_columns'] = 'pma__central_columns';
$cfg['Servers'][$i]['designer_coords'] = 'pma__designer_coords';
$cfg['Servers'][$i]['designer_settings'] = 'pma__designer_settings';
$cfg['Servers'][$i]['export_templates'] = 'pma__export_templates';
$cfg['Servers'][$i]['favorite'] = 'pma__favorite';
</code></pre>
 
替换成

 <pre><code>
 $cfg['Servers'][$i]['pma__bookmark'] = 'pma__bookmark';
 $cfg['Servers'][$i]['pma__relation'] = 'pma__relation';
 $cfg['Servers'][$i]['pma__table_info'] = 'pma__table_info';
 $cfg['Servers'][$i]['pma__table_coords'] = 'pma__table_coords';
 $cfg['Servers'][$i]['pma__pdf_pages'] = 'pma__pdf_pages';
 $cfg['Servers'][$i]['pma__column_info'] = 'pma__column_info';
 $cfg['Servers'][$i]['pma__table_uiprefs'] = 'pma__history';
 $cfg['Servers'][$i]['pma__table_uiprefs'] = 'pma__table_uiprefs';
 $cfg['Servers'][$i]['pma__tracking'] = 'pma__tracking';
 $cfg['Servers'][$i]['pma__userconfig'] = 'pma__userconfig';
 $cfg['Servers'][$i]['pma__recent'] = 'pma__recent';
 $cfg['Servers'][$i]['pma__users'] = 'pma__users';
 $cfg['Servers'][$i]['pma__usergroups'] = 'pma__usergroups';
 $cfg['Servers'][$i]['pma__navigationhiding'] = 'pma__navigationhiding';
 $cfg['Servers'][$i]['pma__savedsearches'] = 'pma__savedsearches';
 $cfg['Servers'][$i]['pma__central_columns'] = 'pma__central_columns';
 $cfg['Servers'][$i]['pma__designer_coords'] = 'pma__designer_coords';
 $cfg['Servers'][$i]['pma__designer_settings'] = 'pma__designer_settings';
 $cfg['Servers'][$i]['pma__export_templates'] = 'pma__export_templates';
 $cfg['Servers'][$i]['pma__favorite'] = 'pma__favorite';
</code></pre>
重启xampp即可。
