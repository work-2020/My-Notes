# C++ Socket编程

## 一、套接字（socket）

网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个socket。

套接字（socket）允许在相同或不同的机器上的两个不同进程之间进行通信。更准确地说，它是使用标准Unix文件描述符与其他计算机通信的一种方式，文件描述符只是与打开文件关联的整数，它可以是网络连接、文本文件、终端或其他内容。

套接字有四种类型：

+ 流（stream）套接字 - 在网络环境中保证交付，使用TCP协议。
+ 数据报（Datagram）套接字 - 无法保证在网络环境中交付，使用UDP协议。
+ 原始（raw）套接字 - 使用原始套接字，用户可以访问底层通信协议，这些协议支持套接字抽象。
+ 顺序数据包（Sequenced Packet）套接字 - 类似于流套接字。

使用socket的时候需要使用各种结构来保存有关地址和端口的信息以及其他信息。  大多数套接字函数都需要一个指向套接字地址结构的指针作为参数。通常使用四元组<源ip，源port，目的ip，目的port>来描述一个网络连接，使用socket的时候，往往也需要数据结构来描述这些信息。

```c++
struct sockaddr {
   unsigned short   sa_family;
   char             sa_data[14];
};
```

`sockaddr`是一个通用的套接字地址结构，在大多数套接字函数调用中都需要使用它。`sa_family`包括`AF_INET`，`AF_UNIX`，`AF_NS`，`AF_IMPLINK`，每个值代表一种地址族（address family），在基于IP的情况中，都使用`AF_INET`。`sa_data`长为14字节，根据地址类型解释协议特定地址。针对Internet协议，采用`sockaddr_in`结构定义地址，

```c++
struct sockaddr_in {
   short int            sin_family;
   unsigned short int   sin_port;//端口号，16位长，网络地址序
   struct in_addr       sin_addr;//IP地址，32位长，网络地址序
   unsigned char        sin_zero[8];
};
```

其中，`sin_family`和`sockadd`的`sa_family`一样，包括四个可选值：`AF_INET`，`AF_UNIX`，`AF_NS`，`AF_IMPLINK`。

socket设计之初本来就是准备支持多个地址协议的。不同的地址协议由自己不同的地址构造，譬如对于IPv4就是`sockaddr_in`， IPV6就是`sockaddr_in6`， 以及对于AF_UNIX就是`sockaddr_un`，sockaddr是对这些地址的上一层的抽象。

在`sockaddr_in`中还有一个结构体，`struct in_addr`，就是一个32位的IP地址。

```c++
struct in_addr {
   unsigned long s_addr;
};
```

不同操作系统存在大端表示和小端表示的区别，为了允许具有不同字节顺序约定的机器相互通信，Internet协议为通过网络传输的数据指定了规范的字节顺序约定， 这称为网络字节顺序。

不用担心这几个数据结构以及字节序，因为socket接口非常贴心地准备好了各种友好的接口。

- `htons()` Host to Network Short
- `htonl() `Host to Network Long
- `ntohl() `Network to Host Long
- `ntohs()` Network to Host Short

## 二、服务端

### 1.创建套接字对象

```c++
/*
* _domain 套接字使用的协议族信息(2层协议)
* _type 套接字的传输类型(3层协议)
* __protocol 通信协议(4层协议)
* */
int socket(int _domain, int _type, int _protocol) _THROW
```

+ _domain，协议族的可选字段，下面列出常见的字段，

  | 含义    | 地址族                     |
  | ------- | -------------------------- |
  | `AF_INET` | IPv4网络协议中采用的地址族 |
  | `AF_INET6` | IPv6网络协议中采用的地址族 |
  | `AF_LOCAL` | 本地通信中采用的UNIX协议的地址族（用的少）|

+ _type，套接字类型

  |  套接字类型   |                             含义                             |
  | :-----------: | :----------------------------------------------------------: |
  | `SOCKET_RAW`  | 原始套接字(SOCKET_RAW)允许对较低层次的协议直接访问，比如IP、 ICMP协议。 |
  | `SOCK_STREAM` |        SOCK_STREAM是数据流，一般为TCP/IP协议的编程。         |
  | `SOCK_DGRAM`  |        SOCK_DGRAM是数据报，一般为UDP协议的网络编程；         |

+ _protocol，最终采用的协议。常见的协议有`IPPROTO_TCP`、`IPPTOTO_UDP`。如果第二个参数选择了`SOCK_STREAM`，那么采用的协议就只能是`IPPROTO_TCP`；如果第二个参数选择的是`SOCK_DGRAM`，则采用的协议就只能是`IPPTOTO_UDP`。

### 2. 向套接字分配网络地址--bind()

```c++
/* 
* __fd:socket描述字，也就是socket引用
* myaddr:要绑定给sockfd的协议地址
* __len:地址的长度
*/
int bind (int __fd, const struct sockaddr* myaddr, socklen_t __len)  __THROW;
```

+ `_fd`，socket文件描述符，即套接字创建时返回的对象；

+ `myaddr`则是填充了一些网络地址信息，包含通信所需要的相关信息，其结构如下：

  ```c++
  struct sockaddr {
     unsigned short   sa_family;
     char             sa_data[14];
  };
  ```

  `sockaddr`是一个通用的套接字地址结构，在大多数套接字函数调用中都需要使用它。`sa_family`包括`AF_INET`，`AF_UNIX`，`AF_NS`，`AF_IMPLINK`，每个值代表一种地址族（address family），在基于IP的情况中，都使用`AF_INET`。`sa_data`长为14字节，根据地址类型解释协议特定地址。

  针对Internet协议，采用该结构体的变体`sockaddr_in`结构定义地址，

  ```c++
  struct sockaddr_in {
     short int            sin_family;
     unsigned short int   sin_port;//端口号，16位长，网络地址序
     struct in_addr       sin_addr;//IP地址，32位长，网络地址序
     unsigned char        sin_zero[8];
  };
  ```

  其中，`sin_family`和`sockadd`的`sa_family`一样，包括四个可选值：`AF_INET`，`AF_UNIX`，`AF_NS`，`AF_IMPLINK`。

  socket设计之初本来就是准备支持多个地址协议的。不同的地址协议由自己不同的地址构造，譬如对于IPv4就是`sockaddr_in`， IPV6就是`sockaddr_in6`， 以及对于AF_UNIX就是`sockaddr_un`，sockaddr是对这些地址的上一层的抽象。

  在`sockaddr_in`中还有一个结构体，`struct in_addr`，就是一个32位的IP地址。

  ```c++
  struct in_addr {
     unsigned long s_addr;
  };
  ```

  不同操作系统存在大端表示和小端表示的区别，为了允许具有不同字节顺序约定的机器相互通信，Internet协议为通过网络传输的数据指定了规范的字节顺序约定， 这称为网络字节顺序。

  而`sin_zero` 无特殊的含义，只是为了与上面介绍的`sockaddr`结构体一致而插入的成员。

  在网络传输时，统一都是以大端序的网络字节序方式传输数据，而我们通常习惯的IP地址格式是点分十进制，例如：“219.228.148.169”，这个时候就会调用以下函数进行转化，将IP地址转化为32位的整数形数据，同时进行网络字节转换：

  ```c++
  in_addr_t inet_addr (const char *__cp) __THROW;
  //或者
  int inet_aton (const char *__cp, struct in_addr *__inp) __THROW;	//windows无此函数
  ```

  如果单纯要进行网络字节序地址的转换，可以采用如下函数：

  ```C++
  /*Functions to convert between host and network byte order.
  
     Please note that these functions normally take `unsigned long int' or
     `unsigned short int' values as arguments and also return them.  But
     this was a short-sighted decision since on different systems the types
     may have different representations but the values are always the same.  */
  
  // h代表主机字节序
  // n代表网络字节序
  // s代表short(4字节)
  // l代表long(8字节)
  extern uint32_t ntohl (uint32_t __netlong) __THROW __attribute__ ((__const__));
  extern uint16_t ntohs (uint16_t __netshort)__THROW __attribute__ ((__const__));
  extern uint32_t htonl (uint32_t __hostlong)__THROW __attribute__ ((__const__));
  extern uint16_t htons (uint16_t __hostshort)
  ```

### 3. 进入等待链接请求状态--listen()

给套接字分配了所需的信息后，就可以调用`listen()`函数对来自客户端的连接请求进行监听（客户端此时要调用`connect()`函数进行连接）

```c++
/* Prepare to accept connections on socket FD.
   N connection requests will be queued before further requests are refused.
   Returns 0 on success, -1 for errors.  */
extern int listen (int __fd, int __n) __THROW;
```

第一个参数：socket文件描述符`__fd`，分配所需的信息后的套接字。

第二个参数：连接请求的队列长度，如果为6，表示队列中最多同时有6个连接请求。

这个函数的fd(socket套接字对象)就相当于一个门卫，对连接请求做处理，决定是否把连接请求放入到server端维护的一个队列中去。

### 4. 受理客户端的链接请求--accept()

`listen()`中的sock(__fd : socket对象)发挥了服务器端接受请求的门卫作用，此时为了按序受理请求，给客户端做相应的回馈，连接到发起请求的客户端，此时就需要再次创建另一个套接字，该套接字可以用以下函数创建：

```c++
/* Await a connection on socket FD.
   When a connection arrives, open a new socket to communicate with it,
   set *ADDR (which is *ADDR_LEN bytes long) to the address of the connecting
   peer and *ADDR_LEN to the address's actual length, and return the
   new socket's descriptor, or -1 for errors.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
extern int accept (int __fd, struct sockaddr *addr, socklen_t *addr_len);
```

函数成功执行时返回socket文件描述符，失败时返回-1。

第一个参数：socket文件描述符`__fd`，要注意的是这个套接字文件描述符与前面几步的套接字文件描述符不同。

第二个参数：保存发起连接的客户端的地址信息。

第三个参数： 保存该结构体的长度。

### 5. 发送消息--write()/send()

linux下的发送函数为：

```C++
/* Write N bytes of BUF to FD.  Return the number written, or -1.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
 ssize_t write (int __fd, const void *__buf, size_t __n) ;
```

而在windows下的发送函数为：

```C++
ssize_t send (int sockfd, const void *buf, size_t nbytes, int flag) ;
```

第四个参数是传输数据时可指定的信息，一般设置为0。

### 6. 接收消息--read()/recv()

linux下的接收函数为

```C++
/* Read NBYTES into BUF from FD.  Return the
   number read, -1 for errors or 0 for EOF.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
ssize_t read (int __fd, void *__buf, size_t __nbytes);
```

而在windows下的接收函数为

```C++
ssize_t recv(int sockfd, void *buf, size_t nbytes, int flag) ;
```

### 7. 关闭连接--close()

```C++
/* Close the file descriptor FD.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
int close (int __fd);
```

退出连接，此时要注意的是：**调用`close()`函数即表示向对方发送了`EOF`结束标志信息**。

## 三、客户端

服务端的socket套接字在绑定自身的IP即 及端口号后这些信息后，就开始监听端口等待客户端的连接请求，此时客户端在创建套接字后就可以按照如下步骤与server端通信，创建套接字的过程不再重复了。

### 1. 请求连接

```C++
/* Open a connection on socket FD to peer at ADDR (which LEN bytes long).
   For connectionless socket types, just set the default address to send to
   and the only address from which to accept transmissions.
   Return 0 on success, -1 for errors.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
int connect (int socket, struct sockaddr* servaddr, socklen_t addrlen);
```

几个参数的意义和前面的accept函数意义一样。要注意的是服务器端收到连接请求的时候并不是马上调用accept()函数，而是把它放入到请求信息的等待队列中。

### 2. 套接字的多种可选项

可以通过如下函数对套接字可选项的参数进行获取以及设置。

```c++
/* Put the current value for socket FD's option OPTNAME at protocol level LEVEL
   into OPTVAL (which is *OPTLEN bytes long), and set *OPTLEN to the value's
   actual length.  Returns 0 on success, -1 for errors.  */
extern int getsockopt (int sock, int __level, int __optname,
		       void *__optval, socklen_t *optlen) __THROW;

/* Set socket FD's option OPTNAME at protocol level LEVEL
   to *OPTVAL (which is OPTLEN bytes long).
   Returns 0 on success, -1 for errors.  */
extern int setsockopt (int sock, int __level, int __optname,
		       const void *__optval, socklen_t __optlen) __THROW;
```

**scok**： 套接字的文件描述符

**__level ** ：可选项的协议层，如下：

| **协议层**  |          **功能**          |
| :---------: | :------------------------: |
| SOL_SOCKET  | 套接字相关通用可选项的设置 |
| IPPROTO_IP  | 在IP层设置套接字的相关属性 |
| IPPROTO_TCP | 在TCP层设置套接字相关属性  |

**__optname** ：要查看的可选项名，几个主要的选项如下

|    **选项名**     |                           **说明**                           | **数据类型** | 所属协议层  |
| :---------------: | :----------------------------------------------------------: | :----------: | ----------- |
|     SO_RCVBUF     |                        接收缓冲区大小                        |     int      | SOL_SOCKET  |
|     SO_SNDBUF     |                        发送缓冲区大小                        |     int      | SOL_SOCKET  |
|    SO_RCVLOWAT    |                        接收缓冲区下限                        |     int      | SOL_SOCKET  |
|    SO_SNDLOWAT    |                        发送缓冲区下限                        |     int      | SOL_SOCKET  |
|      SO_TYPE      |            获得套接字类型(这个只能获取，不能设置)            |     int      | SOL_SOCKET  |
|   SO_REUSEADDR    | 是否启用地址再分配，主要原理是操作关闭套接字的Time-wait时间等待的开启和关闭 |     int      | SOL_SOCKET  |
|    IP_HDRINCL     |                     在数据包中包含IP首部                     |     int      | IPPROTO_IP  |
| IP_MULTICAST_TTL  |             生存时间(Time To Live)，组播传送距离             |     int      | IPPROTO_IP  |
| IP_ADD_MEMBERSHIP |                           加入组播                           |     int      | IPPROTO_IP  |
|    IP_OPTINOS     |                          IP首部选项                          |     int      | IPPROTO_IP  |
|    TCP_NODELAY    |                       不使用Nagle算法                        |     int      | IPPROTO_TCP |
|   TCP_KEEPALIVE   |        TCP保活机制开启下，设置保活包空闲发送时间间隔         |     int      | IPPROTO_TCP |
|   TCP_KEEPINTVL   |    TCP保活机制开启下，设置保活包无响应情况下重发时间间隔     |     int      | IPPROTO_TCP |
|    TCP_KEEPCNT    |    TCP保活机制开启下，设置保活包无响应情况下重复发送次数     |     int      | IPPROTO_TCP |
|    TCP_MAXSEG     |                     TCP最大数据段的大小                      |     int      | IPPROTO_TCP |

**__optval **   ：保存查看(get)/更改(set)的结果

**optlen **  ： 传递第四个参数的字节大小

示例：

+ 设置可选项的IO大小

  ```C++
  int status, snd_buf;
  socklen_t len = sizeof(snd_buf);
  status = getsockopt(serv_socket, SOL_SOCKET, SO_SNDBUF, (void*)&snd_buf, &len);
  cout << "发送缓冲区大小: " << snd_buf <<endl;
  ```

  虽然可以获得的接收/发送缓冲区的大小，但是通过设置接收/发送缓冲区大小时，得到的结果会与我们期望的不一样，因为对缓冲区大小的设置是一件很谨慎的事，其自身会根据设置的值进行一定的优化。


## 四、客户端\服务器代码

客户端代码：

```c++
#include<stdio.h>
#include<unistd.h>
#include<netinet/in.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<string.h>
#include <errno.h>
#include <stdlib.h>
int main(){
	
	int clientfd, conn;
	struct sockaddr_in servaddr,cliaddr;
	char buff[1024];
	char buff2[1024];
	int servlen;	
	int n;

	bzero(buff,1024);
	bzero(buff2,1024);	
	bzero(&cliaddr,sizeof(cliaddr));
	cliaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	cliaddr.sin_family = AF_INET;
	cliaddr.sin_port = htons(0);
	clientfd = socket(AF_INET,SOCK_STREAM,0);
	
	bzero(&servaddr,sizeof(servaddr));

	servaddr.sin_family = AF_INET;
	if(inet_pton(AF_INET,"127.0.0.1",&servaddr.sin_addr)<0) printf("address error1\n");
	//if(inet_pton(AF_INET,"192.168.116.158",&servaddr.sin_addr)<0) printf("address error1\n");
	servaddr.sin_port = htons(2345);

	servlen = sizeof(servaddr);
	conn = connect(clientfd,(struct sockaddr *)&servaddr,servlen);
	if(conn < 0) printf("connect error!\n");
	if(n=recv(clientfd,buff2,sizeof(buff2),0)>0)
		printf("Message %s:",buff2);

	printf("clientfd is %d,connfd is %d.\n",clientfd,conn);
	while(1){
	
	while((n=read(0,buff,sizeof(buff)))>0){
		printf("string length is %d\n",n);	
		if(send(clientfd,buff,n,0)<0) 
			{printf("send error! %s(errno :%d)\n",strerror(errno),errno);
				exit(0);}
		if((n=recv(clientfd,buff2,sizeof(buff2),0))>0){
			printf("echoed from server length is %d\n",n);
			write(0,buff2,n);
			printf("\n");
			
		}
	}
	
	}
	close(clientfd);

}
```

服务器端代码：

```c++
#include<stdio.h>
#include<unistd.h>
#include<netinet/in.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<string.h>
#include <errno.h>
#include <stdlib.h>
int main(){

	int listenfd, connfd;
	struct sockaddr_in servaddr,cliaddr;
	char buff[1024];
	int clilen;	
	int n;

	listenfd = socket(AF_INET,SOCK_STREAM,0);
	bzero(&servaddr,sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port = htons(2345);

	if(bind(listenfd,(struct sockaddr *)&servaddr,sizeof(servaddr))<0)
		printf("bind error!\n");	

	listen(listenfd,10);
	clilen = sizeof(cliaddr);
	printf("serverfd is %d, connfd is %d.\n",listenfd,connfd);
	while(1){
		connfd = accept(listenfd,(struct sockaddr *)&cliaddr,&clilen);
		send(connfd,"Welcome to Server!\n",19,0);

		while((n=read(connfd,buff,sizeof(buff)))>0){
			printf("Received string length is %d.\n",n);		
			//printf("%s\n",buff);	
			write(1,buff,n);
			write(connfd,buff,n);		
			//send(connfd,"hello",5,0);
		}
		

		close(connfd);}
	close(listenfd);

}
```

当服务端未收到消息时，`read`函数阻塞程序，直到收到客户端发来的消息；同理，当服务端未收到连接时，`accept`函数阻塞程序，直到收到客户端的连接。

## 五、Raw Socket编程

对于原始套接字，domain（family）套接字族可以是AF_INET、PF_INET、AF_PACKET和PF_PACKET；套接字类型是SOCK_RAW；至于协议可以查阅if_ether.h头文件。

简单来说，使用AF_INET，是面向IP层的原始套接字；使用AF_PACKET，是面向链路层的套接字。

原始套接字实现sniffer

```c++
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<netinet/ip_icmp.h>
#include<netinet/tcp.h>
#include<netinet/udp.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#include<sys/types.h>

#define BUFFSIZE 1024

int main(){

	int rawsock;
	char buff[BUFFSIZE];
	int n;
	int count = 0;

	rawsock = socket(AF_INET,SOCK_RAW,IPPROTO_TCP);
//	rawsock = socket(AF_INET,SOCK_RAW,IPPROTO_UDP);
//	rawsock = socket(AF_INET,SOCK_RAW,IPPROTO_ICMP);
//	rawsock = socket(AF_INET,SOCK_RAW,IPPROTO_RAW);
	if(rawsock < 0){
		printf("raw socket error!\n");
		exit(1);
	}
	while(1){	
		n = recvfrom(rawsock,buff,BUFFSIZE,0,NULL,NULL);
		if(n<0){
			printf("receive error!\n");
			exit(1);
		}

		count++;
		struct ip *ip = (struct ip*)buff;
		printf("%5d	%20s",count,inet_ntoa(ip->ip_src));
		printf("%20s	%5d	%5d\n",inet_ntoa(ip->ip_dst),ip->ip_p,ntohs(ip->ip_len));	
		printf("\n");
	}
```

为了接收所有分组，可以使用ETH_P_ALL，

`sock = socket(AF_PACKET, SOCK_RAW, htons(ETH_P_ALL))`

如果第一个参数是AF_PACKET，那么是面向链路层的套接字，第三个参数可以是

- `ETH_P_IP `   - 只接收目的mac是本机的IP类型数据帧

- `ETH_P_ARP`   - 只接收目的mac是本机的ARP类型数据帧

- `ETH_P_RARP ` - 只接收目的mac是本机的RARP类型数据帧

- `ETH_P_PAE `  - 只接收目的mac是本机的802.1x类型的数据帧

- `ETH_P_ALL`   - 接收目的mac是本机的所有类型数据帧，同时还可以接收本机发出的所有数据帧，混杂模式打开时，还可以接收到目的mac不是本机的数据帧

  完整代码如下：

  ```c++
  #include<stdio.h>
  #include<stdlib.h>
  #include<string.h>
  #include<netinet/ip_icmp.h>
  #include<netinet/tcp.h>
  #include<netinet/udp.h>
  #include<arpa/inet.h>
  #include<sys/socket.h>
  #include<sys/types.h>
  #include<netinet/ether.h>
  
  #define BUFFSIZE 1024
  
  int main()
  {
  
  	int rawsock;
  	unsigned char buff[BUFFSIZE];
  	int n;
  	int count = 0;
  
  	rawsock = socket(AF_PACKET,SOCK_RAW,htons(ETH_P_ALL));
  	if(rawsock < 0){
  		printf("raw socket error!\n");
  		exit(1);
  	}
  	while(1)
      {	
  		n = recvfrom(rawsock,buff,BUFFSIZE,0,NULL,NULL);
  		if(n<0)
          {
  			printf("receive error!\n");
  			exit(1);
  		}
  		
  		count++;
  		struct ip *ip = (struct ip*)(buff+14);
  		printf("%4d	%15s",count,inet_ntoa(ip->ip_src));
  		printf("%15s	%5d	%5d\n",inet_ntoa(ip->ip_dst),ip->ip_p,n);	
  
  		int i=0,j=0;
  		for(i=0;i<n;i++)
          {
  			if(i!=0 && i%16==0)
              {
  				printf("	");
  				for(j=i-16;j<i;j++)
                  {
  					if(buff[j]>=32&&buff[j]<=128)
  						printf("%c",buff[j]);
  					else printf(".");
  				}
  				printf("\n");
  			}
  			if(i%16 == 0) printf("%04x	",i);			
  			printf("%02x",buff[i]);
  	
  			if(i==n-1)
              {
  				for(j=0;j<15-i%16;j++) printf("  ");
  				printf("	");
  				for(j=i-i%16;j<=i;j++)
                  {
  					if(buff[j]>=32&&buff[j]<127)
                      	printf("%c",buff[j]);
                      else printf(".");
  
  				}
  			}
  		}
  		printf("\n\n");
  	}
  }	
  ```

原始套接字的接收流程主要是：先根据报文的L4层协议类型查找匹配的sock。从网卡接收回来的数据包的L4层protocol肯定是TCP、UDP、ICMP等有效的值，没有哪个数据包的protocol是IPPROTO_RAW，所以用protocol等于IPPROTO_RAW来新建原始套接字最后生成的hash值也就不会匹配到任何的数据包了。也就是说用**IPPROTO_RAW新建的套接字只适合发送，不能接收数据包**。

原始套接字构造发包的话，使用AF_INET时，是面向IP层的套接字，protocol字段定义在netinet/in.h中，常见的包括IPPROTO_TCP、IPPROTO_UDP、IPPROTO_ICMP和IPPROTO_RAW。

如果第三个参数是IPPROTO_TCP、IPPROTO_UDP、IPPROTO_ICMP，则我们所构造的报文从IP首部之后的第一个字节开始，IP首部由内核自己维护，首部中的协议字段会被设置为我们调用socket()函数时传递给它的protocol字段。

如果没有开启IP_HDRINCL选项，那么内核会帮忙处理IP头部。如果设置了IP_HDRINCL选项，那么用户需要自己生成IP头部的数据，其中IP首部中的标识字段和校验和字段总是内核自己维护。可以通过下面代码开启IP_HDRINCL：

```c++
const int on = 1;
if(setsockopt(fd,SOL_IP,IP_HDRINCL,&on,sizeof(int)) < 0)
{
  printf("set socket option error!\n");
}
```

- 在Linux上将协议设置为IPPROTO_RAW时，默认情况下，内核会设置IP_HDRINCL选项，因此不会在其前面加上自己的IP头。
- 当设置了IP_HDRINCL来指明由我们自己来填充IP头部，建socket时候指定的protocol(如：IPPROTO_TCP、IPPROTO_UDP)在发送时没有用到，并不会用指定的protocol来填充IP头部
- 当设置了IP_HDRINCL来指明由我们自己来填充IP头部，发送的数据没有再经过IP层，最后直接调用dst_output发送数据报了，所以如果发送的数据报长度超过了MTU那么不会有IP分片产生，发送失败，返回EMSGSIZE错误码。 相反的如果没有设置IP_HDRINCL来由我们自己定义IP头部，那么流程会走到ip_append_data函数进行IP分片。
- 当设置了IP_HDRINCL来指明由我们自己来填充IP头部，发送的数据报没有经过TCP层


## 六、基于pcap包嗅探器

libpcap 是一个**网络数据包捕获函数库**，功能非常强大，Linux 下著名的 tcpdump 就是以它为基础的。

安装：`sudo apt-get install libpcap-dev`

pcap嗅探器的总体布局，

1. 首先确定我们想要嗅探哪个接口，即网卡名称（字符串格式）
2. 初始化pcap。这是我们实际告诉pcap我们正在嗅探的设备。如果我们愿意，我们可以嗅探多个设备。我们如何区分它们？使用文件句柄。就像打开文件进行读写一样，我们必须命名我们的嗅探“会话”，以便我们可以将其与其他此类会话区分开来。
3. 如果我们只想嗅探特定的流量（例如：TCP/IP数据包，去往端口23的数据包等），我们必须创建一个规则集，“编译”并应用它。这是一个三阶段的过程，所有这些都是密切相关的。规则集保存在一个字符串中，并转换为pcap可以读取的格式（因此编译它。）编译实际上只是通过调用程序中的函数来完成的。它不涉及使用外部应用程序。然后我们告诉pcap将它应用于我们希望过滤的会话。
4. 最后，我们告诉pcap进入它的主要执行循环。在这种状态下，pcap会一直等到它收到了我们想要它的数据包。每次获取新数据包时，它都会调用我们已经定义的另一个函数。它调用的函数可以做任何我们想做的事情；它可以解析数据包并将其打印给用户，它可以将其保存在文件中，或者根本不执行任何操作。
5. 在满足我们的嗅探需求后，我们会结束会话并完成。

**设置设备**

`char *dev = "eth0";`

**创建嗅探会话**

创建嗅探会话的任务非常简单，我们使用pcap_open_live（）。

`pcap_t *pcap_open_live(char *device, int snaplen, int promisc, int to_ms,char *ebuf)`

第一个参数是我们在上一节中指定的设备。 snaplen是一个整数，它定义了pcap要捕获的最大字节数。  promisc，当设置为true时，将接口带入混杂模式（但是，即使设置为false，也可能在特定情况下接口处于混杂模式，无论如何）。  to_ms是读取超时（以毫秒为单位）（值为0表示没有time  out；至少在某些平台上，这意味着可能要等到足够数量的数据包到达才能看到任何数据包，因此应该使用非零值）。  最后，ebuf是一个字符串，我们可以存储任何错误消息（正如我们上面用errbuf所做的那样）。 该函数返回会话处理程序。

考虑以下代码段：

```text
 #include <pcap.h>
	 ...
	 pcap_t *handle;

	 handle = pcap_open_live(dev, BUFSIZ, 1, 1000, errbuf);
	 if (handle == NULL) {
		 fprintf(stderr, "Couldn't open device %s: %s\n", dev, errbuf);
		 return(2);
	 }
```

此代码片段打开存储在“dev”中的设备，告诉它读取BUFSIZ中指定的字节数（在pcap.h中定义）。我们告诉它将设备置于混杂模式，嗅探直到发生错误，如果有错误，将其存储在字符串errbuf中；使用该字符串来打印错误消息。

**过滤流量**

对流量进行过滤需要使用pcap_compile（）和pcap_setfilter（）。

这个过程非常简单。在我们已经调用pcap_open_live（）并进行有效的嗅探会话之后，我们可以应用我们的过滤器。为什么不使用我们自己的if/else  if语句？两个原因。首先，pcap的过滤器效率更高，因为它直接使用BPF过滤器；我们通过让BPF驱动程序直接执行来减少很多步骤。其次，这更容易:)

要编译过滤程序，我们使用pcap_compile（）。原型将其定义为：

`int pcap_compile(pcap_t *p, struct bpf_program *fp, char *str, int optimize, bpf_u_int32 netmask)`

第一个参数是我们的会话句柄（前一个例子中的pcap_t *句柄）。 接下来是对我们将存储过滤器的编译版本的位置的引用。  表达式本身就是常规字符串格式。 接下来是一个整数，它决定表达式是否应该“优化”（0为假，1为真。）最后，我们必须指定过滤器适用的网络的网络掩码。 失败时函数返回-1；所有其他值意味着成功。

在编译表达式之后，是时候应用它了。 输入pcap_setfilter（）。 按照我们解释pcap的格式，我们将看看pcap_setfilter（）原型：

`int pcap_setfilter(pcap_t *p, struct bpf_program *fp)`

这非常简单。 第一个参数是我们的会话处理程序，第二个参数是对表达式的编译版本的引用（可能是与pcap_compile（）的第二个参数相同的变量）。

一个代码示例有助于更好地理解：

```text
#include <pcap.h>
	 ...
	 pcap_t *handle;		/* Session handle */
	 char dev[] = "rl0";		/* Device to sniff on */
	 char errbuf[PCAP_ERRBUF_SIZE];	/* Error string */
	 struct bpf_program fp;		/* The compiled filter expression */
	 char filter_exp[] = "port 23";	/* The filter expression */
	 bpf_u_int32 mask;		/* The netmask of our sniffing device */
	 bpf_u_int32 net;		/* The IP of our sniffing device */

	 if (pcap_lookupnet(dev, &net, &mask, errbuf) == -1) {
		 fprintf(stderr, "Can't get netmask for device %s\n", dev);
		 net = 0;
		 mask = 0;
	 }
	 handle = pcap_open_live(dev, BUFSIZ, 1, 1000, errbuf);
	 if (handle == NULL) {
		 fprintf(stderr, "Couldn't open device %s: %s\n", dev, errbuf);
		 return(2);
	 }
	 if (pcap_compile(handle, &fp, filter_exp, 0, net) == -1) {
		 fprintf(stderr, "Couldn't parse filter %s: %s\n", filter_exp, pcap_geterr(handle));
		 return(2);
	 }
	 if (pcap_setfilter(handle, &fp) == -1) {
		 fprintf(stderr, "Couldn't install filter %s: %s\n", filter_exp, pcap_geterr(handle));
		 return(2);
	 }
```

该程序使嗅探器在设备rl0上以混杂模式嗅探来自或前往端口23的所有流量。

很少有嗅探器（如果有的话）实际使用pcap_next（）。通常，他们使用pcap_loop（）或pcap_dispatch（）。要理解这两个函数的使用，必须了解回调函数的概念。

回调函数不是什么新东西，并且在许多API中非常常见。回调函数背后的概念非常简单。假设有一个等待某种事件的程序。譬如，假设程序希望用户按下键盘上的键。每次按下一个键，就调用一个函数，然后确定要执行的操作。那么这个函数就是回调函数。每次用户按下某个键时，程序都会调用回调函数。在pcap中使用的回调函数，不是在用户按下键时调用，而是在pcap嗅探数据包时调用它们。可用于定义其回调的两个函数是pcap_loop（）和pcap_dispatch（）。  pcap_loop（）和pcap_dispatch（）在回调的使用方面非常相似。每当一个数据包被嗅探满足我们的过滤器要求时，它们都会调用回调函数（当然，如果存在任何过滤器。如果没有，那么所有被嗅探的数据包都会被发送到回调函数。）

pcap_loop（）的原型如下：

`int pcap_loop(pcap_t *p, int cnt, pcap_handler callback, u_char *user)`

第一个参数是我们的会话句柄。接下来是一个整数，它告诉pcap_loop（）在返回之前它应该嗅探多少个数据包（负值意味着它应该嗅探直到发生错误）。第三个参数是回调函数的名称（只是函数名，没有括号）。最后一个参数在某些应用程序中很有用，但很多时候只是设置为NULL。假设除了pcap_loop（）发送的参数外，有希望发送到回调函数的参数就可以使用最后一个参数。显然，必须对u_char指针进行类型转换，以确保正确。正如我们稍后将看到的，pcap使用了一些非常有趣的方法来以u_char指针的形式传递信息。

pcap_dispatch（）的用法几乎相同，他们之间的唯一区别是pcap_dispatch（）只处理它从系统接收的第一批数据包，而pcap_loop（）将继续处理数据包或批量数据包，直到数据包计数用完为止。

在我们提供使用pcap_loop（）的示例之前，我们必须检查回调函数的格式。我们不能随意定义我们的回调原型；否则，pcap_loop（）将不知道如何使用该函数。所以我们使用这种格式作为回调函数的原型：

`void got_packet(u_char *args, const struct pcap_pkthdr *header,  const u_char *packet)`

让我们更详细地看一下。 首先，该函数具有void返回类型。 这是合乎逻辑的，因为pcap_loop（）无论如何都不知道如何处理返回值。  第一个参数对应于pcap_loop（）的最后一个参数。  无论传递什么值，因为pcap_loop（）的最后一个参数在每次调用函数时都会传递给回调函数的第一个参数。  第二个参数是pcap头，它包含有关何时嗅探数据包的信息，它的大小等等。pcap_pkthdr结构在pcap.h中定义为：

```c++
struct pcap_pkthdr {
		struct timeval ts; /* time stamp */
		bpf_u_int32 caplen; /* length of portion present */
		bpf_u_int32 len; /* length this packet (off wire) */
	};
```

最后一个参数是它们中最有趣的， 它是另一个u_char类型的指针，它指向包含由pcap_loop（）嗅探的整个数据包的一大块数据的第一个字节。

**附：如何检测嗅探主机**

原理：举一个现实世界中的例子，说明我们检测处于混杂模式网络节点的方法。设想一下，在一个会议室中正在举行一个会议。某个人把耳朵贴在门上就可以进行窃听(嗅探^_^)。当她进行窃听(嗅探)时，会屏住呼吸，安静地聆听会议室内所有的发言。然而，如果此时会议室内有人忽然叫窃听者的名字：“XX太太”，她就可能答应“唉”。这听起来有点好笑，但是完全可以用于网络嗅探行为的检测。网络进行网络嗅探的节点会接收网络的所有报文，因此其内核可能对某些本该被硬件过滤的分组作出错误回应。根据这个原理，我们可以通过检查节点对ARP报文的响应来检测网络的嗅探行为。

具体来说，我们构造一个ARP查询包，其目的地址不是广播地址，然后向网络上的各个节点发送这个ARP查询包，最后通过各个节点的回应来判断是否处于混杂模式。

首先我们讨论一下整个ARP请求/响应的操作过程。首先，产生一个ARP查询包来解析192.168.1.10的硬件地址。为了使网络上的所有节点都能够收到这个查询包，把这个包的目的地址设置为广播地址。理论上，只有IP地址为192.168.1.10的网卡才能对这个查询包进行响应。

进一步设想，如果我们把这个查询包的目的地址(以太网地址)设置为另外的地址,而不是原来的广播地址又将如何？例如：我们把查询包的目的地址设置为00-00-00-00-00-01会发生什么？处于正常模式下网络节点的以太网卡会认为这个查询包是发往其它主机的，其硬件过滤器会拒绝接收这个包；然而，如果这个网络节点(192.168.1.10)的以太网卡处于混杂模式(promiscuous  mode)下，那么即使以太网地址不匹配，其硬件过滤器也不进行任何过滤，从而使这个查询包能够进入到系统的内核。因为这个节点的IP地址和查询包的要查询IP地址相同，其内核就会认为ARP查询包到达，应该作出应答。

## 参考资料

https://zhuanlan.zhihu.com/p/59296026