---
title: Linux下的Socket编程(一)
date: 2021-03-07 20:58:16
cover: /images/socket.jpg
tags: socket
category: linux
---
Linux环境下的Socket开发，主要讲解socket编程的简单认识以及所需要依赖的库。最后形成一个简单的测试用例

## Socket编程认识
在Linux环境下，任何内容（驱动，硬件，网络）都被认为是文件。在此种语境下，我们可以像操作文件一样操作Linux中的任何内容。
+ Linux下文件操作要素

  Linux下文件操作首先需要获取到一个关于文件的id(int)。通过此id你就可以对文件进行读与写操作。程序的主要任务就是对数据的读与写。（程序的本质是数据与算法）

+ Socket操作要素
  + 意思讲解：`Socket`的意思是“插座”，就是你的程序作为“插头”，插到插座上，只要你们的协议相同那么就可以进行通信。
  + Socket作为一个特殊的文件，同样会返回一个id(int)用于对其进行操作。

## Socket编程涉及函数

关于Socket编程的几个函数如socket、bind、connect、listen、accept都位于<sys/socket.h>中

### socket函数
socket函数用于创建套接字，其原型为：
```
    int socket(int af, int type, int protocol);
```

+ af(Address Family):位于<sys/socket.h>库中，称为地址族，也就是IP地址类型，常用的有`AF_INET`和`AF_INET6`。其中`AF_INET`是ipv4，`AF_INET6`是ipv6。

  > 注意：你也可以使用PF(Protocol Family)作为前缀，即PF_INET和PF_INET6，其位于<netinet/in.h>

+ type:位于<sys/socket.h>库中，是数据传输方式/套接字类型，其中常用的套接字有`SOCK_STREAM`(流格式套接字/面向连接套接字)和`SOCK_DGRAM`(数据报套接字/面向无连接套接字)，其中还有很多类型，将会后续中学习。决定了传输的形式。
+ protocol:表示传输协议，一般的传输协议有`IPPROTO_TCP`和`IPPROTO_UDP`,分别表示tcp和udp。此参数决定了采用哪种协议进行传输

### bind和connect函数

bind用于将套接字绑定到本地的ip和端口上，其原型为：

```
 int bind(int socket, const struct sockaddr *address,
             socklen_t address_len);
```

connect函数用于通过sockaddr传来的ip与端口连接到远端上，其原型为

```
int connect(int socket, const struct sockaddr *address,
             socklen_t address_len);
```



+ socket:刚建立的socket返回的id，

+ address:位于<sys/socket.h>库中，这个数据结构如下

  ```
  struct socketaddr {
     sa_family_t  sin_family;   //地址族（Address Family），也就是地址类型
     char         sa_data[14];  //IP地址和端口号
  }  
  ```

  

+ socketlen_t:表示socketaddr的整个的长度，可以使用`sizeof`获取其长度

### listen函数

listen函数用于启动监听，让其进入被动监听状态，其原型为

```
int listen(int socket, int backlog);
```

+ socket: 刚才建议的socket返回的id

+ backlog: 指定可以接受的最大的请求数。当设置为SOMAXCONN，就会由系统决定请求队列长度，一般比较大，可能是几百

+ 请求队列：在上诉backlog的字段就是控制请求队列的最大长度。在套接字处理客户端的请求时，如果有新的请求来，一般会将其放入到缓冲区中，等到请求处理完成后，再从缓冲区中读取请求来进行处理。这个缓冲区就被称为请求队列（Request Queue）

### accept函数

当套接字处理监听状态，可以通过accept函数接收客户端的请求。这个函数会返回一个新的socketId,后续的所有操作都是通过此id进行操作。其原型为

```
int  accept(int socket, struct sockaddr *address,
             socklen_t *address_len);
```

+ socket:刚才建立的socket返回的id
+ address:位于<sys/socket.h>库中
+ address_len: 通过`sizeof`获取到长度

### read与write函数

向文件中写入和读取数据，其原型如下

```
ssize_t write(int fd, const void *buf, size_t nbytes);
ssize_t read(int fd, void *buf, size_t nbytes);
```

+ fd:要写入或读取的文件描述符
+ buf:需要写入或读取数据的缓存区
+ nbytes:要写入或读取的数据的长度

> size_t 是通过 typedef 声明的 unsigned int 类型；ssize_t 在 "size_t" 前面加了一个"s"，代表 signed，即 ssize_t 是通过 typedef 声明的 signed int 类型。

## 简单测试用例

### 使用问题解释

+ 在使用socket进行编程的时候会用到socketaddr，然而由于转换问题无法使用，需要换成sockaddr_in进行传参，然后进行强转即可。以下是详细解释
  + sockaddr_in结构体：此结构位于<netinet/in.h>中，其中包括如下内容

    ```
    struct sockaddr_in{
        sa_family_t     sin_family;   //地址族（Address Family），也就是地址类型
        uint16_t        sin_port;     //16位的端口号
        struct in_addr  sin_addr;     //32位IP地址
        char            sin_zero[8];  //不使用，一般用0填充
    };
    ```

    

    > 注意：sockaddr与sockaddr_in的占用内存都是14，所以可以进行转换。而且在程序中采用sockaddr_in而非sockaddr，是由于没有相关的函数将"127.0.0.1:80"字符串形式转换成数字表示

  + inaddr结构体：此结构位于<netinet/in.h>中，其中包含内容如下

    ```
    struct in_addr{
        in_addr_t  s_addr;  //32位的IP地址
    };
    ```

    > 其中in_addr_t被定义与<arpa/inet.h>中，

+ inet_addr函数是将字符串的ip转换成数字的函数，其位于<arpa/inet.h>，主要的作用就是将字符串转换成网络传输需要。

+ htons函数：全称host to network short，即将本地字节顺序转换为网络字节顺序。[具体内容可阅读本文](https://blog.csdn.net/myyllove/article/details/83380209)

### server端

```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#include<netinet/in.h>

int main() {
    //创建套接字
    int serv_socket = socket(AF_INET,SOCK_STREAM,IPPROTO_TCP);

    struct sockaddr_in serv_addr;
    memset(&serv_addr,0,sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    serv_addr.sin_port = htons(1234);
    
    bind(serv_socket,(struct sockaddr*)&serv_addr,sizeof(serv_addr));

    //监听状态
    listen(serv_socket,20);

    //等待并接受客户端请求
    struct sockaddr_in clnt_addr;
    socklen_t clnt_addr_size = sizeof(clnt_addr);
    int clnt_sock = accept(serv_socket,(struct sock_addr*)&clnt_addr,&clnt_addr_size);

    char str[] = "http://www.hepng.cool";
    write(clnt_sock,str,sizeof(str));

    //关闭套接字
    close(clnt_sock);
    close(serv_socket);

    return 1;

}
```

+ accept函数阻塞：listen函数让socket进入监听状态，但是会让代码继续往下运行。当运行到accept的时候会进行阻塞，直到客户端的connect进行连接后才往后面继续运行。
+ 

### client端

```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#include<netinet/in.h>

int main() {

    int sock = socket(AF_INET,SOCK_STREAM,0);
    struct sockaddr_in serve_addr;
    memset(&serve_addr,0,sizeof(serve_addr));
    serve_addr.sin_family = AF_INET;
    serve_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    serve_addr.sin_port = htons(1234);
    connect(sock,(struct sockaddr*)&serve_addr,sizeof(serve_addr));

    char buffer[40];

    read(sock,buffer,sizeof(buffer)-1);

    printf("Message from server : %s\n", buffer);

    close(sock);

    return 1;
}

```

## 总结

+ <sys/socket.h>：包含所有的socket通信的方法与数据结构，包括socket、bind、connect、listen、accept，socket连接的AF，type, tcp等

+ <arpa/inet.h>：用于帮助数据转换，包括从本地字节顺序转为网络字节顺序，转换ip地址字符串为数字
+ <netinet/in.h>：主要定义了sockaddr_in的数据类型，帮助开发人员进行数据的传入

## 参考文献

[socket学习](http://c.biancheng.net/view/2123.html)

[socket的函数库](https://pubs.opengroup.org/onlinepubs/7908799/xns/syssocket.h.html)

[inet的函数库](https://pubs.opengroup.org/onlinepubs/7908799/xns/arpainet.h.html)

[netinetin的函数库](https://pubs.opengroup.org/onlinepubs/7908799/xns/netinetin.h.html)