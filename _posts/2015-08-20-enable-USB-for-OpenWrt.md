---
layout: post
title: Enable USB for OpenWrt
categories: [blog ]
tags: [internship, ]
description: 
---

在配置openwrt编译系统过程中，除了基本的配置外，如果还要支持USB挂载、文件系统、语言等内容，那么就需要在#make menuconfig界面下配置以下各项:  
####添加USB相关支持
Kernel modules —> USB Support —> <*>kmod-usb-core.  
Kernel modules —> USB Support —> <*> kmod-usb-ohci. #old usb1.0  
Kernel modules —> USB Support —> <*> kmod-usb-uhci. #usb1.1  
Kernel modules —> USB Support —> <*> kmod-usb-storage.  
Kernel modules —> USB Support —> <*> kmod-usb-storage-extras.  
Kernel modules —> USB Support —> <*> kmod-usb2.     #usb2.0  
####添加USB挂载
Base system —> <*>block-mount  
Utilities —> Filesystem —> <*> badblocks  
Utilities ---> <*> usbutils................................... USB devices listing utilities  
Utilities ---> disc ---> <*> fdisk.................................... manipulate disk partition table  
####添加文件系统支持
Kernel modules —> Filesystems —> <*> kmod-fs-ext4 (移动硬盘EXT4格式选择)  
Kernel modules —> Filesystems —> <*> kmod-fs-vfat(FAT16 / FAT32 格式 选择)  
Kernel modules —> Filesystems —> <*> kmod-fs-ntfs (NTFS 格式 选择)  
####添加UTF8编码,CP437编码，ISO8859-1编码
Kernel modules —> Native Language Support —> <*> kmod-nls-cp437  
Kernel modules —> Native Language Support —> <*> kmod-nls-iso8859-1  
Kernel modules —> Native Language Support —> <*> kmod-nls-utf8  

以上几步也可以在系统中用opgk install安装相应的驱动来实现。 
####自动挂载脚本（可根据需要改动,参考\\http://blog.csdn.net/gubenpeiyuan/article/details/8330660）
以上几步完成后，编译并烧写系统到芯片上，在系统中编写自动挂载脚本。编辑/etc/hotplug.d/block/10-mount如下：  

{% highlight bash %}

#!/bin/sh
 
# Copyright (C) 2009 OpenWrt.org  (C) 2010 OpenWrt.org.cn
 
blkdev=`dirname $DEVPATH`
if [ `basename $blkdev` != "block" ]; then
    device=`basename $DEVPATH`
    case "$ACTION" in
        add)
                mkdir -p /mnt/$device
                # vfat & ntfs-3g check
                if  [ `which fdisk` ]; then
                        isntfs=`fdisk -l | grep $device | grep NTFS`
                        isvfat=`fdisk -l | grep $device | grep FAT`
                        isfuse=`lsmod | grep fuse`
                        isntfs3g=`which ntfs-3g`
                else
                        isntfs=""
                        isvfat=""
                fi 
                # mount with ntfs-3g if possible, else with default mount
                if [ "$isntfs" -a "$isfuse" -a "$isntfs3g" ]; then
                        ntfs-3g -o nls=utf8 /dev/$device /mnt/$device
                elif [ "$isvfat" ]; then
                        mount -t vfat -o iocharset=utf8,rw,sync,umask=0000, \
      dmask=0000,fmask=0000  /dev/$device /mnt/$device
                else
                        mount /dev/$device /mnt/$device
                fi
  if [ -f /dev/${device}/swapfile ]; then
   mkswap /dev/${device}/swapfile
   swapon /dev/${device}/swapfile
  fi
                ;;
        remove)
  if [ -f /dev/${device}/swapfile ]; then
   swapoff /dev/${device}/swapfile
  fi
                umount /dev/$device
                ;;
    esac
 
fi
{% endhighlight %}
