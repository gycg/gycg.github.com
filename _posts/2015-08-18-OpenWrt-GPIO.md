---
layout: post
title: OpenWrt GPIO编程
categories: [blog ]
tags: [internship, ]
description: 
---

####在 OpenWrt 系统中,可以通过/sys/class/gpio 文件夹来对 GPIO 进行操作,/sys/class/gpio 的使用说明
1、gpio_operation 通过/sys/文件接口操作 IO 端口 GPIO 到文件系统
的映射;  

2、控制 GPIO 的目录位于/sys/class/gpio;  

3、/sys/class/gpio/export 文件用于通知系统需要导出控制的 GPIO 引脚编号;  

4、/sys/class/gpio/unexport 用于通知系统取消导出;  

5、/sys/class/gpio/gpiochipX 目录保存系统中 GPIO 寄存器的信息,包括每个寄存器控制引脚的起始编号 base,寄存器名称,引脚总数 导出一个引脚的操作步骤;  

6、首先计算此引脚编号,引脚编号 = 控制引脚的寄存器基数 + 控制引脚寄存器位数;  

7、向/sys/class/gpio/export 写入此编号,比如 12 号引脚,在 shell 中可以通过以下命令实现,命令成功后生成/sys/class/gpio/gpio12 目录, 如 果 没 有 出 现 相 应 的 目 录 , 说 明 此 引 脚 不 可 导 出 : echo 12 > /sys/class/gpio/export;  

8、direction 文件,定义输入输入方向,可以通过下面命令定义为输出;  

9、echo out > direction, direction 接受的参数:in, out, high, low。high/low 同时设置方向为输出,并将 value 设置为相应的 1/0;  

10、value 文件是端口的数值,为 1 或 0。echo 1 > value  

####例如:控制 LED 灯的亮和灭(引脚为 19 号),可以通过以下命令:
{% highlight bash %}
$ echo out > /sys/class/gpio/gpio19/direction
{% endhighlight %}
设置 19 号引脚为输出
{% highlight bash %}
$ echo 1 > /sys/class/gpio/gpio19/value #设置参数为 1,即让 LED 亮
$ echo 0 > /sys/class/gpio/gpio19/value #让 LED 灭
{% endhighlight %}
