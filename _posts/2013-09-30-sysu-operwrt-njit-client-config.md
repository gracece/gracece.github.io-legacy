---
layout: post
title: "RG100A-AA 中大校园网快速上网配置"
description: "本文非技术探讨，关于技术细节的探讨可移步[zhyaof](http://talk.withme.me/?p=30)。  "
category: linux
tags: 
- openwrt

---
{% include JB/setup %}

本文非技术探讨，关于技术细节的探讨可移步[zhyaof](http://talk.withme.me/?p=30)。
本文假设你已经通过淘宝或其它途径获得一部已经刷好 Openwrt 的上海贝尔 RG100A-AA，下面的简单步骤能让你快速连接上校园网。以下配置在S6测试通过。其他型号的解决方案可能需要使用不同的解决方法，如
DB-120可以使用OH3C，请查看[南浦月博文](http://blog.nanpuyue.com/2012/003.html)。

### 配置network

	config 'interface' 'loopback'
		option 'ifname' 'lo'
		option 'proto' 'static'
		option 'ipaddr' '127.0.0.1'
		option 'netmask' '255.0.0.0'

	config 'switch' 'eth1'
		option 'reset' '1'
		option 'enable_vlan' '1'

	config 'switch_vlan'
		option 'device' 'eth1'
		option 'vlan' '0'
		option 'ports' '0 1 2 5*'

	config 'switch_vlan'
		option 'device' 'eth1'
		option 'vlan' '1'
		option 'ports' '3 5*'

	config 'interface' 'lan'
		option 'type' 'bridge'
		option 'ifname' 'eth1.0'
		option 'proto' 'static'
		option 'netmask' '255.255.255.0'
		option 'nat' '1'
		option 'ipaddr' '192.168.1.1'

	config 'interface' 'wan'
		option 'ifname' 'eth1.1'
		option 'proto' 'dhcp' 

复制上面的文本（[Raw](https://gist.github.com/gracece/1b068090411fd4d9b4e6/raw/12571386d2ca99bde898f53f2073d4e98b5dc05d/openwrt+network++config) )，并覆盖到 `/etc/config/network` ，重启网络服务，可能会断开ssh连接。

### 安装libpcap

libcap是njit-client 的依赖包，在[这里](http://pan.baidu.com/s/15QqVn)下载 cp.ipk 并传输到路由器内，使用opkg install 安装。

### 安装njit-client 
在[这里](http://pan.baidu.com/s/15QqVn)下载 nj.ipk,传输到路由器中，使用opkg install 安装。

### 测试连接校园网
网线接lan4,在ssh终端中输入
	
	njit-client 你的账号 你的密码 eth1.1 & 

看是否能正常拨号，并测试网页是否能打开。

### 简易开机自动拨号脚本

已经在[openwrt 配置备忘](http://gracece.net/2012/11/openwrt-settings-note/)中说明，请移步查看。


以上就是所有配置步骤。

