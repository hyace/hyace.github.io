---
layout: post
title: "阿里数据库中间件Cobar环境搭建"
date: 2014-12-15 20:13:39 +0800
comments: true
categories: 
---

装了CentOS7，结果死活连不上网，一直提示没有可用的网络设备，没工夫处理，以后再说，先用之前装好的CentOS6.4，Mysql安装包又下不下来，只好翻出了之前玩Storm装的Archlinux，总共有Alpha、Beta、Delta三台虚机，里头安装了JDK、MariaDB等等。

###准备

测试中有一个schema，两张表tb1、tb2，其中tb1在一个库，tb2分为两个库。
![](http://images.cnitblog.com/blog/316027/201412/022342064987985.jpg)

几个Cobar的[release][1]包都先下载好。

数据准备，三个数据库表都建好：  

    #创建dbtest1
    drop database if exists dbtest1;
    create database dbtest1;
    use dbtest1;
    #在dbtest1上创建tb1
    create table tb1(
    id int not null,
    gmt datetime);
    #创建dbtest2
    drop database if exists dbtest2;
    create database dbtest2;
    use dbtest2;
    #在dbtest2上创建tb2
    create table tb2(
    id int not null,
    val varchar(256));
    #创建dbtest3
    drop database if exists dbtest3;
    create database dbtest3;
    use dbtest3;
    #在dbtest3上创建tb2
    create table tb2(
    id int not null,
    val varchar(256));

####注意系统的JAVA_HOME要配置好

###配置

Cober的server目录下的conf中的是配置文件

schema.xml中需要修改三个数据库的ip和端口，账号和密码。

server.xml中需要修改Cobar的账号和密码。

###启动

通过运行bin目录下的startup.bat来启动Cobar服务，

第一遍没起来，打开startup.bat,其中的`APP_VERSION`版本和release中的server版本不匹配，log4j的版本也不匹配，修改之后才能正确运行。

logs目录下有Cobar运行的log文件，`stdout.log`是服务运行的日志，正确运行时是这样：  

    23:13:00,876 INFO  ===============================================
    23:13:00,876 INFO  Cobar is ready to startup ...
    23:13:00,876 INFO  Startup processors ...
    23:13:00,969 INFO  Startup connector ...
    23:13:00,969 INFO  Initialize dataNodes ...
    23:13:01,047 INFO  dnTest1:0 init success
    23:13:01,063 INFO  dnTest3:0 init success
    23:13:01,063 INFO  dnTest2:0 init success
    23:13:01,063 INFO  CobarManager is started and listening on 9066
    23:13:01,079 INFO  CobarServer is started and listening on 8066
    23:13:01,079 INFO  ===============================================


###测试
访问Cobar可以和访问Mysql一样的方式：

    >mysql -utest -ptest -P8066 -Ddbtest

![](http://images.cnitblog.com/blog/316027/201412/030015092332882.jpg)

三个库看起来是在一个库，成功~

###再实验在Arch下跑Cobar

　　大部分步骤和Windows下一样，不同的是startup的启动，Linux下运行的是startup.sh。
　　先是报错：`JAVA_HOME environment variable is not set.`，但是我的JAVA_HOME已经设定了，虽然用的是openJDK，再查看到有人说JAVA_HOME不应该指向jre，但是openJDK下的java就是软链到jre的，所以把启动脚本中的`if [ ! -e "$JAVA_HOME/bin/java" ] `注释了。

　　运行后又失败，没有报错，查看`../logs/console.log`：

      1 OpenJDK Server VM warning: INFO: os::commit_memory(0x7b000000, 805306368, 0) failed; error='Cannot allocate memory' (errno=12)
      2 #
      3 # There is insufficient memory for the Java Runtime Environment to continue.
      4 # Native memory allocation (malloc) failed to allocate 805306368 bytes for committing reserved memory.
      5 # An error report file with more information is saved as:
      6 # /home/software/cobar-server-1.2.7/hs_err_pid623.log

原来是虚拟机内存不够分配，因为装Arch的时候为了多运行几个，每个只分配了256M内存，而startup.sh中配置如下：

    JAVA_OPTS="-server -Xms1024m -Xmx1024m -Xmn256m -Xss256k"

堆内存分了1G，新生代占了整个Arch内存大小，线程栈256k。将参数修改后可以运行。

[1]: https://github.com/alibaba/cobar/wiki