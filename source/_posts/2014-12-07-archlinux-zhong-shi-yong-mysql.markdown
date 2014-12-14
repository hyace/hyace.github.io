---
layout: post
title: "Archlinux 中使用Mysql"
date: 2014-12-07 09:51:44 +0800
comments: true
categories: 
---

Archlinux中的官方源用的是MariaDB，现在随着人们对Oracle把控Mysql的不信任，更多人开始用MariaDB。

###设置
　　安装很简单，用`pacman`就行，其中的初始设置步骤如下：

	# systemctl start mysqld  
	# mysql_secure_installation  
	# systemctl restart mysqld  

　　其中的第二步可以设置或修改密码。如果还需要后续的修改可以在`/etc/mysql/my.cnf`中配置，比如添加`skip-networking`可以实现禁止网络访问。

###设置远程

　　在上步设置中设置了允许远程连接，但是账户还需配置，先进入MariaDB，执行命令：

    MariaDB [(none)]> grant all privileges on *.* to 'root'@'192.168.125.%' identified by 'q' with grant option;

　　其中root是远程连接用的账户名，q为密码，'%'表示通配。

　　需要的话可以设置mysql守护进程开机启动：`sudo systemctl enable mysqld`