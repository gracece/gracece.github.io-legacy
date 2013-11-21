---
layout: post
title: "git服务器搭建"
description: "本文简述在服务器上搭建git服务器用于小组协同工作"
category: linux
tags:
- git

---
{% include JB/setup %}

本学期有一门课程是`软件工程`，以小组的形式进行。在写了一个多月文档之后，终于要开始写代码了。然后当然要解决的一个
问题就是小组组员之间如何协同工作。其实这个问题没什么疑问，就是用git。

下面是我参照`ProGit--服务器上的Git`进行配置的过程。

#### 协议的选择
Git可以使用四种主要的协议来传输数据：本地传输，SSH协议，Git协议和HTTP协议。在这些协议中，只有SSH协议是能够读和写的协议，
其他几个都是只读的。考虑到每个组员都要能操作代码，则必须要用SSH协议。

#### 部署
- 新建git用户
	
		sudo adduser git
		su git
		cd 

- 创建纯仓库

	在服务器上以git用户的身份把一个现存的仓库导出为一个纯仓库，即不包含当前工作目录的仓库（只有.git目录里面的内容）。

		git clone --bare project project.git

	这里如果你没有一个现存的仓库，即是从零开始的，那就需要先`git init --bare `一个空项目。

- 收集并添加协作者的SSH公钥到git用户

	每个协作者都通过`ssh-keygen`来生成自己的SSH公钥，默认是保存在`~/.ssh/id_rsa.pub`，
	收集这些公钥，添加到服务器上git用户的`～/.ssh/authorized_key`中，注意虽然公
	钥很长，但是每个公钥都只占一行的，不能随便换行。将所有协作者的公钥都添加完毕，则可以准备开始协同工作了。

- Web目录及挂钩部署
	
	因为在`project.git`目录中只有.git目录的东西，没有当前工作目录的具体代码，所以我们需要另外建立一个目录来作为项目的
	代码目录，用于线上测试web服务。在git用户的主目录下

		git clone project.git www

	这样，我们在www目录中就有了当前的代码，这里的内容就跟在协同者电脑上的是一样的，只不过我们还需要一个自动的部署功能。
	当协作者发送新的push到project.git，我们需要实时更新www目录中的代码。这里要用到git提供的挂钩。
	
	Git有两组挂钩：客户端和服务器端，我们这里需要用的是服务器端的挂钩。挂钩都储存在Git目录的/hooks子目录中，里面已经
	有默认的脚本（以.sample结尾）。服务器端主要有 `pre-receive`、`post-receive`以及`update`等，在[Git挂钩](http://git-scm.com/book/zh/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git%E6%8C%82%E9%92%A9)有详细说明，这里我们要用的是
	`post-update`。它会为推送者更新的每一个分支运行一次。我们把默认的`post-update.smaple`更名为`postupdate`，然后编辑`update`
	加入

		cd /home/git/www
		env -i git pull

	这个算是最简单的挂钩，即每次有推送过来，就自动在www目录中pull代码，实现同步更新。

	接着在服务器上增加一个虚拟主机，目录为刚刚所设定的www目录，具体配置过程每个发行版可能略有不同，但应该都不难配置。
	配置完我们就能通过域名来访问我们写好的代码。

#### 使用
在协作者自己的电脑上，就可以通过
	
	git clone git@git-server.com:project.git 

把代码clone下来并进行代码更改等协同工作，这里的`git-server.com`是指服务器的域名或者ip。
如果所有协作者都是用linux当然是极好的事情，但实际上应该是各个平台都会有。在windows上可以用 mysgit 来进行，mysgit提供了
改进过的shell，集成了一些常用命令，所以在windows的git bash下就能跟在linux一样进行git的全部操作。

另外一个问题是，window和linux/Unix的换行回车是不一样的，这个小问题可能会引起一些奇奇怪怪的问题。不过还好git也提供了相应的
支持，具体可以google一下进行相应的配置。

#### 参考
- [Pro Git](http://git-scm.com/book/zh)
- [使用 Git Hooks 实现自动项目部署](http://icyleaf.com/2012/03/apps-auto-deploy-with-git) 
