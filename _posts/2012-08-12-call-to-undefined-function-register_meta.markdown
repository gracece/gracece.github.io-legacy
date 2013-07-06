---
author: admin
comments: true
date: 2012-08-12 05:26:45+00:00
layout: post
slug: call-to-undefined-function-register_meta
title: 'Fatal Error: Call to undefined function register_meta()'
wordpress_id: 315
categories:
- 日志
tags:
- SAE
- wordpress
---

昨天在给worpdress更换cakifo主题时，出现的意外的错误，更换完主题，退出后再次登录直接就down掉了，显示：


> Call to undefined function register_meta()


最后经过谷歌（百度是什么都搜不出来的，稍后继续吐槽），得出是wp4sae的版本不是最新的，而主题里面应用了新的函数，所以博客直接就无法正常载入，此真是[致命性错误](http://wordpress.org/support/topic/theme-oxygen-fatal-error)！



最开始是想通过sae的服务管理项目里面的mysql管理，直接把cakifo替换成默认主题或者其他可用主题，但结果都失败了。

后来看到[wp4sae](http://wp4sae.org/2012/06/wordpress-for-sae-3-4-1-rc0-release/)关于最新测试版文章升级后可能出现问题的解决办法，可以通过sae后台清除memcache，并在代码管理里面将可怜的不被支持的主题删除，使得wordpress改为默认主题重新载入。抓住救命稻草，果然真的可以！我提着的心终于放了下来，谁想每天写完两篇文章然后博客就down掉啊。


[![](http://glygo-wordpress.stor.sinaapp.com/blog/wp-content/uploads/2012/08/memcache.png)](http://gracece.net/blog/wp-content/uploads/2012/08/memcache.png)另外一件事情，经过一番折腾，提交sitemap，用SEO插件，昨天谷歌已经收录我这个博客了，百度还没有任何踪迹，蜘蛛都没来爬过。百度搜索真是一潭深水啊，怪不得搞技术的都喜欢谷歌。国内的其他搜索引擎我就不说什么了。




[![](http://gracece.net/blog/wp-content/uploads/2012/08/serchgly.png)](http://gracece.net/blog/wp-content/uploads/2012/08/serchgly.png)下午试试升级到wp3.4.1测试版，搞定之后真的要开始看CSS 了。
