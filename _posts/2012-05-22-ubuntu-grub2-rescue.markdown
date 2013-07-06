---
author: admin
comments: true
date: 2012-05-22 05:58:00+00:00
layout: post
slug: ubuntu-grub2-rescue
title: ubuntu 修复grub2
wordpress_id: 155
categories:
- linux
tags:
- ubuntu
---

蛋疼手贱，在windows下把winX86分区改为主分区，结果开机就挂了，grub error了，grub rescure啊，无奈，先在win7PE环境下用diskgenius重建了mbr，暂时可以进win7系统了，进入之后用easyBCD增加ubuntu启动项（grub2），结果还是错误。度娘万能，赶紧看看怎么解决。百度很多,再尝试，结果发现都不见效，猛然醒悟，那些都是针对grub的，赶紧再找了下grub2相关到，总算解决了问题。

研究了半天，才解决这个问题。先说说是怎么回事：

安装ubuntu时，启动是用grub2进行启动。我的系统折腾多了，盘分得相当混乱了。 我们知道，每次系统启动时都是先进入grub，也就是先在ubuntu的启动目录里选择进入哪个系统，如果按分区来讲，grub2在(hd0,msdos10)也就是我的ubuntu所在的分区。那么启动时root应该设在(hd0,msdos10),在更改了分区设置之后，我的分区号码发生了改变，msdos10已经不再指向ubuntu的所在分区，而ubuntu是不能识别NTFS这种文件系统的，所以就有了error:unknownfilesystem，这种情况下自然不能启动，那么grub2就会启动grubrescue模式，就是修复模式。那么我们要做的就是把grub重新指向ubuntu的所在分区即可解决问题。

下面是具体步骤：

## 查看分区

因为每个人的分区不一样，所以我们要查看分区,用ls指令

    
    grub rescue>ls


回车后，就会出现

    
    (hd0)(hd0,msdos13) (hd0,msdos12) (hd0,msdos11) (hd0,msdos10)(hd0,msdos5).。。。(hd0,msdos1)
    grub rescue>


注：我用的是grub2,对于grub用户，分区前没有msdos字样
上面是我的分区，每个人的不一样。

    
    grub rescue>set<ENTER>
    prefix=(hd0,msdos10)/boot/grub
    root=hd0,msdos10


从上面可以看出来现在我的ubuntu系统是从(hd0,msdos10)里启动的。
那么怎么知道ubuntu在哪个分区呢？进入第二步

## 寻找ubuntu所在分区 

这一步我们要一个一个的试，
还是用ls指令
先试下在不在（hd0,msdos10）里边

    
    grub rescue>ls (hd0,msdos10)


回车会发现，不是，还是 **unknownfilesystem(NTFS无法识别带来到结果）**
dd接着来
。。。。。。。。。
当我试到
(hd0,msdos8)的时候，可以看到 **badfilename**，难道我的分区内容挂了?其实没有，只是grub2未能识别其中到内容而已。

## 修改启动分区

    
    grub rescue>root=(hd0,msdos8)
    grub rescue>prefix=/boot/grub //grub路径设置
    grub rescue>set root=(hd0,msdos8)
    grub rescue>set prefix=(hd0,msdos8)/boot/grub
    grub rescue>insmod normal //启动normal启动
    grub rescue>normal


出现了熟悉到启动选项画面，太激动了！

进入Ubuntu后,修复grub，在终端里运行：

    
    sudo update-grub


重建grub到第一硬盘mbr

    
    sudo grub-install /dev/sda


好啦，重启，一切搞定！


