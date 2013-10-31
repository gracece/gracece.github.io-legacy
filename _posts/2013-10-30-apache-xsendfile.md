---
layout: post
title: "Apache2借助X-Sendfile实现文件下载控制"
description: "本文简述使用X-Sendfile 实现Apache2下更高效灵活的文件下载控制。"
category: linux
tags: 
- apache2

---
{% include JB/setup %}

##### 现阶段情况：

用户所有的下载请求都由download.php来处理，记录下载次数以及其他一些信息处理，然后使用
header ('Location: xxx') 跳转到文件真实地址。

##### 效果：

简单粗暴，但同时用户也能轻易直接拼接一个URL访问该文件，而不用经过download.php，更可恶的是有爬虫无休止地爬（明明那个文件早就删除了，还是一直请求）。

##### 需求：

想要解决上述问题，有两种想法，一是把资源所在文件夹放在http不可访问的地方，然后在download.php内使用readfile()之类的方法来读取，但是处理大文件的时候怕内存吃不消。
二是用.htaccess 限制referer，但是这个用起来更加不灵活，因为无法在服务器端设置referer(要不然用cURL？），且referer也能轻易被伪造。

以上是我在[V2Ex](http://v2ex.com/t/87554#reply1)的提问内容，只有一个人回复，回复只有一个单词`xsendfile`,但是有谷歌，这一个单词就够了！

#### 什么是X-Sendfile

X-Sendfile是一种机制，它让后台代码从耗费内存的文件读写中解放出来，只需要专注于基本的逻辑判断或者权限检测等，而把文件下载的
工作交给Web服务器。相比readfile(),这种机制更加高效，使用也非常简单，只需要通过指定一个特定的http header，当Web服务器检测到该header，它将忽略其他输出而只发送文件。这就非常适合用于权限检测通过之后输出下载文件，同时还可隐藏文件的真实地址。

#### 如何配置使用

Linux+Apache2 的环境下，还需要配置一个模块 `mod_xsendfile`才可使用。[tn123](https://tn123.org/mod_xsendfile/)
这里有非常详细的说明，下面简要说说配置步骤。

- 安装模块

	下载源代码，使用apache2提供的工具进行编译安装，`apxs -cia mod_xsendfile.c`，可能提示无法找到`apxs`，则先安装
	`apache2-prefork-dev`，重启apache2即可。

- 配置apache2

	上面步骤之后，模块应该是已经启用了的，可以用 `a2enmod xsendfile` 检查下是否启动。然后在配置文件中加入一句话`XSendFile on`,这里的配置文件可以是服务器的配置文件apache2.conf，也可以是虚拟主机的配置文件，或.htaccess文件，甚至可以指定到某一个文件才可启用这个功能。再配置虚拟主机，使得资源所在的文件夹不能直接通过url访问，
	从而达到控制权限且有效记录下载次数的目的。

		<Directory /var/www/upload/>
	         Options  FollowSymLinks 
	         AllowOverride None
	         Order allow,deny
	     </Directory>

- 编写代码
	
	参考tn123中的示例代码，加入自己所需要的功能即可。

		<?php
		//some other code
		if ($user->isLoggedIn())
		{
	        header("X-Sendfile:upload/".$fileTodown);
	        $ext = pathinfo("upload/".$fileTodown,PATHINFO_EXTENSION);
	        if($ext == "pdf")
	        {
	            header("Content-Type:application/pdf");
	            header("Content-Disposition:inline;filename=".$fileTodown);
	        }
	        else
	        {
	            header("Content-Type:application/octet-stream");
	            header("Content-Disposition:attachment;filename=".$fileTodown);
	        }         
		    exit;
		}
		?>
		<h1>Permission denied</h1>
		<p>Login first!</p>

	上面的代码中，我加入了文件类型检测，使得对于pdf文件，浏览器能够直接打开，而不是保存为文件。关于Content-Type,可以参考
	[OSTOOLS](http://www.ostools.net/commons),关于Content-Diposition,可参考[RFC6266](http://tools.ietf.org/html/rfc6266),这两个关键的属性决定了浏览器会怎么处理你使用xsendfile发过来的文件。filename属性要注意可能会产生的浏览器不兼容的问题，比如我上面写的在高贵的IE6上文件名就会乱码。


#### 参考

- [mod_xsendfile for Apache2/Apache2.2](https://tn123.org/mod_xsendfile/)
- [How I PHP: X-SendFile](http://www.jasny.net/articles/how-i-php-x-sendfile/)
- [使用 Nginx 的 X-Sendfile 机制提升 PHP 文件下载性能](http://www.lovelucy.info/x-sendfile-in-nginx.htm)
- [奇妙的Apache 2.x  mod_xsendfile](http://blog.sina.com.cn/s/blog_549212ae010086s9.html)
- [让PHP更快的提供文件下载](http://www.laruence.com/2012/05/02/2613.html)

