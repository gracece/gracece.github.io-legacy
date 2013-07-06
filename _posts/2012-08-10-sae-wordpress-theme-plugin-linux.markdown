---
author: admin
comments: true
date: 2012-08-10 09:54:11+00:00
layout: post
slug: sae-wordpress-theme-plugin-linux
title: linux下SAE平台 WordPress主题、插件安装方法
wordpress_id: 61
categories:
- wordpress
- 学习
tags:
- linux
- SAE
- wordpress
- 主题
- 插件
---

windows传送门：[windows下SAE平台 WordPress主题、插件安装方法](http://gracece.net/2012/08/windows-sae-wordpress-themes-plugins/)

**写在前面**

新浪SAE 提供了一个简单易用的平台，让Web开发者和普通互联网上网人群都能轻松享受到这种快捷易用。使用SAE平台创建新的应用，安装上针对SAE平台移植的wordpress，就可以开始自己的博客之路了。wordpress之强大毋庸置疑，各种功能强大的插件，还有或高雅或低调，各式各样的主题任你挑选，而这一切，都是免费的！这当然令人激动。SAE 平台上的wordpress是特别原版的wordpress特别针对SAE优化移植过来的，保留了wordpress的大部分特性，但还是难免会有缩水。因为SAE 跟一般的网站不同，SAE平台上是不能直接在服务器上更改文件的，出于这个原因，Wordpress博客的主题和插件等的安装与卸载就不能使用以往的方法，我们需要曲线救国，来解决问题。

现在我就来大概说下在linux下如何快速安装wordpress 的主题和插件。




## 安装主题


1、安装SVN

打开终端，ubuntu下输入

    sudo apt-get install subversion


其他平台类似。

2、创建SVN库
SAE默认是没有给我们创建SVN 库的，所以我们需要自己动手点击一下。
在代码管理里面点击创建SVN库。

![创建SVN库](http://ww3.sinaimg.cn/large/50b560a5gw1e6dfum5zh5j20rc0ac0us.jpg)
3、本地checkout
打开终端，输入

    svn co https://svn.sinaapp.com/appname


把上面的appname替换为你自己创建的应用名，比如我这里是grtest
第一次checkout时需要验证，用户名/密码为您的SAE安全邮箱和安全密码(非微博登陆账号密码！！这点一定要注意)。
checkout完后会在home目录中创建与应用同名的文件夹。

4、上传主题
在[wordpress网站](http://wordpress.org/extend/themes/browse/popular/)上选择自己喜爱的主题，下载并解压备用。
将主题文件夹复制到～/appname/1(version number)/wp-content/themes
其中的数字为版本号，默认为1.
具体代码如下：

    
    cd ~/grtest/1/wp-content/themes
    cp -rf ~/downloads/sundance ./
    svn add ./*
    svn commit -m "add new theme"


等待上传完毕：

![upload done](http://ww4.sinaimg.cn/large/50b560a5gw1e6dfuxwnvqj20ex09gabw.jpg)

5、在外观选项更换主题
打开博客的控制面板，选择安装好的主题，应用即可。

![安装主题](http://ww4.sinaimg.cn/large/50b560a5gw1e6dfw4xkkcj20gy0fv0uo.jpg)

## 安装插件

 插件的安装与主题类似，只需把所要安装的插件复制到plugins里面即可。


另外更多信息请参考[官方文档](http://sae.sina.com.cn/?m=devcenter&catId=212#anchor_a56d0a200f29dc4dcb9240ee47c794f9)。
