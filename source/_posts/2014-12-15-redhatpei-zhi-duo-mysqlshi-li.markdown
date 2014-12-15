---
layout: post
title: "Redhat配置多Mysql实例"
date: 2014-12-15 20:21:02 +0800
comments: true
categories: 
---

申请了两台Redhat虚拟机做cobar的实验，空空如也什么都得装。原来在3个Arch上实现了3个datanode，想在Redhat上每个3个总共配6个Mysql实例。

###复制多个实例

在`/var/lib`目录下找到Mysql目录，将其复制两份，以端口号作后缀以区分：

    cp -r /var/lib/mysql /var/lib/mysql3307
    cp -r /var/lib/mysql /var/lib/mysql3308
    chown -R mysql mysql3307
    chown -R mysql mysql3308

###配置参数
Mysql的配置文件是`/etc/my.cnf`,但是默认是没有的，在`/usr/share/mysql/`目录下有5个.cnf后缀的文件，可以根据自己机器内存大小选择一个文件复制为`/etc/my.cnf`:

    /usr/share/mysql/my-small.cnf             64M内存
    /usr/share/mysql/my-medium.cnf            128M内存
    /usr/share/mysql/my-large.cnf             512M内存
    /usr/share/mysql/my-huge.cnf              1-2G内存
    /usr/share/mysql/my-innodb-heavy-4G.cnf   4G内存

我将my-huge.cnf复制为my.cnf。

酌情添加以下配置：  

	[mysqld_multi]
	mysqld          = /usr/bin/mysqld_safe    
	mysqladmin      = /usr/bin/mysqladmin 

	[mysqld1]                                          
	port            = 3306                      
	socket          = /var/lib/mysql/mysql.sock   
	pid-file        = /var/lib/mysql/mysql.pid         
	datadir         = /var/lib/mysql 
	user            = mysql               
	skip-external-locking                              
	key_buffer_size = 384M                                
	max_allowed_packet = 1M                                    
	table_open_cache = 512                               
	sort_buffer_size = 2M                                 
	read_buffer_size = 2M                                
	read_rnd_buffer_size = 8M    
	myisam_sort_buffer_size = 64M                        
	thread_cache_size = 8                              
	query_cache_size = 32M                               
	server-id       = 1 

	[mysqld2]                                          
	port            = 3307                      
	socket          = /var/lib/mysql3307/mysql.sock   
	pid-file        = /var/lib/mysql3307/mysql.pid         
	datadir         = /var/lib/mysql3307 
	user            = mysql                
	skip-external-locking                              
	key_buffer_size = 384M                                
	max_allowed_packet = 1M                                    
	table_open_cache = 512                               
	sort_buffer_size = 2M                                 
	read_buffer_size = 2M                                
	read_rnd_buffer_size = 8M    
	myisam_sort_buffer_size = 64M                        
	thread_cache_size = 8                              
	query_cache_size = 32M                               
	server-id       = 1

	[mysqld3]                                          
	port            = 3308                      
	socket          = /var/lib/mysql3308/mysql.sock   
	pid-file        = /var/lib/mysql3308/mysql.pid         
	datadir         = /var/lib/mysql3308  
	user            = mysql                
	skip-external-locking                              
	key_buffer_size = 384M                                
	max_allowed_packet = 1M                                    
	table_open_cache = 512                               
	sort_buffer_size = 2M                                 
	read_buffer_size = 2M                                
	read_rnd_buffer_size = 8M    
	myisam_sort_buffer_size = 64M                        
	thread_cache_size = 8                              
	query_cache_size = 32M                               
	server-id       = 1

要注意其中的mysqld、mysqladmin后配置的目录参数视具体情况而定。

###配置完将3个Mysql都启动：

    mysqld_multi start

![](http://images.cnitblog.com/blog/316027/201412/041155359205638.png)

成功~

####如果不能正常远程访问可以查看下防火墙配置和Mysql是否配置远程访问。