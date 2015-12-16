---
layout: post
title: OpenWrt MutilSSID Configuration
categories: [blog ]
tags: [internship, ]
description: 
---

实现多个 SSID 需要配置/etc/config 文件夹中的 network、wireless 和 dhcp 三个文件
####wireless 配置
**设备信息,一般不需要改动**  
{% highlight c %}
config wifi-device radio0  
    option type mac80211 //无线架构配置,根据驱动来配置。主要有三种:Broadcom、Atheros、mac80211  
    option channel 11  
    option hwmode 11g  
    option path '10180000.wmac'  
    option htmode HT20  
{% endhighlight %}
**第一个 WIFI,使用 lan**
{% highlight c %}
config wifi-iface  
    option device radio0  
    option network lan  
    option mode ap  
    option ssid OpenWrt  
    option encryption none  
{% endhighlight %}
**第二个 WIFI,使用 lan2**
{% highlight c %}
config wifi-iface
    option device radio0
    option network lan2 #另一个局域网,需要在 network 中配置
    option mode ap
    option ssid OpenWrt2
    option encryption none #常见的加密方式有 psk2、wpa2
    #option key yourpassword
{% endhighlight %}

####network 配置
由于要增加一个局域网 lan2,所以增加一个 vlan3,因此 lan2 的 ifname 就是eth0.3。在端口(port)中,带有 t(tagged)的表示处理器,其他的代表与硬件相连的端口。  
**之前的 lan**
{% highlight c %}
config interface 'lan'
    option ifname 'eth0.1'
    option force_link '1'
    option macaddr '02:c0:d0:00:09:9f'
    option type 'bridge'
    option proto 'static'
    option ipaddr '192.168.1.1'
    option netmask '255.255.255.0'
    option ip6assign '60'
{% endhighlight %}
**新增的 lan2**
{% highlight c %}
config interface 'lan2'     #增加一个 lan
    option ifname 'eth0.3'
    option force_link '1'
    option macaddr '02:c0:d0:00:09:9f'
    option type 'bridge'
    option proto 'static'
    option ipaddr '192.168.2.1'
    option netmask '255.255.255.0'
    option ip6assign '60'
{% endhighlight %}
**vlan 配置**
{% highlight c %}
config switch
    option name 'rt305x'
    option reset '1'
    option enable_vlan '1'

config switch_vlan
    option device 'rt305x'
    option vlan '1'
    option ports '0 1 2 6t'

config switch_vlan
    option device 'rt305x'
    option vlan '2'
    option ports '4 6t'

config switch_vlan  #增加一个 vlan
    option device 'rt305x'
    option vlan '3'
    option ports '3 6t' #使用 3 端口
{% endhighlight %}

####dhcp 配置
**除了 wireless 和 network 之外,还要配置 dhcp 才能实现两个 ssid 独立运行**
{% highlight c %}
config dhcp 'lan'
    option interface 'lan'
    option start '100'
    option limit '150'
    option leasetime '12h'
    option dhcpv6 'server'
    option ra 'server'

config dhcp 'lan2'  #给 lan2 实现和 lan 相同的 dhcp 配置
    option interface 'lan2'
    option start '100'
    option limit '150'
    option leasetime '12h'
    option dhcpv6 'server'
    option ra 'server'
{% endhighlight %}
