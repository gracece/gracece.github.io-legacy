---
author: admin
comments: true
date: 2012-08-11 05:51:15+00:00
layout: post
slug: windows-sae-wordpress-themes-plugins
title: windows下SAE平台 WordPress主题、插件安装方法
wordpress_id: 253
categories:
- wordpress
tags:
- SAE
- wordpress
- 主题
- 插件
---

昨天写了一篇关于linux下安装WP4sae主题和插件的文章（[linux下SAE平台 WordPress主题、插件安装方法](http://glygo.sinaapp.com/2012/08/sae-wordpress-theme-plugin-linux/)），今天我就来整理下windows系统下的操作方法，可参考[官方文档](http://sae.sina.com.cn/?m=devcenter&catId=212#anchor_6bcc83578e796e2a6ab603fe5fc022e0)。

## 1、安装SVN

使用TortoiseSVN客户端


>     在Windows下推荐使用乌龟(Tortoise)SVN客户端。 TortoiseSVN 是 Subversion 版本控制系统的一个免费开源客户端，可以超越时间的管理文件和目录。文件保存在中央版本库（即SAE中央SVN仓库），除了能记住文件和目录的每次修改以 外，版本库非常像普通的文件服务器。你可以将文件恢复到过去的版本，并且可以通过检查历史知道数据做了哪些修改，谁做的修改。这就是为什么许多人将 Subversion 和版本控制系统看作一种“时间机器”。


**下载安装**

TortoiseSVN下载：[http://tortoisesvn.net/downloads.html](http://tortoisesvn.net/downloads.html)

如果不幸被墙或者下载速度太慢，请戳：[网盘下载](http://www.kuaipan.cn/file/id_62794401847443458.htm)


## 2、创建SVN库


SAE默认是没有给我们创建SVN 库的，所以我们需要自己动手点击一下。
在代码管理里面点击创建SVN库。


[![创建SVN库](http://gracece.net/blog/wp-content/uploads/2012/08/create.png)](http://gracece.net/blog/wp-content/uploads/2012/08/create.png)





## 3、创建本地工作目录


第一步，创建一个新文件夹作为本地工作目录(Working directory)，可以使用应用名为文件夹名。如，为我的应用glygo创建本地工作目录。在文件夹内右键-->点击“SVN Checkout”


[![](http://gracece.net/blog/wp-content/uploads/2012/08/createSVN-windows.png)](http://gracece.net/blog/wp-content/uploads/2012/08/createSVN-windows.png)


在弹出页面中填写仓库路径即可，这里是：

    
    https://svn.sinaapp.com/glygo


把后面的改成你app的名字即可，其它默认参数即可，如图:


[![](http://gracece.net/blog/wp-content/uploads/2012/08/new.png)](http://gracece.net/blog/wp-content/uploads/2012/08/new.png)

Reversion处，“HEAD revision”是指最新版，也可以指定Revision为任意一个版本。


此处需要认证，用户名/密码为您的SAE安全邮箱和安全密码

点击“OK”，出现下载界面，如果一切顺利，应用所有版本代码将会全部出现在刚刚创建的glygo文件夹下。


## [![](http://gracece.net/blog/wp-content/uploads/2012/08/data.png)](http://gracece.net/blog/wp-content/uploads/2012/08/data.png)

## 4、修改文件或增加文件夹


如果要修改代码，则在本地使用你喜欢的编辑器，编辑任意文件，保存后该文件图标将会出现红色感叹号。下面需要提交(commit)最近的更新。在需要更改的文件上击右键，出现菜单 ，选择“SVN commit”。然后填写关于本次更新的日志（log message），这是必填项，否则commit会失败。如图：

[![](http://gracece.net/blog/wp-content/uploads/2012/08/commit.png)](http://gracece.net/blog/wp-content/uploads/2012/08/commit.png)

之后若看到finished！的提示，则提交成功，并且项目的SVN版本号加1。

 ** 新增文件/文件夹**

 在 SVN工作目录下，对于文件修改，完成后只需要commit就ok了，但对于新增文件，或者从其它目录复制进来的文件或文件夹，需要在commit之前需 要做一步add操作，即将文件或文件夹添加到svn工作目录中来，否则SVN客户端不认它。具体操作很简单，如图：


[![](http://gracece.net/blog/wp-content/uploads/2012/08/add.png)](http://gracece.net/blog/wp-content/uploads/2012/08/add.png)


add完后，再右击commit，填写必要的commit信息即可上传。

更多Tortoise SVN使用帮助，请参阅：[http://www.subversion.org.cn/tsvndoc/](http://www.subversion.org.cn/tsvndoc/)


## 5、上传主题和插件。


上传主题和插件本质上就是上传新的文件夹到服务器，所以只需安装上述步骤操作即可。
在[wordpress网站](http://wordpress.org/extend/)上选择自己喜爱的主题和插件，下载并解压备用。

    
     将主题文件夹复制到   /appname/1(version number)/wp-content/themes
    将插件文件夹复制到   /appname/1(version number)/wp-content/plugins


其中的数字为版本号，默认为1.

等待上传完毕：


## 6、设置对应的主题和插件。


打开博客的控制面板，选择安装好的主题和插件，应用即可。

希望能帮到你。
