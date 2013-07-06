---
author: admin
comments: true
date: 2012-09-18 04:29:24+00:00
layout: post
slug: archlinux
title: archlinux
wordpress_id: 518
categories:
- linux
tags:
- arch
- linux
---

本来用的是ubuntu12.04，后来尝新鲜把deepinlinux装上去了，deepin确实有许多很不错的地方,比如在本土化方面，以及相当不错的音乐播放器和视频播放器等创新点。但稳定终究才是王道，经过deepin改进的gnome用起来还算顺手，但就是偶尔来一下崩溃，次数多了也就烦了。再加上一直以来就想试试arch的滚动升级，索性就换了吧，以后也不用每个版本发布都要折腾两下。想我上次想装arch还是几个月前，可是迟迟没有行动，现在已经要网络安装了，没办法，那就找个地方蹭网呗。

<!-- more -->

于是在昨天，周一下午，上完还算愉快的马原课，背着电脑来到前台。Beginners' Guide已经看过了，也试着在vbox上装了一次，虽然最后到装X那一步卡住了，囧，但觉得前面的步骤都没问题了，所以直接上吧。先看看我装完后的磁盘：


[![](http://gracece.net/blog/wp-content/uploads/2012/09/mydisk.png)](http://gracece.net/blog/wp-content/uploads/2012/09/mydisk.png)


第一次安装：

心想，装了就装彻底点，然后就直接把硬盘最前面的系统保留分区干掉，然后分为/boot ,原来的/ 格式化 ，/home 不格式化，挂载上去。按照Beginners’s Guide 的步骤，一步一步敲代码，然后重启，悲剧了，显示


> filesystem check failed


谷歌众多内容后无果，润泽大牛说装个五六遍都是正常的，那我还是重新来吧。

插曲:

进winpe用diskgenius把硬盘最前面的系统保留分区恢复，重写mbr，恢复windows启动项，进入系统，还好，还能进入。硬盘分区好乱，想调整下，把系统保留分区调小一点（我也不知道哪次折腾的时候把它搞成1.2G了）,再把root分区移到前面来，home就不管了，移动实在太麻烦。打开acronis Disk Diretor 11 进行调整，提示需要重启，那就重启呗。进入系统发现D盘出问题了，显示需要格式化。折腾多了，也淡定多了，不就是把分区表搞乱了吗，用diskgenius找回，具体过程就不写了。话说这个DG救了我好多次，是不是该考虑捐赠下。。 总之，最后还是保持原来的分区情况，重来一次吧。

第二次：

下面的安装内容是截取自[Beginners' Guide](https://wiki.archlinux.org/index.php/Beginners%27_Guide)的，就当是自己的安装备忘。

联网是直接dhcp的，所以直接就能用，不用过多设置。

分区：选择了保守点，不把/boot单独拿出来了，/home分区也先不管了，装完系统再挂载上去。

    
    # cfdisk /dev/sda


安装提示，在原来的/root分区重新建立，然后格式化

    
     mkfs.ext4 /dev/sda5


别忘了还有swap

    
    # mkswap /dev/sdaX && swapon /dev/sdaX




### 挂载分区


检查当前磁盘的标识符和布局：

    
    # fdisk -l


挂载根分区到/mnt.

    
    # mount /dev/sda1 /mnt




### 选择安装镜像


编辑 `/etc/pacman.d/mirrorlist`，把中国的镜像前面的#去掉，ustc还是挺快的


### 安装基本系统


使用 [pacstrap](https://github.com/falconindy/arch-install-scripts/blob/master/pacstrap.in) 脚本安装基本系统：

    
    # pacstrap /mnt base base-devel




### 生成 fstab



    
    # genfstab -p /mnt >> /mnt/etc/fstab




### Chroot 到新系统


下面要 [chroot](https://wiki.archlinux.org/index.php/Chroot) 到新安装的系统：

    
    # arch-chroot /mnt




### 配置系统




#### Hostname


将 _hostname_ 写入 `/etc/hostname`.

    
    gracece


将 _hostname_ 写入 `/etc/hosts`,作为别名：

    
    127.0.0.1   localhost.localdomain   localhost <strong>gracece</strong>
    ::1         localhost.localdomain   localhost <strong>gracece</strong>




#### 终端字体和键盘映射


编辑`/etc/vconsole.conf`.

    
    KEYMAP=us
    FONT=
    FONT_MAP=







#### 时区


编辑文件 `/etc/timezone`

    
    Asia/Shanghai


更多选项请阅读`man 5 timezone`。

将`/etc/localtime` 软链接到 `/usr/share/zoneinfo/Zone/SubZone`.

    
    # ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime




#### Locale




##### 启用 locales


选定你需要的本地化类型(移除前面的＃即可), 比如中文系统可以使用:

    
    en_US.UTF-8 UTF-8
    zh_CN.GB18030 GB18030
    zh_CN.GBK GBK
    zh_CN.UTF-8 UTF-8
    zh_CN GB2312


运行：

    
    # locale-gen




##### 设置系统 locale


Set [locale](https://wiki.archlinux.org/index.php/Locale#Setting_system-wide_locale) preferences in `/etc/locale.conf`. Example:

    
    /etc/locale.conf
    LANG=zh_CN.UTF-8
    LC_TIME=en_GB.UTF-8




#### 硬件时间


`/etc/adjtime` 中设置，默认、推荐的设置为[UTC](http://en.wikipedia.org/wiki/Coordinated_Universal_Time)。请在同一个机器使用同一个硬件时钟模式，否则它们会覆盖时间导致时间错乱。 详细信息请阅读[Time (简体中文)](https://wiki.archlinux.org/index.php/Time_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))。 可以用下面命令自动生成 `/etc/adjtime`。


**注意: **请确保`/etc/rc.conf`中未配置 HARDWARECLOCK。


UTC:

    
    # hwclock --systohc --utc




### 安装配置启动加载器




#### Grub 2


安装到 BIOS 主板系统：

    
    # pacman -S grub-bios
    # grub-install --target=i386-pc --recheck /dev/sda


创建`grub.cfg`，要搜索硬盘上安装的其它操作系统，请先用 `# pacman -S os-prober` 安装 [os-prober](https://www.archlinux.org/packages/?name=os-prober)。

    
    # grub-mkconfig -o /boot/grub/grub.cfg




### Root密码


最后，用 `passwd` 设置一个root密码：

    
    # passwd




### 卸载分区并重启系统


如果还在 chroot 环境，先用 exit 命令退出系统：

    
    # exit
    # umount /mnt
    # reboot







## 配置




### 更新




##### 软件包源


**装64位的注意：如果想在 Arch x86_64 上运行 32 位应用程序，请在 /etc/pacman.conf 中加入如下内容以启用 [multilib] 源：**

    
    [multilib]
    Include = /etc/pacman.d/mirrorlist




##### 选择即时更新的镜像


[ArchLinux 镜像状态](http://www.archlinux.org/mirrors/status/) 报告了镜像的各种状态，包括网络问题，数据收集问题、上次同步时间等等。如果需要最新的软件包，最好手动检查一下 `/etc/pacman.d/mirrorlist`，确保文件包含了最新的镜像。

刷新软件包列表：

    
    # pacman -Syy


传入两个 --refresh 或 -y 标记强制 pacman 刷新所有软件包，不管是否被认为是最新的。**只要镜像有变化**就执行 pacman -Syy 是防止出现令人头疼问题的好习惯。


#### 更新系统


同步、刷新、升级整个系统：

    
    #pacman -Syu


或者：

    
    #pacman --sync --refresh --sysupgrade


pacman 会从服务器下载 `/etc/pacman.conf` 中定义的主软件包列表，进行所有可用的升级操作。此时可能会提示说 pacman 自己需要先进性升级，如果这样，请选择是并在完成后重新执行 `pacman -Syu` 。


> 

> 
> ##### Arch 滚动发布模式
> 
> 
请记住 Arch 是 **滚动发布** 的发行版。这意味着升级到新版本不需要重装或者重新建构。只需要定期执行 `pacman -Syu` 就可以将系统保持在最新前沿状态。升级后，系统完全是最新的。如果内核升级了，请记得 **重新启动**。




**警告: 进行系统升级时请小心，执行前请仔细阅读并理解 [这个帖子](https://bbs.archlinux.org/viewtopic.php?id=57205) 。**







### 添加一个用户



    
    adduser grace


要求输入信息，直接回车，到additional grounp的时候输入

    
    lp,audio,video,power,scanner,games,wheel,storage




> _Additional groups_ 中的列表是桌面系统的典型选择，特别推荐给新手：

> 
> 
	
>   * **audio** - 让任务可以调用声卡以及相关软件
> 
	
>   * **lp** - 管理打印任务
> 
	
>   * **optical** - 管理光驱相关任务
> 
	
>   * **storage** - 管理存储设备
> 
	
>   * **video** - 视频任务以及硬件加速
> 
	
>   * **wheel** - 使用 sudo
> 
	
>   * **games** - 得到那些属于游戏组的权限，比如手柄
> 
	
>   * **power** - 笔记本用户需要这个
> 
	
>   * **scanner** - 使用扫描仪
> 




然后继续输入密码。

为了让我们的非root用户也能使用sudo，需要安装sudo接。

    
    pacman -S sudo
    visudo


系统会打开/etc/sudoers文件，找到

    
    #%wheel ALL=(ALL) ALL PASSWD=NO


取消注释，变成

    
    %wheel ALL=(ALL) ALL PASSWD=NO


保存退出，以后普通用户就可以使用sudo暂时取得root权限，并且无须密码。

到这里，系统的基本安装就完成了。

接下来是装X，待续。



参考：[Beginners' Guide](https://wiki.archlinux.org/index.php/Beginners%27_Guide)，[ArchLinux安装与配置 ](http://songtl.com/2012/05/06/archlinux%e5%ae%89%e8%a3%85%e4%b8%8e%e9%85%8d%e7%bd%ae/)
