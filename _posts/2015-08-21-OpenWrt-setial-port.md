---
layout: post
title: OpenWrt Serial Port Configuration
categories: [blog ]
tags: [internship, ]
description: 
---

####以一个例子来说明OpenWrt串口的使用。本例打开串口1，设置9600波特率、8位数据位、1位停止位以及空校验，之后利用while语句循环判断串口中是否可以读出数据，将串口中数据连续读出后重新写回到串口中。  

####可以用minicom打开串口1对实验结果进行观察，运行程序，设置minicom相关参数，在minicom中输入数据，程序每隔2秒统计一次写入的字符个数，并打印出输入的字符以及个数。  

{% highlight c %}
#include     <stdio.h>
#include     <stdlib.h> 
#include     <unistd.h>  
#include     <sys/types.h>
#include     <sys/stat.h>
#include     <fcntl.h> 
#include     <termios.h>
#include     <errno.h>
   
main()
{
    int fd;
    int i;
    int len;
    int n = 0;      
    char read_buf[256];
    char write_buf[256];
    struct termios opt; 
    
    fd = open("/dev/ttyS0", O_RDWR | O_NOCTTY);    
    if(fd == -1)
    {
        perror("open serial 0\n");
        exit(0);
    }

    tcgetattr(fd, &opt);      
    cfsetispeed(&opt, B9600);
    cfsetospeed(&opt, B9600);
    
    if(tcsetattr(fd, TCSANOW, &opt) != 0 )
    {     
       perror("tcsetattr error");
       return -1;
    }
    
    opt.c_cflag &= ~CSIZE;  
    opt.c_cflag |= CS8;   
    opt.c_cflag &= ~CSTOPB; 
    opt.c_cflag &= ~PARENB; 
    opt.c_cflag &= ~INPCK;
    opt.c_cflag |= (CLOCAL | CREAD);
 
    opt.c_lflag &= ~(ICANON | ECHO | ECHOE | ISIG);
 
    opt.c_oflag &= ~OPOST;
    opt.c_oflag &= ~(ONLCR | OCRNL);    
 
    opt.c_iflag &= ~(ICRNL | INLCR);
    opt.c_iflag &= ~(IXON | IXOFF | IXANY);    
    
    opt.c_cc[VTIME] = 0;
    opt.c_cc[VMIN] = 0;
    
    tcflush(fd, TCIOFLUSH);
 
    printf("configure complete\n");
    
    if(tcsetattr(fd, TCSANOW, &opt) != 0)
    {
        perror("serial error");
        return -1;
    }
    printf("start send and receive data\n");
  
    while(1)
    {    
        n = 0;
        len = 0;
        bzero(read_buf, sizeof(read_buf));    
        bzero(write_buf, sizeof(write_buf));
 
        while((n=read(fd,read_buf,sizeof(read_buf)))>0)
        {
            for(i = len; i < (len + n); i++)
            {
                write_buf[i] = read_buf[i - len];
            }
            len += n;
        }
        write_buf[len] = '\0';
              
        printf("Len %d \n", len);
        printf("%s \n", write_buf);
 
        n = write(fd, write_buf, len);
        printf("write %d chars\n",n);
        
        sleep(2);
    }
}
{% endhighlight %}
