---
layout: post
title: Luci Configuration
categories: [blog ]
tags: [internship, ]
description: 
---

LUCI的设计基于MVC理念，即Model，View，Controller，分别在/usr/lib/lua/luci文件夹中的model，view，controller中进行配置。view的配置较为复杂，所以可以将要建立的页面放在其他界面之下，只需配置model和controller即可。  
####建立一个controller
编辑controller/mycbi.lua  
{% highlight c %}
module("luci.controller.mycbi", package.seeall)
function index()
    entry({"admin", "network", "mycbi_change"}, 
            cbi("mycbi-model/mycbimodule"), 
            "Change My Conf", 30).dependent=false
end
{% endhighlight %}
entry第一个参数为web路径，即mycbi放在了主菜单的network之下，所以无需单独创建view。第二个参数为Model路径。第三个参数Change My Conf为链接标题。  
####创建Model
{% highlight bash %}
#mkdir /usr/lib/lua/luci/model/cbi/mycbi-model
#vim /usr/lib/lua/luci/model/cbi/mycbi-model/mycbimodule.lua
m = Map("mycbi", "mycbi conf change interface")
s = m:section(TypedSection, "MySection")
s.addremove = true
s:option(Value, "username", "Name")
key=s:option(Value, "password", "Password")
key.password=true;
return m
{% endhighlight %}
####创建配置文件
{% highlight bash %}
#vim /etc/config/mycbi
config  'MySection'  'mycbi'
    option 'username' 'youruser'
    option 'password' 'yourpass'
{% endhighlight %}

####进行测试
配置完之后，重启路由器，连接路由器wifi，进入192.168.1.1,点击进入network界面，就会发现设置的页面Change My Conf，进入之后，有Name和Password选项，在其中填入要设置的用户名和密码，点击保存并应用，即可更改设置。在路由器的/etc/config/mycbi中查看，可以看到刚才设置的用户名和密码。  
其他页面的设置与此类似，都可以在/usr/lib/lua/luci中进行配置。  

参考：[开发OpenWrt路由器上LuCI的模块](http://www.cnblogs.com/mayswind/p/3468124.html?utm_source=tuicool)
