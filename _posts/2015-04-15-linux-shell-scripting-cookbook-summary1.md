---
layout: post
title: linux shell脚本攻略总结（一)
categories: [blog ]
tags: [shell, ]
description: 小试牛刀
---

**1 终端彩色输出**

{% highlight bash %}
$ echo -e "\e[1;31m This is red text \e[0m"
{% endhighlight %}
这段脚本输出红色的字符串“This is red text”；

-e表示转换双引号中的转义字符；

\e[1;31m表示将文本颜色设置为红色，其中1指的是字体，31指的是红色；

表示文本的还有以下颜色：重置=0，黑色=30，红色=31，绿色=32，黄色=33，蓝色=34，洋红=35，青色=36，白色=37。

\e[0m表示还原颜色。
{% highlight bash %}
$ echo -e "\e[1;42m Green Background \e[0m"
{% endhighlight %}
这段代码输出绿色背景的字符串“Green Background”；
表示背景的还有以下颜色：重置=0，黑色=40，红色=41，绿色=42，黄色=43，蓝色=44，洋红=45，青色=46，白色=47。

**2 数学运算高级工具bc**

可以借助bc执行浮点数运算，并应用一些高级函数。
{% highlight bash %}
$echo "4 * 0.56" | bc
2.24

$ no=54;
$result=`echo "$no * 1.5" | bc`
$ echo $result
81.0
{% endhighlight %}
设定小数精度

{% highlight bash %}
$ echo "scale=2;3/8" | bc
0.37
{% endhighlight %}
进制转换

{% highlight bash %}
$ no=100
$ echo "obase=2;$no" | bc #obase表示输出进制
1100100
$ no=1100100
$ echo "obase=10;ibase=2;$no" | bc #ibase表示输入进制
100
{% endhighlight %}
计算平方&平方根

{% highlight bash %}
$ echo "sqrt(100)" | bc
$ echo "10^10" | bc
{% endhighlight %}

**3 检查一段代所码花费时间**

{% highlight bash %}
#!/bin/bash
#文件名：time_taken.sh
start=$(date +%s)
#command;
#statements;

end=$(date +%s)
difference=$(( end - start ))
echo Time taken to execute commands is $difference seconds.
{% endhighlight %}

**4 输出计数**

{% highlight bash %}
#!/bin/bash
#文件名：sleep.sh
echi=o -n Count:
tput sc	#存储光标位置

count=0;
while true;
do
	if [ $count -lt 40 ];
	then
		let count++;
		sleep 1;
		tput rc #恢复光标位置
		tput ed #清楚当前光标位置到行尾之间的所有内容
		echo -n $count;
	else exit 0;
	fi
done
{% endhighlight %}

**5 用read读取输入**

{% highlight bash %}
$ read -n 2 var #读取2个字符存入var中
$ read -s var #无回显方式输入
$ read -p "Please enter input:" var #提示信息
$ read -t 2 var #在2秒内将键入的字符读入变量var
$ read -d ":" var #以冒号“：”作为结束
{% endhighlight %}

**6 运行命令直至执行成功**

{% highlight bash %}
repeat() { while :; do $@ && return; sleep 30; done }
{% endhighlight %}
用冒号“：”而不用true是因为true一般作为二进制文件来实现，每次执行都会生成一个进程。增加延时sleep 30防止发送数据过于频繁。

**7 内部字段分隔符IFS**

内部字段分隔符（Internal Field Separator，IFS）是用于特定用途的定界符。IFS变量是存储定界符的环境变量。

{% highlight bash %}
#!/bin/bash
data="1,2,3,4"
oldIFS=$IFS
IFS=,
for item in $data;
do
    echo Item: $item
done

IFS=$oldIFS
{% endhighlight %}

**8 比较与测试**

算数比较
{% highlight bash %}
-eq     相等
-ne		不相等
-gt		大于
-lt		小于
-ge		大于或等于
-le		小于或等于
{% endhighlight %}

文件系统相关测试
	

{% highlight bash %}
-f      正常文件路径或文件名，则返回真
-d		目录，则返回真
-e		文件存在，则返回真
-c		字符设备文件，则返回真
-b		块设备文件，则返回真
-x		文件可执行，则返回真
-w		文件可写，则返回真
-r		文件可读，则返回真
-L		符号链接，则返回真
{% endhighlight %}
