---
author: admin
comments: true
date: 2012-11-23 03:03:44+00:00
layout: post
slug: php-offline-download
title: 用php实现简单的离线下载功能
wordpress_id: 594
categories:
- php
tags:
- php
---

近期，班里一个同学（大神级别）把自己实验室的电脑贡献出来做班级网站的服务器，班级网站主要是借助于校内局域网的速度来分享学习资料，以及展示班级风采等。经过我跟另外一位同学的折腾，这网站算是搞起来了，也确实是方便了同学也方便了自己。闲下来了，我又想还能再干点什么呢？

既然用的是校园网，当然是少不了对校园网速度的各种抱怨各种吐槽，即使是100多M大的文件，在校园网100k的网速面前也是苦不堪言啊！我又想到既然班级网站的服务器天天开着，而且内网的资源分享又不会影响外网的访问速度，那何不利用起来，让服务器把要下载的东西先下载下来，然后再从服务器上快速下载到自己的电脑上呢，说干就干！

<!-- more -->

### 现有资源

linux服务器一台，ssh帐号一个（无root权限）

### 实现方法
帐号没有root权限，所以就先不考虑安装其他软件，而是用php和wget来实现。

首先要解决的问题，就是如何从url下载文件并保存到指定的文件夹。google一番之后，php自带的一些函数都能实现从远程地址下载文件，比如copy ，fopen等，但这有一个问题，就是必须等到文件下载完成之后才能继续进行php程序，所以无法实现后台下载。

那就考虑通过调用shell的wget命令来实现。wget相当强大，用来实现简简单单的http下载当然是轻轻松松。具体代码如下：

	shell_exec("wget -b -nc --restrict-file-names=nocontrol -P ./download ".escapeshellarg($url));
	   // -b 后台下载
	   // -nc 文件已经存在时不覆盖
	   // --restrict-file-names 解决中文地址导致文件名乱码的问题
	   // -P 保存路径
	   //escapeshellarg相当于加引号，可以防止url中有空格时导致的错误


核心的功能实现了，那就看怎么方便使用。一开始我是在index.php中填好了url再post到file.php的。后来发现这样使用起来很不方便，打开了file.php又要让它自动关闭，没有必要，就直接post到自己。这里有个问题，因为一开始url是没有赋值的，所以php会有warning产生，这看起来很不友好。所以要用safePost().

	function safePost($str)
	{
	$val = !empty($_POST["$str"]) ? $_POST[$str]:null;
	return $val;
	}

出于安全的考虑，还是要对下载的文件类型进行简单的限制。

	$url = safePost("url");
	$allow_type=array("iso","xls","xlsx","cpp","pdf","gif","mp3","mp4","zip","rar","doc","docx","mov","ppt","pptx","txt","7z","jpeg","jpg","JPEG","png");
	//允许的文件类型
	$torrent = explode(".",$url);
	$file_end = end($torrent);
	$file_end = strtolower($file_end);
	if(in_array($file_end,$allow_type))
	{
	//白名单，执行程序
	}

服务器下好了，也当然要能方便地下载到本地来，使用php遍历并列出download文件夹中的文件（暂未实现按照文件修改时间排序）,另外，由于wget运行后就在后台了，无法获取到是否完全下载完成的信息。那只能曲线救国，通过获取文件的大小，刷新查看文件的大小变化情况来确认下载进度。

	function writelist($Spath){
	echo'</pre>
	'; if(is_dir($Spath)){ $path = scandir($Spath); for($a=0,$index=0;$a
	<table>
	<thead>
	<tr>
	<th style="text-align: center;">#</th>
	<th>名称</th>
	<th>文件大小</th>
	</tr>
	</thead>
	<tbody>
	<tr>
	<td style="text-align: center; width: 60px;">'.$index.'</td>
	<td><a title="点击下载" href="'.$Spath.'/'.$path[$a].'" target="_blank">'.$path[$a]."</a></td>
	<td>".$size."MB</td>
	</tr>
	</tbody>
	</table>

### 细节调整

使用了bootsrap，方便了许多（其实是自己的css，js功底基本为0），再加了一些简单的js，比如按下enter后自动刷新，使用感受好一点。

[view code on github](https://github.com/gracece/offline-download)

### 待解决问题
对迅雷p2p，ut等暂时无力。
