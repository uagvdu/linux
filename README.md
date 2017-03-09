这是一个客户端与服务器端进行连接通信的TCP连接程序：只能支持一台客户机连接一个服务器的程序

代码描述：

TCP/IP协议规定,网络数据流应采用大端字节序,即低地址高字节

套接字：IP地址 + TCP端口号
(socket)          一个套接字，唯一标识网络通讯中的一个进程
                      TCP/IP协议中，两个进程若想建立连接，就离不开套接字，它是TCP连接中的端点

套接字地址结构：（用来保存套接字的相应信息）
  linux下

IPv4套接字地址结构通常也叫做“网际套接字地址结构”，以sockaddr_in命名：定义在<netinet/in.h>
   struct in_addr
     {
          in_addr_t s_addr;  /*32位IP地址*/
     };

     struct sockaddr_in
     {
          short sin_family;                 /*Address family一般来说AF_INET（地址族）PF_INET（协议族）*/
          unsigned short sin_port;          /*Port number(必须要采用网络数据格式,普通数字可以用htons()函数转换成网络数据格式的数字)*/
          struct in_addr sin_addr;          /*IP address in network byte order（Internet address）*/
          unsigned char sin_zero[8];        /*Same size as struct sockaddr没有实际意义,只是为了 跟SOCKADDR结构在内存中对齐，是为了让sockaddr与sockaddr_in两个数据结构保持大小相同而保留的空字节。*/
     };
   


通用套接字地址结构：不建立该类型的结构体变量，而只是在传参的时候，进行强制类型转换成该类型，和void*类型一样，为了函数支持各种类型的数据
struct sockaddr               
{
     unsigned  short  sa_family;        /* address family, AF_xxx */
     char  sa_data[14];                 /* 14 bytes of protocol address */
};
     AF_前缀代表地址族，PF_前缀代表协议族
     sa_family是地址家族，一般都是“AF_xxx”的形式。好像通常大多用的是都是AF_INET。
     sa_data是14字节协议地址。
     
     此数据结构用做bind、connect、recvfrom、sendto等函数的参数，指明地址信息。
     但一般编程中并不直接针对此数据结构操作，而是使用另一个与sockaddr等价的数据结构,如sockaddr_in,sockaddr_un,sockaddr_in6...

套接字地址结构体使用方式:

     IPv4地址用sockaddr_in结构体表示，IPv6地址用sockaddr_in6结构体表示，UNIX Domain Socket地址用sockaddr_un结构体表示，为了让socket API可以接受各种类型的sockaddr结构体指针做参数，例如bind，accept，connect等函数，这些函数的参数应该设置为void* 类型的指针，但socket API的实现早于ANSI C标准化，那个时候还没有void* 类型，因此这些函数的参数都用struct sockaddr* 类型表示，所以struct sockaddr*是一个通用类型指针。在传递参数之前要强制类型转换：如
     struct sockaddr_in servaddr;
     bind(sock_fd,(struct sockaddr *)&servaddr,sizeof(servaddr));

所用函数：
 
1.socket();
      函数原型：  int socket(int domain,int type,int protocol);
      作用 ：socket函数是一种可用于根据指定的地址族、数据类型和协议来分配一个套接口的描述字及其所用的资源的函数
      参数：
           domain：　 一个地址描述。目前仅支持AF_xxxx格式：AF_INET代表IPv4地址
           type：　      新套接字的类型描述：SOCK_STREAM  ： 提供面向连接的稳定数据传输，即TCP协议。 SOCK_dGRAM：表示面向数据报的传输协议，即UDP协议
           protocol :    套接字所用的协议。如调用者不想指定，可用0指定，表示缺省。
       返回值：
          成功返回文件描述符>0，失败返回<0,


2.bind()
     函数原型：int bind(int sockfd, const struct sockaddr *addr,  socklen_t addrlen);
     作用： 绑定服务器ip地址和端口，因为服务器程序所监听的⽹网络地址和端⼜⼝口号通常是固定不变的，将参数sockfd和addr绑定在一
           起，使sockfd这个用于网络通讯的文件描述符监听addr所描述的地址和端口号
     参数：
               sockfd：socket创建成功的返回值
               addr :存有套接字信息的结构体的地址（注意强制类型转换）
               addrlen： 前一个套接字结构体类型的变量的长度:addr参数可以接受多种协议的sockaddr结构体，而它们的长度各不
                         相同，所以需要第三个addrlen指定结构体的长度
     返回值：
                    成功返回0，失败返回-1


3.  listen()：
         函数原型：int listen(int sockfd,int backlog);
         作用：仅被TCP服务器调用，声明sockfd处于监听状态，并且最多有backlog个客户端处于连接等待状态，
         返回值：成功返回0，失败返回-1；


4.accept():
           函数原型：int accept(int sockfd,struct sockaddr *addr,socklen_t *addrlen)
           作用：用于接收客户端的连接请求
           参数：
                    sockfd:称为监听套接字，即服务器的套接字，也是其文件描述符
                    addr : 客户端的套接字地址结构体的变量的地址
                    addrlen:传入传出型参数，传入的是调用者提供的缓冲区的长度，以避免缓冲区溢出，传出的是客户端地址结构体    
                            的实际长度（有可能没有占满调用者提供的缓冲区）
           返回值：
                 成功返回文件描述符（也是套接字，但和sockfd是不同的，前者叫监听套接口服务器关闭才结束，后者叫已连接套接
                 口，客户端推出结束），失败返回-1
           



  5.网络字节序和主机字节序的转换：（为使网络程序具有可移植性，使同样的代码在大端小端机器都可编译正常运行)

       #include <arpa/inet.h>

       uint32_t htonl(uint32_t hostlong);
       uint16_t htons(uint16_t hostshort);
       uint32_t ntohl(uint32_t netlong);
       uint16_t ntohs(uint16_t netshort);

     h表示host，n表示network，l表示32位长整数（IP地址），s表示16为短整数（端口号）；
     例如：htonl表示将32位的长整数从主机字节序转换为网络字节序，即：将IP地址转换后准备发送，若主机是小端字节序，这些函数将参数做相应的大小端转换后返回，若是大端字节序，这些函数不做转换，将参数原封不动的返回




6.字符串转in_addr的函数:
       
        int inet_aton(const char *strptr, struct in_addr *addrptr);
        in_addr_t inet_addr(const char *strptr);
        int inet_pton(int family,const char* strptr,void *addrptr);





7.in_addr转字符串的函数：
   
       char *inet_ntoa(struct in_addr inaddr);
       const char *inet_ntop(int family,const void* addrptr,char* strptr,size_t len);

      其中inet_pton和inet_ntop不仅可以转换IPv4的in_addr,还可以转换IPv6的in6_addr,因此函数接口是void *addrptr


8. connect()：
     函数原型：int connect (int sockfd,const struct sockaddr *addr,socklen_t addrlen)
     作用：客户端调用connect连接服务器
     参数：其参数形式和bind一致，区别在于bind的参数是自己的地址，而connect参数是对方的地址。
     返回值：
          成功返回0，失败返回-1.
