---
layout: post
title: WifiDog的安装和配置
categories: [blog ]
tags: [internship, ]
description: 
---

WifiDog是路由器的一种上网认证功能,如果开启此功能,所有通过路由器上网的设备都会跳转到指定的界面,需要通过某种方式认证才可以上网。  
###安装 WifiDog
####通过内核配置来选择WifiDog相关选项
在源代码文件夹中输入命令#make menuconfig进入OpenWrt的配置界面,进入Network--->Captive Portals,将wifidog配置为<*>(built-in)。然后通过#make V=s进行编译。完成后,将bin/ramips目录下面的openwrt-ramips-rt305x-mpr-a2-squashfs-sysupgrade.bin拷进/var/lib/tftpboot中进行烧录。(在烧写固件过程中遇到的Arptimeout问题,有可能是tftp的bug。解决办法是完全卸载tftp、tftpd-hpa等软件,并且删除/var/lib/tftpboot文件夹,重新安装。)  
####直接从安装源进行安装
如果 OpenWrt 连接了网络,可以在系统中通过以下命令进行安装。
{% highlight bash %}
#opkg update
#opkg install wifidog
{% endhighlight %}
###申请 wiwiz 账号或部署 AuthPuppy
Wifidog 只是实现 AP 认证网关,需要配合外部的 Portal 服务器才能使用,可以部署一个 AuthPuppy,不过如果为了测试方便,可以直接申请一个 wiwiz 的账号。进入 http://cp.wiwiz.com 进行注册激活,然后就可以创建一个 HotSpot,记录下得到的 HotSpot ID。
###配置 WifiDog
登陆 OpenWrt 系统,编辑/etc/wifidog.conf 文件,加入以下配置:
{% highlight c %}
GatewayID XXXXXXXXXXX
AuthServer {
Hostname cp.wiwiz.com
Path /as/s/
}
{% endhighlight %}
其中,XXXXXXXXXXX 就是刚才得到的 11 为 HotSpot ID。  
配置完成后,重启路由器,然后用终端通过路由器访问互联网,就会弹出 wiwiz 的认证页面。注意一定要将路由器的网口与互联网相连。
