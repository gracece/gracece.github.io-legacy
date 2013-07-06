---
author: admin
comments: true
date: 2012-08-25 08:40:11+00:00
layout: post
slug: linux-bash-shell
title: 给linux添加google搜索命令
wordpress_id: 458
categories:
- linux
tags:
- linux
---

跟终端打交道，自然希望这个终端能够更强大一些。经常要使用google，所以就给终端增加个google搜索命令。

方法：


	cd /usr/bin/
	sudo vim google


输入如下代码：


	#!/bin/bash
	if [ ! -n "$1" ]; then
	xdg-open http://www.google.com.hk
	fi

	if [ -n $1 ] ; then
	xdg-open http://www.google.com.hk/search?q=$1;
	fi


那 xdg-open 又是什么命令呢？原来 xdg-oepn 可通过用户默认程序打开文件或 URL。如 xdg-open 123.png 就会通过默认图像程序打开该图片。知道这些之后，您就可以轻松进行个性化设置。

用`chmod +x` 使其具有可执行属性。	

然后在终端输入google就可以直接打开谷歌页面，输入 `google sth`就可以搜索这个关键词。如果嫌google太长，那也可以直接改为单字母g，这样更加简介。效果：


[![](http://ww2.sinaimg.cn/large/50b560a5gw1e6df1znw6gj20np0fhjv5.jpg)](http://ww2.sinaimg.cn/large/50b560a5gw1e6df1znw6gj20np0fhjv5.jpg)


可以参考 [shell编程基础](http://wiki.ubuntu.org.cn/Shell%E7%BC%96%E7%A8%8B%E5%9F%BA%E7%A1%80)，[Linux Deepin 终端彩蛋](http://planet.linuxdeepin.com/2012/03/12/easter-egg-in-linux-deepin-terminal/)，增加属于你自己的命令，比如输入weibo就打开微博页面等（虽然微博控是不好滴）。
