---
layout: post
title: "Ubuntu下安装Sublime3的Markdown插件"
date: 2014-08-27 01:04:21 +0800
comments: true
categories :
---
***  
   开始打算用vim编辑markdown文件的，搜了下各种解决方案，也尝试了很久，  
但还是不行，比如vim-instant-markdown([git][1])，按照要求的各种依赖也都  
安装了，但是用vim打开md文件还是悄然无声。
    据说Sublime也可以支持markdown，所以就在Ubuntu下安装了一个Sublime3.  
之后就可以安装markdown插件了。  
    按ctrl+～，输入命令：  

    import urllib.request,os; pf = 'Package Control.sublime-package';  
    ipp = sublime.installed_packages_path();  
    urllib.request.install_opener( urllib.request.build_opener(    \
    urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf),  \
    'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/'\
    + pf.replace(' ','%20')).read())  

   之后可以安ctrl+shift+p，输入命令：  

    package, 选择install package  
    markdown, 选择markdown preview  

重启sublime，打开一个md文件，输入命令：  

    markdown preview， 选择 preview in browser  

这样就可以预览了，但是不是实时的。  

***
ps: 刚刚在编辑post的时候是用vim编辑的，结果久等的实时窗口终于来了。原来  
它识别的是markdown后缀，而不是md后缀。
pss: 话说这个主题和配色好丑啊，有空了再弄吧。


[1]: https://github.com/hyace/vim-instant-markdown
