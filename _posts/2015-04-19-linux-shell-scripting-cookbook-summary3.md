---
layout: post
title: linux shell 脚本攻略总结（三）
categories: [blog ]
tags: [shell, ]
description: 以文件之名
---

###1 生成任意大小的文件  
{% highlight bash %}
$ dd if=/dev/zero of=junk.data bs=1M count=1
1+0 records in
1+0 records out
1048576 bytes (1.0 MB) copied, 0.00277712 s, 378 MB/s
{% endhighlight %}
if代表输入文件（input file），of代表输出文件（output file），bs代表以字节为单位的块大小（block size），count代表需要被复制的块数。  
**块大小的计量单位**  

|  单元大小         |  代码  |
|  :-------         |  :---: |
|  字节（1B）       |  c     |
|  字（2B）         |  w     |
|  块（512B）       |  b     |
|  千字节（1024B）  |  k     |
|  兆字节（1024KB） |  M     |
|  吉字节（1024MB） |  G     |

###2 文本文件的交集与差集  
{% highlight bash %}
$ cat A.txt
apple
orange
gold
silver
steel
iron

$ cat B.txt
orange
gold
cookies
carrot

$ sort A.txt -o A.txt ; sort B.txt -o B.txt

$ comm A.txt B.txt 
apple
    carrot
    cookies
        gold
iron
        orange
silver
steel

$ comm A.txt B.txt -1 -2    #删除第一二列
gold
orange

$ comm A.txt B.txt -3       #删除第三列
apple
    carrot
    cookies
iron
silver
steel

$ comm A.txt B.txt -3 | sed 's/^\t//'
apple
carrot
cookies
iron
silver
steel
#sed中的s表示替换（substitute）。/^\t/匹配行前的\t（^是行首标记）。//（两个/操作符之间没有任何字符）是用来替换行首的\t的字符串。
{% endhighlight %}

###3 查找并删除重复文件
{% highlight bash %}
# !/bin/bash
# 文件名： remove_duplicates.sh
# 用途： 查找并删除重复文件， 每一个文件值保留一份

ls -lS --time-style=long-iso | awk 'BEGIN {   
    getline; getline;  
    name1=$8; size=$5  
}  
{  
    name2=$8;  
    if (size==$5)  
    {  
        "md5sum "name1 | getline; csum1=$1;  
        "md5sum "name2 | getline; csum2=$1;  
        if ( csum1==csum2 )  
        {  
            print name1; print name2  
        }  
    };  
    
    size=$5; name1=name2;  
}' | sort -u > duplicate_files  

cat duplicate_files | xargs -I {} md5sum {} | sort |uniq -w 32 | awk '{ print "^"$2"$" }' | sort -u > duplicate_sample

echo Removing..
comm duplicate_files duplicate_sample -2 -3 | tee /dev/stderr | xargs rm
echo Removed duplicates files successfully.
{% endhighlight %}
ls -lS对当前目录下的所有文件按照文件大小进行排序，并列出文件的详细信息。awk读取其输出，对行列进行比较，找出重复文件。  

首先执行BEGIN{}语句块。ls -lS的输出第一行是文件数，所以直接getline丢弃。再用getline读取第二行。储存文件名和大小，分别是第8列和第5列。  

然后执行{}语句块，读取其余文本行。将当前行读取到的文件大小与之前储存在变量size中的值进行比较。如果相等，再用md5sum进行进一步的检查。  

在awk中，外部命令的输出可以用 "cmd" | getline 获取，这样$0就是输出，$1,$2,$3,...就是输出中的每一列。将文件的md5sum保存在变量csum1和csum2中，如果相等，就打印文件名到duplicate\_files。  

通过-w 32比较每一行md5sum（md5sum通常由32个字符组成），然后找出不相同的行，这样，每组重复文件中的一个采用就被写入duplicate\_sample。  

然后将duplicate\_files中列出的，且未包含在duplicate\_sample之内的全部文件删除。由于comm只接受排序后的文件，所以在重定向到duplicate\_files和duplicate\_sample之前，首先用sort -u作为一个过滤器。  

tee将文件名传递给rm命令的同时，也起到了print的作用。tee将来自stdin的行写入文件，同时将其发送到stdout。  

###4 文件权限、所有权和粘滞位
文件的用户权限（权限序列：rwx------）如果设置了setuid（S）位（rwS------），就意味着允许用户以其拥有者的权限执行可执行文件，即使这个可执行文件是由其他用户运行的。  

文件的用户组权限（权限序列：---rwx---）如果设置了setgid（S）位（---rwS---），就意味着允许以同该目录拥有者所在组相同的有效组权限来执行可执行文件。但是这个组和实际发起命令的用户组未必相同。  

目录有一个特殊的权限，叫粘滞位（sticky bit）。如果设置了粘滞位，只有创建该目录的用户才能删除目录中的文件，即使用户组和其他用户也有写权限，也无能为力。如果没有设置执行权限，但设置了粘滞位，就使用t（------rwt）;如果同时设置了执行权限和粘滞位，就使用T（------rwT）。  

{% highlight bash %}
$ chown user.group filename #更改所有权（用户和用户组）
$ chmod a+t directory_name #设置粘滞位
$ chmod 777 . -R    #选项-R指定以递归的方式修改权限
$ chmod 777 "$pwd" -R   #同上
$ chown user.group . -R #以递归的方式设置所有权
{% endhighlight %}

###5 创建不可修改的文件
{% highlight bash %}
# chattr +i file    #将文件设置为不可修改
# rm file
rm: cannot remove ‘file’: Operation not permitted 
# chattr -i file    #移除不可修改属性
{% endhighlight %}

###6 批量生成空白文件
touch命令可以用来生成空白文件或是修改文件的时间戳（如果文件已存在）。
{% highlight bash %}
$ touch filename    #生成名为filename的空白文件

for name in {1..100}.txt    #{1..100}会扩展成一个字符串“1,2,3,4,5,6,7,...,100”
do
    touch $name
done                
{% endhighlight %}
如果文件以经存在，那么touch命令会将与该文件相关的所有时间戳都更改为当前时间。如果我们只想更改某些时间戳，则可以使用下面的选项。  
->touch -a 只更改文件访问时间  
->touch -m 只更改文件内容修改时间  

还可以将时间戳指定特定的时间和日期：  
{% highlight bash %}
touch -d "Fri Jun 25 20:50:14 IST 1999" filename
{% endhighlight %}

###7 查找符号链接及其指向目标
{% highlight bash %}
$ ln -s target symbolic_link_name #创建符号链接
$ ls -l symbolic_link_name  #验证是否创建链接
$ ls -l | grep "^l"     #打印出当前目录下的符号链接
$ find . -type l -print #打印出当前目录以及子目录下的符号链接
$ readlink symbolic_link_name   #打印出符号链接所指向的目标路径
{% endhighlight %}

###8 列举文件类型统计信息
{% highlight bash %}
$ file /etc/passwd
/etc/passwd: ASCII text
$ file -b /etc/passwd
ASCII text
{% endhighlight %}
**生成文件统计信息的脚本**
{% highlight bash %}
# !/bin/bash
# 文件名： filestat.sh

if [ $# -ne 1 ];
then
    echo "Usage is $0 basepath";
    exit;
fi
path=$1

declare -A statarry;

while read line;
do
    ftype=`file -b "$line" | cut -d, -f1`
    let statarry["$ftype"]++;

done < <(find $path -type f -print)

echo =========== File types and counts ===========
for ftype in "${!statarry[@]}";
do
    echo $ftype : ${statarry["$ftype"]}
done
{% endhighlight %}

###9 使用环回文件
环回文件系统是指那些在文件中而非物理设备中创建的文件系统。我们可以将这些文件作为文件系统挂载到挂载点上。  
{% highlight bash %}
$ dd if=/dev/zero of=loopbackfile.img bs=1G count=1 #这个命令可以创造一个1GB大小的文件
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB) copied, 9.30371 s, 115 MB/s

$ mkfs.ext4 loopbackfile.img    #用mkfs命令将1GB的文件格式化为ext4文件系统

$ file loopbackfile.img     #检查文件系统
loopbackfile.img: Linux rev 1.0 ext4 filesystem data, UUID=f82122c4-994c-46eb-9f6a-aeec192defb1 (extents) (large files) (huge files)

$ sudo mkdir /mnt/loopback
$ sudo mount -o loop loopbackfile.img /mnt/loopback/    #挂载环回文件系统
#另一种方法，手动挂载
$ sudo losetup /dev/loop1 loopbackfile.img
$ mount /dev/loop1 /mnt/loopback/

$ sudo umount /mnt/loopback/    #卸载挂载点
$ sudo umount /dev/loop1    #也可以用设备文件路径作为参数

$ sync  #当对挂载设备作出更改之后，这些改变并不会被立即写入物理设备。只有当缓冲区被写满之后才会进行设备回写。但是我们可以用sync命令强制将更改即刻写入。
{% endhighlight %}

###10 生成ISO文件及混合型ISO
{% highlight bash %}
# cat /dev/cdrom > image.iso    #创建ISO镜像
# dd if=/dev/cdrom of=image.iso #更好的方法
$ mkisofs -V "Label" -o image.iso source_dir/   #创建ISO文件系统：选项-o指定了ISO文件的路径。source_dir/是作为ISO文件内容来源的目录路径，选项-V指定了ISO文件的卷标
{% endhighlight %}
**能够启动闪存或硬盘的混合型ISO**
{% highlight bash %}
# isohybrid image.iso   #把标准ISO文件转换成混合ISO，可用于写入USB存储设备
# dd if=image.iso of=/dev/sdb1  #将该ISO写入USB存储设备
# cat image.iso >> /dev/sdb1    #同上
{% endhighlight %}
**用命令行刻录ISO**
{% highlight bash %}
# cdrecord -v dev=/dev/cdrom image.iso  #刻录CD-ROM的方法
# cdrecord -v dev=/dev/cdrom image.iso -speed 8 #参数8表明其刻录速度为8x。
# cdrecord -v dev=/dev/cdrom image.iso image.iso -multi #多区段方式刻录,一张光盘上分多次刻录数据$ eject #弹出光驱托盘
{% endhighlight %}

###11 查找文件差异并进行修补
{% highlight bash %}
$ vim version1.txt

this is the original text
line2
line3
line4
happy hacking!

$ vim version2.txt

this is the original text
line2
line4
happy hacking!
GNU is not UNIX

$ diff version1.txt version2.txt    #非一体化输出
3d2
< line3
5a5
> GNU is not UNIX

$ diff -u version1.txt version2.txt     #一体化输出
--- version1.txt    2015-12-10 18:37:16.186256916 +0800
+++ version2.txt    2015-12-10 18:36:14.562258532 +0800
@@ -1,5 +1,5 @@
 this is the original text
 line2
-line3
 line4
 happy hacking!
+GNU is not UNIX

$ diff -u version1.txt version2.txt > version.patch     #重定向到一个文件
$ patch -p1 version1.txt < version.patch    #修补，之后version1.txt的内容就和version2.txt一模一样
patching file version1.txt
$ patch -p1 version1.txt < version.patch    #撤销修改
patching file version1.txt
Reversed (or previously applied) patch detected!  Assume -R? [n] y
{% endhighlight %}
**生成目录的差异信息**
{% highlight bash %}
$ diff -Naur directory1 directory2
{% endhighlight %}
-N:将所有缺失的文件视为空文件 
-a:将所有文件视为文本文件
-u:生成一体化输出
-r:遍历目录下的所有文件

###12 使用head与tail打印文件的前10行和后10行
{% highlight bash %}
$ head file #打印文件前10行
$ cat text | head   #打印文件前10行
$ head -n 4 file    #打印前4行
$ head -n -5 file   #打印除了后5行之外的所有行
$ tail file #打印文件最后10行
$ cat text | tail   #打印文件最后10行
$ tail -n 5 file    #打印最后5行
$ tail -n +(M+1)    #打印除了前M行之外所有的行
{% endhighlight %}

###13 只列出目录的各种方法
{% highlight bash %}
$ ls -d */
$ ls -F | grep "/$"
$ ls -l | grep "^d"
$ find . -maxdepth 1 -type d -print
{% endhighlight %}

###14 在命令行中使用pushd和popd进行快速定位
{% highlight bash %}
~$ pushd /var   #在～目录下将/var压入栈
/var ~
$ pushd /usr/src/   #再将/usr/src/压入栈
/usr/src /var ~
$ dirs      #列出栈中的目录
/usr/src /var ~ #从0开始编号
$ pushd +2      #第二个就是～目录，所有切入～目录
~ /usr/src /var
~$ popd     #删除当前路径
/usr/src /var
$ popd +1   删除编号为1的路径
/usr/src

$ cd -  #切换到上一个路径
{% endhighlight %}

###15 统计文件的行数、单词数和字符数
wc是一个用于统计的工具，它是Word Count的缩写。
{% highlight bash %}
$ wc -l file        #统计行数
$ cat file | wc -l  #统计行数
$ wc -w file        #统计单词数
$ cat file | wc -w  #统计单词数
$ wc -c file        #统计字符数
$ cat file | wc -c  #统计字符数
$ wc    #分别打印出文件的行数、单词数和字符数
$ wc file -L    #打印出文件中最长一行的长度
{% endhighlight %}

###打印目录树
tree命令以图形化的树状结构打印文件和目录。
{% highlight bash %}
$ tree /tmp #打印/tmp目录的树状结构
.
├── A.txt
├── config-err-5alb5a
├── evince-9062
│   ├── image.IYH38X.png
│   ├── image.RHS88X.png
│   ├── image.RICK9X.png
│   ├── image.W3398X.png
│   ├── image.W5K28X.png
│   └── image.ZKFJ9X.png
├── fcitx-socket-:0
├── hsperfdata_gycg
├── orbit-gycg
├── orbit-gycg-2828a868
├── text
├── text~
├── unity_support_test.0
├── version1.txt
├── version2.txt
└── version.patch

4 directories, 15 files

$ tree path -P PATTERN  #用通配符描述样式
$ tree path -I PATTERN  #重点标记出符合某种样式之外的文件
$ tree -h   #同时打印出文件和目录的大小

$ tree PATH -H http://localhost -o out.html
#创建一个包含目录树的输出的HTML文件
{% endhighlight %}
