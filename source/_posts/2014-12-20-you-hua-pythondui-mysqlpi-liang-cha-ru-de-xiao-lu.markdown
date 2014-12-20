---
layout: post
title: "优化Python对MySQL批量插入的效率"
date: 2014-12-20 13:48:01 +0800
comments: true
categories: 
---


之前测试cobar的效率，因为cobar不支持存储过程，所以需要应用程序插入数据，jdbc不灵活，用Python的MySQLdb模块可以实现。

　　开始测试的时候用单条insert语句循环n次，最后commit，结果慢的要死，插一万条用了两分钟，十万条我去吃了个饭回来还在插。十万条用存储过程插单库也用了50多秒。

　　从[UC的这篇文章](http://tech.uc.cn/?p=634)学习了一些SQL优化的知识。

　　主要有三条：
1、insert的时候尽量多条一起插，不要单条插。这样可以减少日志量，降低日志刷盘数据量和频率，效率提高很多。

这是文章提供的测试对比：

![](http://images.cnitblog.com/blog/316027/201412/191419430163905.jpg)


2、在事务中进行插入。也就是每次部分commit，否则每条insert commit 创建事务的消耗也是不小的。

![](http://images.cnitblog.com/blog/316027/201412/191419553293001.jpg)


3、 数据插入的时候保持有序。比如Innodb用的是B+树索引，对B+树的插入如果是在索引中间就会需要树节点分裂合并，这也会有一定的消耗。

![](http://images.cnitblog.com/blog/316027/201412/191420095488711.jpg)


几种方法综合起来的测试数据如下：

![](http://images.cnitblog.com/blog/316027/201412/191420251412408.jpg)


　　可以看到1000万以下数据的优化效果是明显的，但是单合并数据+事务的方式在1000万以上性能会有急剧下降，因为此时已经达到了innodb_buffer的上限，随机插入每次需要大量的磁盘操作，性能下降明显。而有序插入在1000万以上时也表现稳定，因为索引定位方便，不需要频繁对磁盘读写，维持较高性能。

据此修改了Python程序：

     import MySQLdb

     db=MySQLdb.connect(host='127.0.0.1', user='test', passwd='test', port=8066)

     cur=db.cursor()

     cur.execute('use dbtest')

     cur.execute('truncate table tb5')

     for t in range(0,100):

         sql = 'insert into tb5 (id, val) values '
 
         for i in range(1,100000):
              	sql += ' ('+`t*100000+i`+', "tb5EXTRA"),'
         sql += ' ('+`t`+'00000, "tb5EXTRA")'
 
         cur.execute(sql)
 
         db.commit()
 
     cur.close()
 
     db.close()

共插入了1000万条数据，sublime显示用了333.3s，比之前插入10万条都减少了很多，测试插10万条只需要3s。

![](http://images.cnitblog.com/blog/316027/201412/191420588766948.png)


15个分库每个60十多万条。

* 　　过程中有个问题，开始我的sql变量是连接所有插入条目的，但是16节点的cobar在输100万条的时候就连接断开了，而单库直接10条也插不进去，显示mysql连接断开。

![](http://images.cnitblog.com/blog/316027/201412/191421204694678.png)


　　这个是sql语句长度限制的问题，在mysql的配置文件中有一个`max_allowed_packet = 1M`，10万条插入语句已经超过这个限额了，100万条分给16个cobar节点也超过了，所以可以把这个参数调大，或者代码里分段执行sql。

*  　　还有一个问题是事务大小的问题，`innodb_log_buffer_size`参数决定这个，超限的话数据会写入磁盘，从而导致效率下滑，所以事务提交也不能攒得太大。