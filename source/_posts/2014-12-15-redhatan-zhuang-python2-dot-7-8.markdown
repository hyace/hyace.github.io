---
layout: post
title: "Redhat安装Python2.7.8"
date: 2014-12-15 20:27:13 +0800
comments: true
categories: 
---

想用Python操作Mysql，但是Redhat中的Python是2.4的，所以需要自己装下2.7

过程比较简单，下载源码包，解压。

运行：

    ./configure --prefix=/usr/local/python2.7 

然后编辑`Modules/Setup`文件，将如下几行取消注释：

	SSL=/usr/local/ssl
	_ssl _ssl.c \
	        -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
	        -L$(SSL)/lib -lssl -lcrypto
	...
	zlib zlibmodule.c -I$(prefix)/include -L$(exec_prefix)/lib -lz

之后运行：

    make all && make install && make clean && make distclean

之后将`/usr/local/python2.7`软链到`/usr/bin`下的python：

    ln -sf /usr/local/python2.7 /usr/bin/python

一般情况下这样就好了，但是过程中我遇到几个问题：

1、make的时候报错，回显以`error: openssl/rsa.h: No such file or directory`开头的一坨错误信息  

虽然我装了openssl，但是make的时候需要的是有头文件的版本，所以重新安开发版：

    yum install openssl-devel

同理安装了zlib-devel.

2、装好以后可以进入Python的交互界面，但是很多按键异常，backspace也失灵

这个是readline的问题，可以在安装python前先安装readline：

    yum install readline-devel

安装结束。

但是有些软件是依赖老版本的Python的，比如yum，可以打开`/usr/bin/yum`,将第一行的`#!/usr/bin/python`改为`#!/usr/bin/python2.4`.