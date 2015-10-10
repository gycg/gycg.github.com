---
layout: post
title: linux shell脚本攻略总结（二)
categories: [blog ]
tags: [shell, 建站前 ]
description: shell学习第二弹
---

**1 cat技巧**

{% highlight ruby %}
$ cat -s file	#删除多余的空白行
$ cat -T file	#将制表符显示为^|
$ cat -n file	#在行首加行号打印
{% endhighlight %}
**2 录制并回放终端回话**

录制命令
{% highlight ruby %}
$ script -t 2> timing.log -a output.session
$ type commands
$ ...
$ exit
#timing.log存放时序信息，output.session存储命令输出;-t将时序数据导入stderr。
{% endhighlight %}
回放命令

{% highlight ruby %}
scriptreply timing.log output.sesssion
{% endhighlight %}
**3 文件查找**

{% highlight ruby %}
find . -name "*.txt" -print	#打印当前目录以.txt结尾的文件名

find . -iname "*example.*" -print	#忽略大小写

find . \( -name "*.txt" -o -name "*.pdf" \) -print	#-o表示或（OR）

find /home/users -path "*/gycg/*" -print	#打印含有/gycg/的路径

find . -regex ".*\(\.py\|\.sh\)$"	#打印当前目录以.py或.sh结尾的文件名

find . -iregex ".*\(\.py\|\.sh\)$"	#忽略大小写"

find . ! -name "*.txt" -print	#打印不以.txt结尾的文件名

find . -maxdepth 1 -name "f*" -print	#打印当前目录（1级目录）以f打头的文件名

find . -mindepth 2 -name "f*" -print   #打印深度距离当前目录至少两个子目录的文件名

find . -type d -print	#列出所有目录（d表示目录）
#f：普通文件，l：符号链接，c：字符设备，b：块设备，s：套接字，p：FIFO。

find . -type f -atime -7 -print #打印最近7天被访问过的文件
#-aitme：访问时间，-mtime：修改时间，-ctime：变化时间（元数据改变时间）;单位：天
#-7：7天之内，7：恰好7天，+7：超过7天。
#-amin，-mmin，-cmin，以分钟为单位

find . -type f -newer file.txt  -print	#列出比file.txt修改时间更近的所有文件

find . -type f -size +2k	#列出大于2k的文件
#-2k：小于2k，2k：等于2k;
#b：块（512字节），c：字节，w：字（2字节），k：1024字节，M：1024k字节，G：1024M字节

find . -type f -name "*.swp" -delete	#删除以.swp结尾的文件

find . -type f -name "*.php" ! -perm 644 -print	#打印权限不为644的php文件

find . -type f -user gycg -print	#打印gycg拥有的所有文件

find . -type f -user root -exec chown gycg {} \;	#将root拥有的文件所有者改为gycg

find . -type f -name "*.c" -exec cat {} \; >all_c_files.txt	#将所有c文件拼接起来写入一个文件

find . -type f -mtime +10 -name "*.txt" -exec cp {} OLD \;	#将10天前的.txt文件复制到OLD目录中

find . -type f -name "*.txt" -exec printf "Text file: %s\n" {} \;

find /src \( -name ".git" -prune \) -o \(-type f -print \)	#打印不包括在.git中的所有文件名称
{% endhighlight %}
另外，匹配E-mail地址（name@host.toot），可以将其一般化为[a-z0-9]+@[a-z0-9]+.[a-z0-9]+。符号+指明在它之前的字符类中的字符可以出现一次或多次。

**4 xargs**

{% highlight ruby %}
cat example.txt | xargs		#将多行输入转换成单行输出

cat example.txt | xargs -n 3	#多行输出，每行3个参数，每个参数是以空格分开的字符串

echo "splitXsplitXsplitXsplit" | xargs -d X	#以X为定界符打印

echo "splitXsplitXsplitXsplit" | xargs -d X -n 2	#分两行输出

find . -type f -name "*.txt" -print0 | xargs -0 rm -f	#删除所有.txt文件

find . -type f -name "*.c" -print0 | xargs -0 wc -l	#统计.c文件的行数

find files.txt | ( while read arg; do cat $arg; done )	#等同于cat files.txt | xargs -I {} cat {} 
{% endhighlight %}
**5 tr**

{% highlight ruby %}
echo "HELLO WORLD" | tr 'A-Z' 'a-z'	#转换为小写

echo 12345 | tr '0-9' '9876543210'	#加密
echo 87654 | tr '9876543210' '0-9'	#解密

echo "Hello 123 world 456" | tr -d '0-9'	#删除数字

echo hello 1 char 2  next 4 | tr -d -c '0-9 \n'	#删除除了数字、空格、换行符之外的所有字符

echo "GNU is     not     UNIX. Recursive    right ?" | tr -s ' '	#-s压缩输入中重复的字符

tr '[:lower:]' '[:upper:]'	#小写转换为大写
#alnum，alpha,cntrl,digit,graph,lower,print（可打印字符）,punct(标点)，space，upper，xdigit（十六进制）
{% endhighlight %}
**6 校验和与核实**

{% highlight ruby %}
md5sum filename > file_sum.md5	#将校验和存入文件中
md5sum -c file_sum.md5	#输出校验和是否匹配的消息
{% endhighlight %}
**7 加密工具与散列**

{% highlight ruby %}
crypt PASSPHASE <input_file >encrypted_file	#命令行也可以不输密码，会提示输入密码
crypt PASSPHASE -d <encrypted_file >output_file

gpg -c filename	#加密
gpg filename.gpg	#解密

base64 filename > base64_file #或cat file | base64 > base64_file;加密
base64 -d base64_file > outputfile	#或cat base64_file | base64 -d > outputfile	#解密
{% endhighlight %}
**8 排序**

{% highlight ruby %}
sort -r file.txt	#按逆序排序

sort -k 2 data.txt	#根据第二列进行排序

sort -rk 1 data.txt 	#根据第一列逆序排列

sort -nk 2,3 data.txt	#根据2，3列进行排序，-n用于指明按照数字排序

sort -nk 1,1 data.txt	#用第一个字符作为键

sort -bd unsorted.txt	#-b用于忽略前导空白行，-d用于指明以字典序进行排序

uniq sorted.txt OR sort unsorted.txt | uniq

uniq -u sorted.txt OR sort unsorted.txt | uniq -u	#只显示唯一的行

sort unsorted.txt | uniq -c	#统计各行在文件中出现的次数

sort unsorted.txt | uniq -d	#找出重复的行

sort data.txt | uniq -s 2 -w 2	#忽略前两个字符（-s 2），最多比较两个字符（－ｗ　２）

uniq -z file.txt	#生成包含０值字节终止符的输出

uniq -z file.txt | xargs -0 rm	#删除file.txt中指定的文件
{% endhighlight %}
**9　临时文件**

{% highlight ruby %}
$ filename=`mktemp`
$ echo $filename	#创建一个临时文件，并打印出文件名

$ dirname=`mktemp -d`
$ echo $dirname		#创建一个临时目录，并打印出目录名

$ tempfile=`mktemp -u`
$ echo $tempfile	#文件名存储在tempfile中，但没有创建对应的文件

$ mktemp test.XXX	＃根据模板创建文件名，至少有３个Ｘ
{% endhighlight %}
**10　分割文件**

{% highlight ruby %}
split -b 10k data.file	#将data.file分割成多个文件，每个１０ｋ,默认命名x**,"*"为字母，如xaa,xab...

split -b 10k data.file -d -a 4	#－ｄ指明以数字命名，－ａ　４指明数字长度为４,前缀为ｘ

split -b 10k data.file -d -a 4　split_file	#设置前缀为split_file

split -l 10 data.file	#分割成多个文件，每个文件包含１０行
{% endhighlight %}
csplit是split的一个变体，示例如下：
{% highlight ruby %}
$ cat server.log
SERVER-1
[connection] 192.168.0.1 success
[connection] 192.168.0.2 failed
[disconnect] 192.168.0.3 pending
[connection] 192.168.0.4 success
SERVER-2
[connection] 192.168.0.1 failed
[connection] 192.168.0.2 failed
[disconnect] 192.168.0.3 success
[connection] 192.168.0.4 failed
SERVER-3
[connection] 192.168.0.1 pending
[connection] 192.168.0.2 pending
[disconnect] 192.168.0.3 pending
[connection] 192.168.0.4 failed
{% endhighlight %}
我们需要将这个日志文件分割成server1.log、server2.log和server3.log，这些文件的内容分别取自原文件中不同的SERVER部分。那么，可以使用下面的方法来实现：
{% highlight ruby %}
$ csplit server.log /SERVER/ -n 2 -s {*}  -f server -b "%02d.log"  ; rm server00.log
　
$ ls
server01.log  server02.log  server03.log  server.log
{% endhighlight %}
有关这个命令的详细说明如下。

    /SERVER/用来匹配某一行，分割过程即从此处开始。

    /[REGEX]/表示文本样式。包括从当前行（第一行）直到（但不包括）包含“SERVER”的匹配行。

    {*}表示根据匹配重复执行分割，直到文件末尾为止。可以用{整数}的形式来指定分割执行的次数。

    -s使命令进入静默模式，不打印其他信息。

    -n指定分割后的文件名后缀的数字个数，例如01、02、03等。

    -f指定分割后的文件名前缀（在上面的例子中，server就是前缀）。

    -b指定后缀格式。例如%02d.log，类似于C语言中printf的参数格式。在这里文件名=前缀+后缀=server + %02d.log。

因为分割后的第一个文件没有任何内容（匹配的单词就位于文件的第一行中），所以我们删 除了server00.log。 

**11 根据扩展名切分文件**

{% highlight ruby %}
$ URL="www.google.com"
$ echo ${URL%.*}	#移除．*所匹配的最右边的内容
www.google

$ echo ${URL%%.*}	#将从右边开始一直匹配到最左边的．＊移除（贪婪操作符）
www

$ echo ${URL#*.}	#移除＊．所匹配的最左边的内容
google.com

$ echo ${URL##*.}	#将从左边开始一直匹配到最右边的＊．移除（贪婪操作符）
com
{% endhighlight %}

**12 重命名**

{% highlight ruby %}
#!/bin/bash
#文件名：rename.sh
#用途：重命名.jpg和.png文件

count=1;
for img in `find . -iname '*.png' -o -iname '*.jpg' -type f -maxdepth 1`
do
	new==image-$count.${img##*.}
	
	echo "Renaming $img too $new"
	mv "$img" "$new"
	let count++
done
{% endhighlight %}
**13 拼写检查和词典操作**

{% highlight ruby %}
#!/bin/bash
#文件名：checkword.sh
word=$1
grep "^$1$" /usr/share/dict/british-english -q
#^标记着单词的开始，$标记着单词的结束，－ｑ禁止产生任何输出
if [ $? -eq 0 ]; then
	echo $word is a dictionary word;
else
	echo $word is not a dictionary word;
fi
{% endhighlight %}

{% highlight ruby %}
#!/bin/bash
#文件名：aspellcheck.sh

word=$1

output=`echo \"$word\" | aspell list`
#当输入的不是一个词典单词，aspell list产生文本输出，反之不产生任何输出

if [ -z $output ]; then	#确认$output是否为空
	echo $word is a dictionary word;
else
	echo $word is not a dictionary word;
fi
{% endhighlight %}

{% highlight ruby %}
$ look word filepath OR look "^word" filepath	#列出文件中以特定单词开头的所有单词，如果没有给出文件参数，则使用默认词典（/usr/share/dict/words）
{% endhighlight %}
**14 交互输入自动化**

{% highlight ruby %}
#!/bin/bash
#文件名：interctive.sh
read -p "Enter number:" no;
read -p "Enter name:" name;
echo You have entered $no,$name;
{% endhighlight %}

{% highlight ruby %}
$ echo -e "1\nhello\n" | bash interctive.sh
You have ectered 1,hello

$ echo -e "1\nhello\n" > input.data
$ bash interctive.sh < input.data
{% endhighlight %}

{% highlight ruby %}
#!/usr/bin/expect
#文件名：automate_expect.sh
spawn bash interactive.sh	#指定自动化哪个命令
expect "Enter number:"		#等待的消息
send "1\n"			
#发送的消息
expect "Enter name:"
send "hello\n"
expect eof			#命令交互结束
{% endhighlight %}

**15　利用并行进程加速命令执行**

{% highlight ruby %}
#!/bin/bash
#文件名：generate_checksums.sh
PIDARRAY=()
for file in File1.iso File2.iso
do
	md5sum $file &
	PIDARRAY+=("$!")
done
wait ${PIDARRY[@]}
#多个md5sum命令是同时运行的，更快获得运行结果
{% endhighlight %}
