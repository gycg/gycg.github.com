---
layout: post
title: 为Ubuntu配置shadowsocks
categories: [blog ]
tags: [Ubuntu, ]
description:
--- 

####安装shadowsocks-libev,构建deb  
{% highlight bash %}
cd /usr/local/src
sudo git clone https://github.com/shadowsocks/shadowsocks-libev.git
sudo apt-get install -y build-essential autoconf libtool libssl-dev gawk debhelper dh-systemd init-system-helpers pkg-config nano
cd shadowsocks-libev
sudo dpkg-buildpackage -us -uc -i
cd ..
sudo dpkg -i shadowsocks-libev*.deb
{% endhighlight %}
####建立配置文件  
编辑/etc/shadowsocks.json文件，将配置信息填入相应的位置  
{% highlight bash %}
sudo vim /etc/shadowsocks.json
{
    "server":"45.35.*.*",
    "server_port":39896,
    "local_port":1080,
    "local_address":"127.0.0.1",
    "password":"64028872",
    "timeout":300,
    "method":"aes-256-cfb"
}
{% endhighlight %}
"server"是服务器IP地址，"server_port"是其端口号，"password"是密码。这个可以在www.ss-link.com中购买或试用。  
然后就可以运行命令
{% highlight bash %}
sslocal -c /etc/shadowsocks.json
{% endhighlight %}
来开启代理。这时就可以访问外网了。（不过我的FireFox浏览器还需要设置将代理模式设置成Global Mode）。  
####设置开机自启  
在/etc/rc.local中exit 0之前加一句
{% highlight bash %}
nohup /usr/local/bin/sslocal -c /etc/shadowsocks.json &
{% endhighlight %}
就可以使其开机自启动。输出会重定向到一个名叫nohup.out的文件中。  
如果不想产生输出文件，可以使用如下命令：
{% highlight bash %}
nohup /usr/local/bin/sslocal -c /etc/shadowsocks.json >/dev/null 2>&1 &
{% endhighlight %}
其中>/dev/null是指将标准输出重定向到null中，这就相当于删除，然后2>&1是指将标准错误重定向到标准输出，也就是同样删除。  
至此就完成了Ubuntu中shadowsocks的配置，可以科学上网了。
