---
layout: post
title: "Thrift的安装以及问题"
date: 2014-08-27 13:33:42 +0800
comments: true
categories: 
---
对Thrift这个框架知之甚少，想先了解下，我自己的习惯是先安装好跑个小demo。  
   首先从[这里下载][1]到Thrift的包。  
   Thrift对于安装环境的要求是Unix/Linux系统，Windows要装需要配合cygwin。  
   解压安装包，进入，运行：  

    ./configure  
    make  
    make install  
   
   大体步骤是这样，但是中间遇到了很多问题。最主要的一个是在make的时候，运  
   行ruby的bunble这一步出错，显示如下：  

    Bundler could not find compatible versions for gem "bundler":
      In Gemfile:
          bundler (~> 1.3.1) ruby

        Current Bundler version:
          bundler (1.7.2)

    This Gemfile requires a different version of Bundler.
    Perhaps you need to update Bundler by running `gem install bundler`?  

   Stackoverflow上的答案是[这样][2]：  

    % gem install bundler -v '~> 1.0.0'
    Successfully installed bundler-1.0.22
    Then force rubygems to use the version you want (see this post):

    % bundle _1.0.22_ install  

   这个也是版本不对，只不过不是1.3.1的，但是执行了始终不对。装了1.3.1的  
bundler但是默认识别的还是1.7.2的，于是用gem把高版本的删了，接着make才成  
功，之后有装回来了。  
   ps：寻找答案过程中才知道墙内有个网站叫[segmentfault][3],还有个网站叫  
[outofmemory][4]，方知山寨之伟大～

[1]: http://incubator.apache.org/thrift/download/
[2]: http://stackoverflow.com/questions/12092928/how-to-bundle-install-when-your-gemfile-requires-an-older-version-of-bundler  
[3]: http://segmentfault.com/
[4]: http://outofmemory.cn/
