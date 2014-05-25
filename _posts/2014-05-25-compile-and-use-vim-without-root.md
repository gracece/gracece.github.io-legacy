---
layout: post
title: "无root权限编译配置使用vim74"
description: "为了愉快地写代码，折腾编译一下vim找回熟悉的配置还是相当值得的。"
category: linux
tags: 
- vim
- compile
---
{% include JB/setup %}

### 需求
实习的时候，发现公司的服务器上用的vim版本是6.4(ORZ 好像我开始接触的linux的时候就已经是7.3了)，查了下wiki，这版本是2005年，那时
我估计刚上初中。版本低，最大的问题就是各种插件都用不了，像我这等插件党，没有了那一套风骚的插件配置自动补全，感觉都不能愉快地打代码了。
我的岗位是业务运维，长时间写代码的时候不多，更多是用前人写好的工具做事，所以配个熟悉的vivdchalk,加上简单的vimrc还是可以用的。
但是未曾想分配给我运营开发的活，需要用python开发web应用。无插件写了一会，感觉还是得搞起。

### 准备
- 系统环境为suse10,gcc版本4.2，无root权限，无外网，所以只能曲线救国编译到自己的家目录下。
- 下载[vim 7.4源码](ftp://ftp.vim.org/pub/vim/unix/vim-7.4.tar.bz2),解压备用。

### 编译及安装
在`vim74/src/INSTALL`有这么一段话:
>Unix: PUTTING vimrc IN /etc
>
>Some Linux distributions prefer to put the global vimrc file in /etc, and the
>Vim runtime files in /usr.  This can be done with:

>        ./configure --prefix=/usr
>        make VIMRCLOC=/etc VIMRUNTIMEDIR=/usr/share/vim MAKE="make -e"
>

所以就这么简单，只需要声明prefix即可。具体来说就是将prefeix指定到自己有权限的目录：

    ./configure --prefix=/home/grace/usr --with-features=huge --enable-multibyte 

上面这一句我声明prefix到家目录的usr文件夹，同时启用可用的最大特性支持，最后加了一个多字符集文件支持。对于
configure的更多选项，可以通过执行`./configure --help`查看，写得非常清楚。同时，关于编译还可以参考
[Building Vim from source ](https://github.com/Valloric/YouCompleteMe/wiki/Building-Vim-from-source)
和[Building_vim](http://vim.wikia.com/wiki/Building_Vim),都写得非常清楚了。完成上一步后，如果没有报错或提示
缺少依赖，那么就可以进行下一步了，如果不幸遇到有未安装的依赖，那么还是得求助有root权限的用户，或者自己再手动
编译安装缺少的依赖。

    make VIMRCLOC=/home/grace/etc VIMRUNTIMEDIR=/home/grace/usr/share/vim/vim74 MAKE="make -e"

执行上面一条命令，将vim安装到家目录下，同时指定了vimrc的目录以及VIMRUNTIMEDIR值，这里需要注意后面的`vim/vim74`，如果写错
了到时打开vim的时候会报错。VIMRUNTIMEDIR是vim运行时候所需资源的目录，非常重要。编译的时候，如果有条件，比如服务器是16核32G
内存的，可以加 -j 选项，这样编译更快。编译完成，即可在刚刚指定的prefix目录下找到vim的二进制文件，执行就可以看到熟悉的
vim7.4啦。

### 配置及使用
编译好vim之后，可以通过alias的方式启动，也可以将自己放二进制的目录export到PATH。 在vim的help文件中，可以看到vim载入文件配置的详细说明，最关键的：

    'runtimepath' 'rtp'	string	(default:
					Unix: "$HOME/.vim,
						$VIM/vimfiles,
						$VIMRUNTIME,
						$VIM/vimfiles/after,
						$HOME/.vim/after"
                        ......

可以看到vim会尝试从home目录的.vim文件夹载入相关插件等配置资源。如果你是按刚刚的方式，有自己独立的用户，那么就可以直接把自己熟悉的vim配置及插件
等复制过来，文件夹命名为.vim，vimrc的位置为`～/.vim/vimrc`,还算是比较熟悉的。

假设有另外一种情况，你所使用的服务器由于某些原因，是跟别人共用
一个用户的，所以你就不能在这主目录下的.vim文件夹乱搞的，因为这样vim6.4会报错的。你可以这样，在自己的vimrc最前面加入:

    :set runtimepath=~/my-vim-dir,$VIMRUNTIME

找一个目录放你自己的配置文件，同时通过vimrc将该目录加入到$VIMRUNTIME中，在启动vim的时候，通过`-u`选项指定所使用的vimrc文件，
这样就可以愉快地使用 插件了。

再如果，你原来是使用`vundle`管理你的插件的，比如[我](https://github.com/gracece/dot-vimrc),那么需要改多一个地方，在bundles.vim中：

    set rtp+=~/.vim/bundle/vundle/
    call vundle#rc('path-to-bundle-path')

确保rtp的vundle正确，同时在vundle的初始化函数中声明插件所在的目录，具体参考[stackoverflow](http://stackoverflow.com/questions/9809209/how-to-make-vim-vundle-install-plugin-to-another-path)。
话说在没有网络的情况下，vundle都没有任何用武之地了，最终的结果是直接把自己电脑上原来的.vim文件夹搬过来，简单粗暴。

### 结束
至此，熟悉的vim环境就搞定了，又可以愉快地写代码了。


