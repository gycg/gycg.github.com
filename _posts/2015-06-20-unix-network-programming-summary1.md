---
layout: post
title: Unix Network Programming总结（一）
categories: [blog ]
tags: [Network, 建站前 ]
description: Network学习第一弹
---

##套接字编程简介
###套接字地址结构
####IPv4套接字地址结构(16字节)
{% highlight c %}
struct in_addr {                /* 32-bit IPv4 address */
    in_addr_t   s_addr;         /* network byte ordered */
};
{% endhighlight %}

{% highlight c %}
struct sockaddr_in {
    uint8_t         sin_len;        /* length if structure (16) */
    sa_family_t     sin_family;     /* AF_INET */
    in_port_t       sin_port;       /* 16-bit TCP or UDP port number */
                                    /* network byte ordered */
    struct in_addr  sin_addr;       /* 32-bit IPv4 address */
                                    /* network byte ordered */
    char            sin_zero[8];    /* unused */
};
{% endhighlight %}
**POSIX规范要求的数据结构**

| 数据类型   |    说明                 |     头文件         |
| :--------  | :---------------------- | :----------------: |
| int8_t     | 带符号的8位整数         |  \<sys/types.h\>   |
| uint8_t    | 无符号的8位整数         |  \<sys/types.h\>   |
| int16_t    | 带符号的16位整数        |  \<sys/types.h\>   |
| uint16_t   | 无符号的16位整数        |  \<sys/types.h\>   |
| int32_t    | 带符号的32位整数        |  \<sys/types.h\>   |
| uint32_t   | 无符号的32位整数        |  \<sys/types.h\>   |
| sa\_family\_t | 套接字地址结构的地址族 |  \<sys/socket.h\>  |
| socklen_t   | 套接字地址结构的长度，一般为uint32_t | \<sys/socket.h\> |
| in_addr_t  | IPv4地址，一般为uint32_t | \<netinet/in.h\>  |
| in_port_t  | TCP或UDP端口，一般为uint16_t | \<netinet/in.h\>  |

####通用套接字地址结构
{% highlight c %}
struct sockaddr {
    uint8_t     sa_len;
    sa_family_t sa_family;      /* address family: AF_xxx value */
    char        sa_data[14];    /* proti=ocol-specific address */
}
{% endhighlight %}
调用时对指向特定协议的套接字地址结构指针进行类型强制转换（casting）：
{% highlight c %}
int bind(int, struct sockaddr *, socklen_t);
struct sockaddr_in serv;        /* IPv4 socket address structure */
/* fill in serv{} */
bind(sockfd, (struct sockaddr *) &serv, sizeof(serv));
{% endhighlight %}

####IPv6套接字地址结构(28字节)
{% highlight c %}
struct in6_addr {
    uint8_t s6_addr[16];        /* 128-bit IPv6 address */
                                /* network byte ordered */
};
#define SIN6_LEN                /* required for compile-time tests */
struct sockaddr_in6 {
    uint8_t         sin6_len;       /* length of this struct (28) */
    sa_family_t     sin6_family;    /* AF_INET6 */
    in_port_t       sin6_port;      /* transport layer port# */
                                    /* network byte ordered */
    uint32_t        sin6_flowinfo;  /* flow infomation, undefined */
    struct in6_addr sin6_addr;      /* IPv6 address */
                                    /* network byte ordered */
    uint32_t        sin6_scope_id;  /* set of interfaces for a scope */
};
{% endhighlight %}
####新的通用套接字地址结构(系统中最大长度)
{% highlight c %}
struct sockaddr_storage {
    uint8_t         ss_len;         /* length of this struct (implementation dependent) */
    sa_family_t     ss_family;      /* address family: AF_xxx value */
    /* implementation-dependent elements to provides:

     * a) alignment sufficient to fulfill the alignment requirements of

     *    all socket address types that the system supports.

     * b) enough storage to hold any type of socket address that the

     *    system supports.

     */
};
{% endhighlight %}
###值-结果参数
从进程到内核传递套接字地址结构的函数：bind、connect和sendto。其中一个参数是指向某个套接字地址结构的指针，另一个参数是该结构的整数大小，例如：
{% highlight c %}
struct sockaddr_in serv;
/* fill in serv{} */
connect(sockfd, (SA *) &serv, sizeof(serv));
{% endhighlight %}
从内核到进程传递套接字地址结构的函数：accept、recvfrom、getsockname和getpeername。其中两个参数是指向某个套接字地址结构的指针和指向表示该结构大小的整型变量的指针，例如：
{% highlight c %}
struct sockaddr_un  cli;        /* Unix domain */
socklen_t len;
len = sizeof(cli);              /* len is a value */
getpeername(unixfd, (SA *) &cli, &len);
/* len may have changed */
{% endhighlight %}
把套接字地址结构大小这个参数从一个整数改为指向某个整数变量的指针，其原因在于：当函数被调用时，结构大小是一个值（value），它告诉内核该结构的大小，这样内核在写该结构的时候不至于越界；当函数返回时，结构大小又是一个结果（result），它告诉进程内核在该结构中究竟存储了多少信息。这种类型的参数称为值-结果（value-result）参数。
####字节排序函数
