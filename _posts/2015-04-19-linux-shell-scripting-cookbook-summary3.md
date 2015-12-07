---
layout: post
title: linux shell 脚本攻略总结（三）
categories: [blog ]
tags: [shell, ]
description: shell学习第三弹
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
|  :-------------   |  :---: |
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

###文件权限、所有权和粘滞位
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
