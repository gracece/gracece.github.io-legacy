---
author: admin
comments: true
date: 2012-11-03 04:44:29+00:00
layout: post
slug: openwrt-settings-note
title: openwrt 配置备忘
wordpress_id: 576
categories:
- linux
tags:
- linux
- openwrt
---

暂时还不好编译固件，先用着别人的。

1、更改root密码，打开无线，安装njit-client，连上网。

njit-client开机启动 /etc/init.d/xclient

	#!/bin/sh /etc/rc.common
	#(c) 2012 grace
	START=50

	start() {
	njit-client 你的账号 你的密码 eth1.1 &;
	}

	stop()
	{
	killall njit-client
	killall udhcpc
	}

然后 

	chmod +x /etc/init.d/xclient  
	/etc/init.d/xclient enable   






2、安装luci-theme-bootstrap

	cd /tmp
	wget http://nut-bolt.nl/files/luci-theme-bootstrap_1-1_all.ipk
	opkg install luci-theme-bootstrap_1-1_all.ipk

[link](http://nut-bolt.nl/2012/openwrt-bootstrap-theme-for-luci/)

3、[为OpenWrt的luci Web界面加速](http://www.vinoca.org/2012/09/07/%e4%b8%baopenwrt%e7%9a%84luci-web%e7%95%8c%e9%9d%a2%e5%8a%a0%e9%80%9f/)

	opkg install uhttpd-mod-lua luci-sgi-uhttpd
	
4、防火墙打开80端口，实现远程访问luci

端口转发到电脑，实现远程连接，远程连接端口为3389

dhcp 去掉域名解析过滤，不然不能解析到局域网ip。

### 2013-09-30补充

关于域名解析本地ip过滤，后来发现不是所有的固件版本都能在 `网络->DHCP和DNS` 找到`域名解析过滤`并去掉勾选，或者有可能去掉勾选之后仍不生效，这时候可以尝试直接更改配置文件

	sed -i 's/option rebind_protection 1/option rebind_protection 0/'  /etc/config/dhcp

具体说明可参见 [狼王的博文](http://chliny.me/?p=104171) 。
