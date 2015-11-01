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
###字节排序函数
**确定主机字节序的程序**
{% highlight c %}
/* filepath: intro/byteorder.c */
#include    "unp.h"
int
main(int argc, char **argv)
{
    union {
        short   s;
        char    c[sizeof(short)];
} un;
    un.s = 0x0102;
    printf("%s: ", CPU_VENDOR_OS);
    if (sizeof(short) == 2) {
        if(un.c[0] == 1 && un.c[1] == 2)
            printf("big-endian\n");
        else if(un.c[0] == 2 && un.c[1] == 1)
            printf("little-endian\n");
        else
            printf("unknow\n");
    } else
        printf("sizeof(short) = %d\n", sizeof(short));
    exit(0);
}
{% endhighlight %}
**网络字节序和主机字节序的转换**
{% highlight c %}
#include <netinet/in/h>
uint16_t htons(uint16_t host16bitvalue);
uint32_t htonl(uint32_t host32bitvalue);        均返回：网络字节序的值
uint16_t ntohs(uint16_t net16bitvalue);
uint32_t ntohl(uint32_t net32bitvalue);         均返回：主机字节序的值
{% endhighlight %}
###字节操纵函数
第一组名字以b开头，起源于4.2BSD
{% highlight c %}
#include <string.h>
void bzero(void *dest, size_t nbytes);
void bcopy(const void *src, void *dest, size_t nbytes);
int bcmp(const void *ptr1, const void *ptr2, size_t nbytes);
                                                返回：若相等则为0,否则为非0
{% endhighlight %}
第二组名字以mem开头，起源于ANSI C标准
{% highlight c %}
#include <string.h>
void *memset(void *dest, int c, size_t len);
void *memcpy(void *dest, const void *src, size_t nbytes);
int memcmp(const void *ptr1, const void *ptr2, size_t nbytes);
{% endhighlight %}
###inet\_aton,inet\_addr和inet\_ntoa函数
用于在点分十进制数串与它长度为32位的网络字节序二进制值间转换IPv4地址。
{% highlight c %}
#include <arpa/inet.h>
int inet_aton(const char *strptr, struct in_addr *addrptr);
                                    返回：若字符串有效则为1,否则为0
in_addr_t inet_addr(const char *strptr);
                                    返回：若字符串有效则为32位二进制网络字节序的IPv4地址，否则为INADDR_NONE
char *inet_ntoa(struct in_addr inaddr);
                                    返回：指向一个点分十进制数串的指针
{% endhighlight %}
inet\_aton将strptr所指C字符串转换成一个32位网络字节序二进制值，并通过指针addrptr来存储。  
inet\_addr与inet\_aton进行相同的转换，但是存在一些问题，已经被废弃。  
inet\_ntoa函数将一个32位的网络字节序二进制IPv4地址转换成相应的点分十进制数串。由该函数的返回值所指向的字符串驻留在静态内存中，所以该函数是不可重入的。
###inet\_pton和inet\_ntop函数
函数名中p和n分别代表*表达*（presentation）和*数值*（numeric）。  
{% highlight c %}
#include <arpa/inet.h>
int inet_pton(int family, const char *strptr, void *addrptr);
                                    返回：若成功则为1,若输入不是有效的表达格式则为0,若出错则为-1
const char *inet_ntop(int family, const void *addrptr, char *strptr, size_t len);
                                    返回：若成功则为指向结果的指针，若出错则为NULL
{% endhighlight %}
len参数是目标存储单元的大小，以免该函数溢出其调用者的缓冲区。为有助于指定这个大小，在<netinet/in.h>有文件中有如下定义：
{% highlight c %}
#define INET_ADDRSTRLEN     16  /* for IPv4 dotted-decimal */
#define INET6_ADDRSTRLEN    46  /* for IPv6 hex string */
{% endhighlight %}
只支持IPv4的inet_pton函数的简化版本：
{% highlight c %}
/* filepath: libfree/inet_pton_ipv4.c */
int
inet_pton(int family, const char *strptr, void *addrptr)
{
    if (family == AF_INET) {
        struct in_addr in_val;
        if (inet_aton(strptr, &in_val)) {
            memcpy(addrptr, &in_val, sizeof(struct in_addr));
            return (1);
        }
        return (0);
    }
    errno = EAFNOSUPPORT;
    return (-1);
}
{% endhighlight %}
只支持IPv4的inet_ntop函数的简化版本：
{% highlight c %}
/* filepath: libfree/inet_ntop_ipv4.c */
const char *
inet_ntop(int family, const void *addrptr, char *strptr, size_t len)
{
    const u_char *p = (const u_char *) addrptr;
    if (family == AF_INET) {
        char temp[INET_ADDRSTRLEN];
        snprintf(temp, sizeof(temp), "%d.%d.%d.%d", p[0], p[1], p[2], p[3]);
        if (strlen(temp) >= len) {
            errno = ENOSPC;
            return (NULL);
        }
        strcpy(strptr, temp);
        return (strptr);
    }
    errno = EAFNOSUPPORT;
    return (NULL);
}
{% endhighlight %}
###sock_ntop和相关函数
在使用sock\_ntop时，调用者需要传递一个指向二进制地址的指针，所以知道套接字地址结构的格式，使得代码与协议相关了。  
所以我们编写了一个名为sock\_ntop，它以指向某个套接字地址结构的指针为参数，查看该结构的内部，然后调用适当的函数返回该地址的表达格式。  
这里的表达格式是在IPv4地址的点分十进制数串之后，或者在一个括以方括号的IPv6地址的十六进制数串格式之后，跟一个终止符（我们使用一个冒号），再跟一个十进制的端口号（5个字节），最后跟一个空字符。因此，缓冲区大小对IPv4来说至少16+6=22字节，对IPv6至少为46+8=54字节。
{% highlight c %}
#include "unp.h"
char *sock_ntop(const struct sockaddr *sockaddr, socklen_t addrlen);
                                    返回：若成功则为非空指针，若出错则为NULL
{% endhighlight %}
函数sock_ntop仅在AF\_INET情形下的源代码：
{% highlight c %}
/* filepath: lib/sock_ntop.c */
char *
sock_ntop(const struct sockaddr *sa, socklen_t salen)
{
    char    portstr[8];
    static char str[128];       /* Unix domain is largest */
    switch (sa->sa_family) {
    case AF_INET: {
        struct sockaddr_in *sin = (struct sockaddr_in *) sa;
        if (inet_ntop(AF_INET, &sin->sin_addr, str, sizeof(str)) == NULL)
            return(NULL);
        if (ntohs(sin->sin_port) != 0) {
            snprintf(portstr, sizeof(portstr), ":%d", ntohs(sin->sin_port));
            strcat(str, portstr);
        }
        return(str);
    }
    }
}
{% endhighlight %}
为操作套接字地址结构定义的其他几个函数：
{% highlight c %}
#include "unp.h"
int sock_bind_wild(int sockfd, int family);
                                    返回：若成功则为0,若出错则为-1
int sock_cmp_addr(const struct sockaddr *sockaddr1,
                  const struct sockaddr *sockaddr2, socklen_t addrlen);
                                    返回：若地址为同一协议族且相同则为0,否则为非0
int sock_cmp_port(const struct sockaddr *sockaddr1,
                  const struct sockaddr *sockaddr2, socklen_t addrlen);
                                    返回：若地址为同一协议族且端口相同则为0,否则为非0
int sock_get_port(const struct sockaddr *sockaddr, socklen_t addrlen);
                                    返回：若为IPv4或IPv6地址则为非负端口号，否则为-1
int *sock_ntop_host(const struct sockaddr *sockaddr, socklen_t addrlen);
                                    返回：若成功则为非空指针，若出错则为NULL
void sock_set_addr(const struct sockaddr *sockaddr, socklen_t addrlen, void *ptr);
void sock_set_port(const struct sockaddr *sockaddr, socklen_t addrlen, int port);
void sock_set_wild(struct sockaddr *sockaddr, socklen_t addrlen);
{% endhighlight %}
###readn,writen和readline函数
字节流套接字上调用read和write输入或输出的字节数可能比请求的数量少。为了不让实现返回一个不足的字节计数值，我们改为调用readn和writen来取代它们。  
{% highlight c %}
#include "unp.h"
ssize_t readn(int filedes, void *buff, size_t nbytes);
ssize_t written(int filedes, const void *buff, size_t nbytes);
ssize_t readline(int filedes, void *buff, size_t nbytes);
{% endhighlight %}
size\_t一般为unsigned int，是c语言标准里的，用来表示对象的size;ssize\_t一般为有符号整型（signed size\_t）,来自于POSIX，用来表示字节数或错误标记.  
  

**readn函数：从一个描述符读n字节**
{% highlight c %}
/* filepath: lib/readn.c */
#include "unp.h"
ssize_t                       /* Read "n" bytes from a descriptor. */
readn(int fd, void *vptr, size_t n)
{
    size_t  nleft;
    ssize_t nread;
    char *ptr;
    ptr = vptr;
    nleft = n;
    while (nleft > 0) {
        if ( (nread = read(fd, ptr, nleft)) < 0) {
            if(errno == EINTR)
                nread = 0;      /* and call read() again */
            else
                return(-1);
        } else if (nread == 0)
            break;              /* EOF */
        nleft -= nread;
        ptr   += nread;
    }
    return(n - nleft);          /* return >= 0 */
}
{% endhighlight %}
**writen函数：往一个描述符写n字节**
{% highlight c %}
/* filepath: lib/written.c */
#include "unp.h"
ssize_t                         /* Write "n" bytes to a descriptor. */
writen(int fd, const void *vptr, size_t n)
{
    size_t  nleft;
    ssize_t nwritten;
    const char *ptr;
    ptr = vptr;
    nleft = n;
    while (nleft > 0) {
        if ( (nwritten = write(fd, ptr, nleft)) <= 0) {
            if (nwritten < 0 && errno == EINTR)
                nwritten = 0;   /* and call write() again */
            else
                return(-1);     /* error */
        }
        nleft -= nwritten;
        ptr   += nwritten;  
    }
    return(n);
}
{% endhighlight %}
**readline函数：从一个描述符读文本行，一次1个字节**
{% highlight c %}
/* filepath: test/readline1.c */
#include "unp.h"
/* PAINFULLY SLOW VERSION -- ezample only */
ssize_t
readline(int fd, void *vptr, size_t maxlen)
{
    ssize_t n, rc;
    char    c, *ptr;
    ptr = vptr;
    for (n = 1; n < maxlen; n++) {
    again:
        if ( (rc = read(fd,&c, 1)) == 1) {
            *ptr++ = c; 
            if (c == '\n')
                break;          /* newline is stored, like fgets() */
        } else if (rc == 0) {
            *ptr = 0;
            return(n - 1);      /* EOF, n - 1 bytes were read */
        } else {
            if (errno == EINTR)
                goto again;
            return(-1);         /* error, error set by read() */
        }
    }
    *ptr = 0;                   /* null terminate like fgets() */
    return(n);
}
{% endhighlight %}
**readline函数的改进版**
{% highlight c %}
/* filepath: lib/realine.c */
#include "unp.h"
static int  read_cnt;
static char *read_ptr;
static char read_buf[MAXLINE];
static ssize_t;
my_read(int fd, char *ptr)
{
    if (read_cnt <= 0) {
    again:
        if ( (read_cnt = read(fd, read_buf, sizeof(read_buf))) < 0) {
            if (errno == EINTR)
                goto again;
            return(-1);
        } else if (read_cnt == 0)
            return(0);
        read_ptr = read_buf;
    }
    read_cnt--;
    *ptr = *read_ptr++;
    return(1);
}
ssize_t
readline(int fd, void *vptr, size_t maxlen)
{
    ssize_t n, rc;
    char    c, *ptr;
    ptr = vptr;
    for (n = 1; n < maxlen; n++) {
        if ( (rc = myread(fd, &c)) == 1) {
            *ptr++ = c;
            if (c == '\n')
                break;          /* newline is stored, like fgets() */
        } else if (rc == 0) {
            *ptr = 0;
            return(n - 1);      /* EOF, n - 1 bytes were read */
        } else 
            return(-1);         /* error, errno set by read() */
    }
    *ptr = 0;                   /* null terminate like fgets() */
    return(n);
}
ssize_t
readlinebuf(void **vptrptr)
{
    if (read_cnt)
        *vptrptr = read_ptr;
    return(read_cnt);
}
{% endhighlight %}
