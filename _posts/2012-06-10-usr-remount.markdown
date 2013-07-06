---
author: admin
comments: true
date: 2012-06-10 11:53:20+00:00
layout: post
slug: usr-remount
title: /usr 重新挂载
wordpress_id: 104
categories:
- linux
- 学习
tags:
- /usr
- linux
---




由于一开始在网上看的分区方案是说/只要10G左右就够了，然后就只给/分了10G，装了众多软件后，容量就显然不够了，那应该是几百年前的教程了。蛋疼啊我750G硬盘居然要面对容量不足的问题。linux暂时无法无损增加分区，考虑重新挂载/usr。

参考：[cnblogs](http://www.cnblogs.com/jjyoung/archive/2011/10/27/2226326.html)、[danielblack](http://www.danielblack.name/archives/47)

/usr在Linux一般是很多软件的默认安装目录，因此很大一部分空间是被/usr所占，如果新建一个较大的分区，让其在启动时挂载到/usr下，替换原来的/usr，就达到了目的。但是，值得注意，其实重挂载/usr很危险，不像/home，改动/usr很容易导致系统挂掉

1、

    
     sudo fdisk-l


可以列出当且的分区信息，然后用gparted之类的分区工具，分出一个新的ext4分区。注意分出来到的新分区是否改变了原来分区到数字，反正我的是没变。

2、

    
    sudo fdisk -l


查看你的新分区的位置（如/dev/sda7），然后挂载（如mkdir/mnt/new，sudomount /dev/sda7 /mnt/new）。

3、复制/usr下的所有文件到新分区，

    
    sudo cp -p -r /usr/*/mnt/new


值得提醒的是，不能忘记-p，-p表示复制时维持复制源的文件信息（权限、用户），/usr下很多文件都是右权限限制的，如sudo命令的权限带有s，表setuid，不带-p的复制便不会复制其s、t权限，而且会将owner全变为root，这样挂载后会发现很多命令都失效，一些硬件驱动也读不了。另注意不能在新分区下另建一个/usr文件夹，（怕复制零散的文件速度太慢，可改用tar命令来操作

    
    #sudo su 
    cd /usr
    usr# tar cvfp usr.tar ./usr
    # mv usr.tar /Your/New/Partion (简称YNP)
    YNP#tar xvfp usr.tar //ok 现在usr目录已经复制过来了）


4、修改fstab,

    
    sudo gedit /etc/fstab
    ls-l /dev/disk/by-uuid


－－查看分区的UUID。此文件是记录启动时的挂载方式、对象。根据注释填吧。/dev/sda*（*代表你的盘符序号，当然你有可能不是sda。。总之换成你的新分区名） /usr ext4 defaults 0 0，然后save。

5、最后

    
    sudo mv /usr /usr_old


不要把/usr删了。。万一出啥事再到livecd上改回来就行。万事都不要断后路。

6、

    
    mkdir/usr


－－创建新的/usr挂载点，启动时自动挂载/usr分区到此处。

7、

    restart


这个时候由于/usr目录没有了，图形界面应该会崩溃吧 ，于是进入虚拟终端后 使用 reboot，shutdown等命令都提示bashnot found 之类的话.按下 Ctrl+ Alt + Delete 这时候应该就会自动重启了。重启无任何异常，成功！可以把usrold移到home里面，或者直接干掉。

PS：/home 的挂载可参照上面的方法，基本没有危险。


