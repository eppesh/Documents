[toc]

# Basic Knowledge

## User space & Kernel space

![Five IO models of UNIX](https://imgs.developpaper.com/imgs/4167911056-5ebe4bed1ac4e_articlex.png)

[Kernel space](http://www.ruanyifeng.com/blog/2016/12/user_space_vs_kernel_space.html) 是 Linux 内核的运行空间，User space 是用户程序的运行空间。Kernel space 可以执行任意命令，调用系统的一切资源（如：读写本地文件需要访问磁盘，创建socket需要网卡，CPU管理，键盘工作等）；User space 只能执行简单的运算，不能直接调用系统资源，必须通过系统接口（又称`system call`），才能向内核发出指令。

<img src="https://bigbyto.gitee.io/assets/images/fd/us_ks.png" alt="pic" style="zoom:50%;" />

```c++
str = "my string" // 用户空间
x = x + 2
file.write(str) // 切换到内核空间

y = x + 4 // 切换回用户空间
```

查看 CPU 时间在 User space 与 Kernel Space 之间的分配情况，可以使用`top`命令。它的第三行输出就是 CPU 时间分配统计。

![pic](https://www.ruanyifeng.com/blogimg/asset/2016/bg2016120203-1.png)

其中，第一项`24.8 us`（user 的缩写）就是 CPU 消耗在 User space 的时间百分比，第二项`0.5 sy`（system 的缩写）是消耗在 Kernel space 的时间百分比。

随便也说一下其他 6 个指标的含义。

> - `ni`：niceness 的缩写，CPU 消耗在 nice 进程（低优先级）的时间百分比
> - `id`：idle 的缩写，CPU 消耗在闲置进程的时间百分比，这个值越低，表示 CPU 越忙
> - `wa`：wait 的缩写，CPU 等待外部 I/O 的时间百分比，这段时间 CPU 不能干其他事，但是也没有执行运算，这个值太高就说明外部设备有问题
> - `hi`：hardware interrupt 的缩写，CPU 响应硬件中断请求的时间百分比
> - `si`：software interrupt 的缩写，CPU 响应软件中断请求的时间百分比
> - `st`：stole time 的缩写，该项指标只对虚拟机有效，表示分配给当前虚拟机的 CPU 时间之中，被同一台物理机上的其他虚拟机偷走的时间百分比

参考资料：

- [Kernel space](http://www.ruanyifeng.com/blog/2016/12/user_space_vs_kernel_space.html) ；
- [Five I/O Models of UNIX](https://developpaper.com/five-io-models-of-unix/); 

## File Descriptor

`File Desciptor` (简称`fd`) 又叫文件描述符，用一个非负整数表示。它指向了由系统内核维护的一个`file table`中的某个条目（`entry`)。在Windows中，`file descriptors`又被称为`file handles`（句柄）。

`fd`之所以存在，是因为用户程序无法直接访问硬件；因此当程序向内核发起`system call`打开一个文件时，在用户进程中必须有一个东西标识着打开的文件，这个东西就是`fd`.

和`fd`相关的一共有3张表，分别是file descriptor、file table、inode table，如下图所示。

![~replace~https://user-images.githubusercontent.com/3600657/103748658-67d36100-503f-11eb-960b-e77683e751f7.png](https://bigbyto.gitee.io/assets/images/fd/file-descriptor.jpg)

- file descriptors

  file descriptors table由用户进程所有，每个进程都有一个这样的表，这里记录了进程打开的文件所代表的fd，fd的值映射到file table中的条目(entry)。

  另外，每个进程都会预留3个默认的fd: stdin、stdout、stderr;它们的值分别是0、1，2。

  | Integer value |                          Name                           | symbolic constant | file stream |
  | :-----------: | :-----------------------------------------------------: | :---------------: | :---------: |
  |       0       |  [Standard input](https://en.wikipedia.org/wiki/Stdin)  |   STDIN_FILENO    |    stdin    |
  |       1       | [Standard output](https://en.wikipedia.org/wiki/Stdout) |   STDOUT_FILENO   |   stdout    |
  |       2       | [Standard error](https://en.wikipedia.org/wiki/Stderr)  |   STDERR_FILENO   |   stderr    |

- file table

  file table是全局唯一的表，由系统内核维护。这个表记录了所有进程打开的文件的状态(是否可读、可写等状态)，同时它也映射到inode table中的entry。

- inode table

  inode table同样是全局唯一的，它指向了真正的文件地址(磁盘中的位置)，每个entry全局唯一。

When a program asks to **open** a file — or another data resource, like a [network socket](https://www.computerhope.com/jargon/n/network-socket.htm) — the [kernel](https://www.computerhope.com/jargon/k/kernel.htm):

1. Grants access. (允许程序访问)
2. Creates an entry in the global file table. (在`file table`表中创建一个`entry`)
3. Provides the software with the location of that entry. (告诉程序刚才的`entry`的地址)

当程序再次发起`read()`system call时，需要把相关的fd传给内核，内核定位到具体的文件(fd –> file table –> inode table)向磁盘发起读取请求，再把读取到的数据返回给程序处理。

**小结：**

1. 程序无法直接访问硬件资源，对硬件资源的访问（如对磁盘的读写）需要发起`system call`，让`kernel`处理；
2. 大部分和`I/O`有关的`system call`，都需要把`file descriptor (fd)` 作为参数传递给`kernel`。

参考资料：

- [理解Linux中的file descriptor](https://wiyi.org/linux-file-descriptor.html); 

## What's a Socket?

### Socket Definition

From: [Oracle - Socket Definition](https://docs.oracle.com/javase/tutorial/networking/sockets/definition.html); 

Normally, a server runs on a specific computer and has a socket that is bound to a specific port number. The server just waits, listening to the socket for a client to make a connection request.

On the client-side: The client knows the hostname of the machine on which the server is running and the port number on which the server is listening. To make a connection request, the client tries to rendezvous with the server on the server's machine and port. The client also needs to identify itself to the server so it binds to a local port number that it will use during this connection. This is usually assigned by the system.

**![A client's connection request](https://docs.oracle.com/javase/tutorial/figures/networking/5connect.gif)**

If everything goes well, the server accepts the connection. Upon acceptance, the server gets a new socket bound to the same local port and also has its remote endpoint set to the address and port of the client. It needs a new socket so that it can continue to listen to the original socket for connection requests while tending to the needs of the connected client.

![The connection is made](https://docs.oracle.com/javase/tutorial/figures/networking/6connect.gif)

On the client side, if the connection is accepted, a socket is successfully created and the client can use the socket to communicate with the server.

The client and server can now communicate by writing to or reading from their sockets.

------

Definition:

A *socket* is **one endpoint** of a two-way communication link between two programs running on the network. A socket is bound to a port number so that the TCP layer can identify the application that data is destined to be sent to.

------

An **endpoint** is a **combination of an IP address and a port number**. Every TCP connection can be uniquely identified by its two endpoints. That way you can have multiple connections between your host and the server.



---

From: [Bogotobogo - Socket Server Client](https://www.bogotobogo.com/cplusplus/sockets_server_client.php); 

**Sockets** let apps attach to the local network at different ports. 

![pic](https://www.bogotobogo.com/cplusplus/images/socket/socket_port.png)

### Server/Client Applications

The basic mechanisms of client-server setup are:

1. A client app send a request to a server app.
2. The server app returns a reply.
3. Some of the basic data communications between client and server are:
   1. File transfer - sends name and gets a file.
   2. Web page - sends url and gets a page.
   3. Echo - sends a message and gets it back.

### Server Socket

1. create a socket - Get the file descriptor!
2. bind to an address -What port am I on?
3. listen on a port, and wait for a connection to be established.
4. accept the connection from a client.
5. send/recv - the same way we read and write for a file.
6. shutdown to end read/write.
7. close to releases data.

### Client Socket

1. create a socket.
2. `bind\*` - this is probably be unnecessary because you're the client, not the server.
3. connect to a server.
4. `send/recv` - repeat until we have or receive data
5. shutdown to end read/write.
6. close to releases data.

### Socket Functions

Sockets, in C, behaves like files because they use file descriptors to identify themselves. Sockets behave so much like files that we can use the `read()` and `write()` to receive and send data using socket file descriptors.

There are several functions, however, specifically designed to handle sockets. These functions have their prototypes defined in `/usr/include/sys/sockets.h`.

1. `int socket(int domain, int type, int protocol)`

   Used to create a new socket, returns a file descriptor for the socket or `-1` on error.
   It takes three parameters:

   1. `domain`: the protocol family of socket being requested
   2. `type`: the type of socket within that family
   3. and the `protocol`.

   The parameters allow us to say what kind of socket we want (IPv4/IPv6, stream/datagram(TCP/UDP)).

   1. The protocol family should be `AF_INET` or `AF_INET6`
   2. and the protocol type for these two families is
      either `SOCK_STREAM` for TCP/IP or `SOCK_DGRAM` for UDP/IP.
   3. The protocol should usually be set to zero to indicate that the default protocol should be used.

2. `int bind(int fd, struct sockaddr \*local_addr, socklen_t addr_length)`

   Once we have a socket, we might have to associate that socket with a port on our local machine.
   The port number is used by the kernel to match an incoming packet to a certain process's socket descriptor.
   A server will call bind() with the address of the local host and the port on which it will listen for connections.
   It takes file descriptor (previously established socket), a pointer to (the address of) a structure containing the details of the address to bind to, the value INADDR_ANY is typically used for this, and the length of the address structure.
   The particular structure that needs to be used will depend on the protocol, which is why it is passed by the pointer.
   So, this bind() call will bind the socket to the current IP address on port, portno
   Returns 0 on success and -1 on error.

3. `int listen(int fd, int backlog_queue_size)`

   Once a server has been bound to an address, the server can then call listen() on the socket.
   The parameters to this call are the socket (fd) and the maximum number of queued connections requests up to backlog_queue_size.
   Returns 0 on success and -1 on error.

4. `int accept(int fd, struct sockaddr \*remote_host, socklen_t addr_length)`

   Accepts an incoming connection on a bound socket. The address information from the remote host is written into the remote_host structure and the actual size of the address structure is written into *addr_length.
   In other words, this accept() function will write the connecting client's address info into the address structure.
   Then, returns a new socket file descriptor for the accepted connection.
   So, the original socket file descriptor can continue to be used for accepting new connections while the new socket file descriptor is used for communicating with the connected client.
   This function returns a new socket file descriptor to identify the connected socket or -1 on error.

   Here is the description from the man page:
   "It extracts the first connection request on the queue of pending connections for the listening socket, sockfd, creates a new connected socket, and returns a new file descriptor referring to that socket. The newly created socket is not in the listening state. The original socket sockfd is unaffected by this call".

   If no pending connections are present on the queue, and the socket is not marked as nonblocking, accept() blocks the caller until a connection is present.

5. `int connect(int fd, struct sockaddr \*remote_host, socklen_t addr_length)`

   Connects a socket (described by file descriptor fd) to a remote host.
   Returns 0 on success and -1 on error.

   This is a blocking call. That's because when we issue a call to connect(), our program doesn't regain control until either the connection is made, or an error occurs. For example, let's say that we're writing a web browser. We try to connect to a web server, but the server isn't responding. So, we now want the connect() API to stop trying to connect by clicking a stop button. But that can't be done. It waits for a return which could be 0 on success or -1 on error.

6. `int send(int fd, void \*buffer, size_t n, int flags)`

   Sends n bytes from *buffer to socket fd.
   Returns the number of bytes sent or -1 on error.

7. `int receive(int fd, void \*buffer, size_t n, int flags)`

   Receives n bytes from socket fd into *buffer.
   Returns the number of bytes received or -1 on error.

   This is another blocking call. In other words, when we call recv() to read from a stream, control isn't returned to our program until at least one byte of data is read from the remote site. This process of waiting for data to appear is referred to as blocking. The same is true for the write() and the connect() APIs, etc. When we run those blocking APIs, the connection "blocks" until the operation is complete.

The following server code listens for TCP connections on port 20001. When a client connects, it sends the message "Hello world!", and then it receives data from client.

server.c  [server.c](https://www.bogotobogo.com/cplusplus/sockets_server_client.php); 

```c++
#include <cstdio>
#include <cstring>
#include <iostream>
#include <sys/socket.h>
#include <sys/types.h>
#include <unistd.h>
#include <arpa/inet.h>

void error(const char *msg)
{
    perror(msg);
    exit(1);
}

int main(int argc, char *argv[])
{
    int sockfd = 0;
    int new_sockfd = 0;
    int port_no = 0;
    socklen_t cliient_len;
    char buffer[256] = { 0 };
    sockaddr_in server_addr;
    sockaddr_in client_addr;

    int n = 0;
    if (argc < 2)
    {
        fprintf(stderr, "ERROR, no port provided\n");
        exit(1);
    }

    // create a socket
    // socket(int domain, int type, int protocol)
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0)
    {
        error("ERROR opening socket");
    }

    // clear address structure
    memset((char *)&server_addr, 0, sizeof(server_addr));
    port_no = atoi(argv[1]);

    // setup the host_addr structure for use in bind call
    server_addr.sin_family = AF_INET;                       // server byte order
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);               // automatically be filled with current host's IP address
    server_addr.sin_port = htons(port_no);

    // This bind() call will bind the socket to the current IP address on port
    if (bind(sockfd, (sockaddr *)&server_addr, sizeof(server_addr)) < 0)
    {
        error("ERROR on binding");
    }

    // This listen() call tells the socket to listen to the incoming connections.
    // The listen() function places all incoming connection into a backlog queue
    // until accept() call accepts the connection.
    // Here, we set the maximum size for the backlog queue to 5
    listen(sockfd, 5);

    // The accept() call actually accepts an incoming connection
    cliient_len = sizeof(client_addr);

    // This accept() function will write the connecting client's address info into
    // the address structure and the size of that structure is client_len.
    // The accept() returns a new socket file descriptor for the accepted connection.
    // So, the original socket file descriptor can continue to be used for 
    // accepting new connections while the new socket file descriptor is used for
    // communicating with the connected client.
    new_sockfd = accept(sockfd, (sockaddr *)&client_addr, &cliient_len);
    if (new_sockfd < 0)
    {
        error("ERROR on accept");
    }

    std::cout << "server: got connection from " << inet_ntoa(client_addr.sin_addr) <<
        " port " << ntohs(client_addr.sin_port) << std::endl;

    // This send() function sends the 13 bytes of the string to the new socket 
    send(new_sockfd, "Hello, world!\n", 13, 0);

    memset(buffer, 0, 256);

    n = read(new_sockfd, buffer, 255);
    if (n < 0)
    {
        error("ERROR reading from socket");
    }

    std::cout << "Here is the message: " << buffer << std::endl;

    close(new_sockfd);
    close(sockfd);
    return 0;
}
```

```
serv_addr.sin_family = AF_INET;
serv_addr.sin_addr.s_addr = INADDR_ANY;
serv_addr.sin_port = htons(portno);
```

1. sin_family = specifies the address family, usually the constant AF_INET
2. sin_addr = holds the IP address returned by inet_addr() to be used in the socket connection.
3. sin_port = specifies the port number and must be used with htons() function that converts the host byte order to network byte order so that it can be transmitted and routed properly when opening the socket connection. The reason for this is that computers and network protocols order their bytes in a non-compatible fashion.

The lines above set up the serv_addr structure for use in the bind call.

```
/* Structure describing a generic socket address. */
struct sockaddr
{
    __SOCKADDR_COMMON (sa_);    /* Common data: address family and length. */
    char sa_data[14];           /* Address data. */
};
```

The address family is AF_INET, since we are using IPv4 and the sockaddr_in structure. The short integer value for port must be converted into network byte order, so the htons() (Host-to-Network Short) function is used.



The bind() call passes the socket file descriptor, the address structure, and the length of the address structure. This call will bind the socket to the current IP address on port 20001.

```
if (bind(sockfd, (struct sockaddr *) &serv;_addr,
           sizeof(serv_addr)) < 0) error("ERROR on binding");
```

The listen() call tells the socket to listen for incoming connections, and a subsequent accept() call actually accepts an incoming connection. The listen() function places all incoming connections into a backlog queue until an accept() call accepts the connections. The last argument to the listen() call sets the maximum size for the backlog queue.

```
listen(sockfd,5);
clilen = sizeof(cli_addr);
newsockfd = accept(sockfd, (struct sockaddr *) &cli;_addr, &clilen;);
if (newsockfd < 0) error("ERROR on accept");
```

The final argument of the accept() is a pointer to the size of the address structure. This is because the accept() function will write the connecting client's address information into the address structure and the size of that structure is clilen. The accept() function returns a new socket file descriptor for the accepted connection:

```
newsockfd = accept(sockfd, 
             (struct sockaddr *) &cli;_addr,&clilen;);
```

This way, the original socket file descriptor can continue to be used for accepting new connections, while the new socket file descriptor is used for communicating with the connected client.

The send() function sends the 13 bytes of the string Hello, world\n" to the new socket that describes the new connection.

```
send(newsockfd, "Hello, world!\n", 13,0);
```

To compile, the server.c:

```
g++ -o server server.c
```

and to run

```
./server port# 
```

client.c

```c++
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>

void error(const char *msg)
{
    perror(msg);
    exit(0);
}

int main(int argc, char *argv[])
{
    int sockfd, portno, n;
    struct sockaddr_in serv_addr;
    struct hostent *server;

    char buffer[256];
    if (argc < 3) {
       fprintf(stderr,"usage %s hostname port\n", argv[0]);
       exit(0);
    }
    portno = atoi(argv[2]);
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0) 
        error("ERROR opening socket");
    server = gethostbyname(argv[1]);
    if (server == NULL) {
        fprintf(stderr,"ERROR, no such host\n");
        exit(0);
    }
    bzero((char *) &serv;_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    bcopy((char *)server->h_addr, 
         (char *)&serv;_addr.sin_addr.s_addr,
         server->h_length);
    serv_addr.sin_port = htons(portno);
    if (connect(sockfd, (struct sockaddr *) &serv;_addr, sizeof(serv_addr)) < 0) 
        error("ERROR connecting");
    printf("Please enter the message: ");
    bzero(buffer,256);
    fgets(buffer,255,stdin);
    n = write(sockfd, buffer, strlen(buffer));
    if (n < 0) 
         error("ERROR writing to socket");
    bzero(buffer,256);
    n = read(sockfd, buffer, 255);
    if (n < 0) 
         error("ERROR reading from socket");
    printf("%s\n", buffer);
    close(sockfd);
    return 0;
}
```

To compile

```
g++ -o client client.c
```

and to run

```
./client hostname port# 
```

---

First, we run server.c as in

```
$ ./server 20001
```

Then, on client side

```
$ ./client myhostname 20001
Please enter the message: 
```

Then, server side has the following message when connected successfully:

```
$ ./server 20001
server: got connection from 127.0.0.1 port 47173
```

Then, on client side

```
$ ./client myhostname 20001
Please enter the message: Hello from client
```

Then, server side has the following message:

```
$ ./server 20001
server: got connection from 127.0.0.1 port 47173
Here is the message: Hello from Client
```

Clent side gets message (Hello, world!) from the server:

```
$ ./client myhostname 20001
Please enter the message: Hello from Client
Hello, world!
```

To run this codes, we don't need two machines. One is enough!

### Head file

相关头文件及说明；

- `<netdb.h>`: 设置及获取域名等函数；Linux适用；

  ```c++
  /*通过IP地址获得主机有关的网络信息*/
  struct hostent*gethostbyaddr(const void *addr, size_t len, int type);
  /*通过主机名获得主机的网络信息*/
  struct hostent*gethostbyname(const char *name);
  ```

- `<sys/types.h>`: 系统数据类型；

- `<sys/socket.h>`: 提供socket函数及数据结构；

- `<cstring> or <string.h>`: `memset`等函数的头文件；[CPlusPlus - cstring](http://www.cplusplus.com/reference/cstring/); 

- `<netinet/in.h>`: `sockaddr_in`结构体等的定义；`htons`等函数的头文件；

- `<unistd.h>`: `read(), write(), close()`等函数的头文件；

- `<arpa/inet.h>`: `inet_ntoa(), inet_addr()`等函数的头文件；

常用头文件示例：

```c++
// server
#include <arpa/inet.h>			// inet_ntoa()
#include <cstdio>				// printf, perror()
#include <cstring>				// memeset()
#include <sys/socket.h>			// socket()
#include <sys/types.h>
#include <unistd.h>				// read(), write(), close()


// client
#include <cstdio>
#include <cstdlib>				// atoi()
#include <cstring>				// memeset()
#include <netinet/in.h>			// htons()
#include <netdb.h>				// gethostbyname()
#include <sys/socket.h>			// socket()
#include <sys/types.h>
#include <unistd.h>				// read(), write(), close()
```



### Summary

Here is the summary of key concepts:

1. Socket is a way of speaking to other programs using standard file descriptors.
2. Where do we get the file descriptor for network communication?
   Well, we make a call to the socket() system routine.
   After the socket() returns the socket descriptor, we start communicate through it using the specialized send()/recv() socket API calls.
3. A TCP socket is an endpoint instance.
4. A TCP socket is not a connection, it is the endpoint of a specific connection.
5. A TCP connection is defined by two endpoints aka sockets.
6. The purpose of ports is to differentiate multiple endpoints on a given network address.
7. The port numbers are encoded in the transport protocol packet header, and they can be readily interpreted not only by the sending and receiving computers, but also by other components of the networking infrastructure. In particular, firewalls are commonly configured to differentiate between packets based on their source or destination port numbers as in port forwarding.
8. It is the socket pair (the 4-tuple consisting of the client IP address, client port number, server IP address, and server port number) that specifies the two endpoints that uniquely identifies each TCP connection in an internet.
9. Only one process may bind to a specific IP address and port combination using the same transport protocol. Otherwise, we'll have port conflicts, where multiple programs attempt to bind to the same port numbers on the same IP address using the same protocol.

---

To connect to another machine, we need a socket connection.

## What's a Connection?

From: [Bogotobogo - Socket Server Client](https://www.bogotobogo.com/cplusplus/sockets_server_client.php); 

A relationship between two machines, where two pieces of software know about each other. Those two pieces of software know how to communicate with each other. In other words, they know how to send bits to each other.
A socket connection means the two machines have information about each other, including network location (IP address) and TCP port. (If we can use analogy, IP address is the phone number and the TCP port is the extension).

A socket is an object similar to a file that allows a program to accept incoming connections, make outgoing connections, and send and receive data. Before two machines can communicate, both must create a socket object.

A socket is a resource assigned to the server process. The server creates it using the system call socket(), and it can't be shared with other processes.

## TCP vs UDP

From: [Bogotobogo - Socket Server Client](https://www.bogotobogo.com/cplusplus/sockets_server_client.php); 

There are several different types of socket that determine the structure of the transport layer. The most common types are stream sockets and datagram sockets.



**![tcp_udp_headers.jpg](https://www.bogotobogo.com/cplusplus/images/socket/tcp_udp_headers.jpg)**

| TCP (Streams)                                                | UDP (Datagrams)                                              |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Connections                                                  | Connectionless sockets We don't have to maintain an open connection as we do with stream sockets. We just build a packet, put an IP header on it with destination information, and send it out. No connection needed: datagram sockets also use IP for routing, but they don't use TCP  *note: can be connect()'d if we really want. |
| SOCK_STREAM                                                  | SOCK_DGRAM                                                   |
| If we output two items into the socket in the order "A, B", they will arrive in the order "A, B" at the opposite end. They will also be error-free. | If we send a datagram, it may arrive. But it may arrive out of order. If it arrives, however, the data within the packet will be error-free. |
|                                                              | Why would we use an unreliable protocol? Speed! We just ignore the dropped packets. |
| Arbitrary length content                                     | Limited message size                                         |
| Flow control matches sender to receiver                      | Can send regardless of receiver state                        |
| Congestion control matches sender to network                 | Can send regardless of network state                         |
| http, telnet                                                 | tftp (trivial file transfer protocol), dhcpcd (a DHCP client), multiplayer games, streaming audio, video conferencing  *note: They use complementary protocol on top of UDP to get more reliability |

1. Stream Sockets
   Stream sockets provide reliable two-way communication similar to when we call someone on the phone. One side initiates the connection to the other, and after the connection is established, either side can communicate to the other.
   In addition, there is immediate confirmation that what we said actually reached its destination.
   Stream sockets use a Transmission Control Protocol (TCP), which exists on the transport layer of the Open Systems Interconnection (OSI) model. The data is usually transmitted in packets. TCP is designed so that the packets of data will arrive without errors and in sequence.
   Webservers, mail servers, and their respective client applications all use TCP and stream socket to communicate.

2. Datagram Sockets
   Communicating with a datagram socket is more like mailing a letter than making a phone call. The connection is one-way only and unreliable.
   If we mail several letters, we can't be sure that they arrive in the same order, or even that they reached their destination at all. Datagram sockets use User Datagram Protocol (UDP). Actually, it's not a real connection, just a basic method for sending data from one point to another.
   Datagram sockets and UDP are commonly used in networked games and streaming media.

   Though in this section, we mainly put focus on applications that maintain connections to their clients, using connection-oriented TCP, there are cases where the overhead of establishing and maintaining a socket connection is unnecessary.
   For example, just to get the data, a process of creating a socket, making a connection, reading a single response, and closing the connection, is just too much. In this case, we use UDP.
   Services provided by UDP are typically used where a client needs to make a short query of a server and expects a single short response. To access a service from UDP, we need to use the UDP specific system calls, sendto() and recvfrom() instead of read() and write() on the socket.

   UDP is used by app that doesn't want reliability or bytestreams.

   1. Voice-over-ip (unreliable) such as conference call. (visit [VoIP](http://www.bogotobogo.com/VideoStreaming/VoIP.php))
   2. DNS, RPC (message-oriented)
   3. DHCP (bootstrapping)

## Client/Server

The client-server model distinguishes between applications as well as devices. Network clients make requests to a server by sending messages, and servers respond to their clients by acting on each request and returning results.

For example, let's talk about telnet.
When we connect to a remote host on port 23 with telnet (the client), a program on that host (called telnetd, the server) springs to life. It handles the incoming telnet connection, sets us up with a login prompt, etc.

One `server` generally supports numerous `clients`, and multiple servers can be networked together in a pool to handle the increased processing load as the number of clients grows.

Some of the most popular applications on the Internet follow the client-server model including email, FTP and Web services. Each of these clients features a user interface and a client application that allows the user to connect to servers. In the case of email and FTP, users enter a computer name (or an IP address) into the interface to set up connections to the server.

The **steps** to establish a socket on the **server** side are:

1. **Create a socket** with the socket() system call.

   ```c++
   #include <sys/socket.h>
   int sockfd = socket(AF_INET, SOCK_STREAM, 0);
   ```

2. The `server` process gives the socket a name. In linux file system, local sockets are given a filename, under `/tmp` or `/usr/tmp` directory. For network sockets, the filename will be a service identifier, port number, to which the clients can make connection. This identifier allows to route incoming connections (which has that the port number) to connect server process. **A socket is named** using `bind()` system call. 

   用`bind()`将 `socket`和一个`Endpoint (IP + Port)`绑定；

   ```c++
   #include <arpa/inet.h>		// inet_ntoa()
   #include <netinet/in.h>		// sockaddr_in;htons();
   
   sockaddr_in server_addr;
   memset((char *)&server_addr, 0, sizeof(server_addr));
   
   server_addr.sin_family = AF_INET;
   server_addr.sin_addr.s_addr = htonl(INADDR_ANY);// automatically be filled with current host's IP address
   //server_addr.sin_addr.s_addr = inet_addr("127.0.0.1"); // 这两种方式均可
   server_addr.sin_port = htons(8888);
   
   bind(sockfd, (sockaddr *)&server_addr, sizeof(server_addr)
   ```

3. The `server` process then waits for a `client` to **connect** to **the named socket**（Sean注：这个`socket`，即上面的`sockfd`，主要用于监听是否有客户端连接上来，因此可重命名为`listening_sockfd`）, which is basically listening for connections with the `listen()` system call. If there are more than one client are trying to make connections, the `listen()` system call make a queue.
   The machine receiving the connection (the server) must bind its socket object to a known port number. A port is a 16-bit number in the range 0-65535 that's managed by the operating system and used by clients to uniquely identify servers. Ports 0-1023 are reserved by the system and used by common network protocols.
   
4. **Accept** a connection with the `accept()` system call. At `accept()`, **a new socket** is created that is distinct from the named socket. This new socket is used solely for communication with this particular client.
   For TCP servers, the socket object used to receive connections **is not the same socket** used to perform subsequent communication with the client. In particular, the `accept()` system call returns a new socket object that's actually used for the connection. This allows a server to manage connections from a large number of clients simultaneously.
   
   ```c++
   sockaddr_in client_addr;	// 用于保存客户端的地址信息
   socklen_t cliient_len = sizeof(client_addr);
   memset((char *)&client_addr, 0, sizeof(client_addr));
   
   int new_sockfd = accept(sockfd, (sockaddr *)&client_addr, &cliient_len);
   ```
   
   > Sean注：
   >
   > 1. 前面利用`socket()`创建并在`bind(), listen()`中使用的那个`socket`是监听的（`listening_sockfd`）；
   > 2. 每个`connect()`成功后返回的`new_sockfd`就是这条`connection`的客户端的`socket`，即`client1_sockfd`；
   
5. **Send** and **receive** data.

6. The named socket remains for further connections from other clients. A typical web server can take advantage of multiple connections. In other words, it can serve pages to many clients at once. But for a simple server, further clients wait on the listen queue until the server is ready again.

The steps to establish a socket on the client side are:

1. Create a socket with the socket() system call.
2. Connect the socket to the address of the server using the connect() system call.
3. Send and receive data. There are a number of ways to do this, but the simplest is to use the read() and write() system calls.

**![Socket_API.png](https://www.bogotobogo.com/cplusplus/images/socket/Socket_API.png)**

TCP communication:

**![TCP_IP_socket_diagram.png](https://www.bogotobogo.com/cplusplus/images/socket/TCP_IP_socket_diagram.png)**

UDP communication - clients and servers don't establish a connection with each other:

**![UDP_socket_diagram.png](https://www.bogotobogo.com/cplusplus/images/socket/UDP_socket_diagram.png)**

call block, go to [Blocking socket vs non-blocking socket ](https://www.bogotobogo.com/cplusplus/sockets_server_client.php#block_vs_non_blocking).

## send(),recv() 与 write(), read()

在socket中都可用于发送/接收数据；

```c++
// 下面形式不是标准原型，只是为了说明差异
int send(int sockfd,void *buf,int len,int flags)
int recv(int sockfd,void *buf,int len,int flags)
// flags 可以是0或是下面的组合
// MSG_DONTROUTE: 不查找路由表 (send()使用；这个标志告诉IP，目的主机在本地网络上，没有必要查找表，这个标志一般用在网络诊断和路由程序里面。)
// MSG_OOB: 接受或发送带外数据 (表示可以接收和发送带外数据)
// MSG_PEEK: 查看数据,并不从系统缓冲区移走数据 (recv()使用；表示只是从系统缓冲区中读取内容，而不清除系统缓冲区的内容。这样在下次读取的时候，依然是一样的内容，一般在有多个进程读写数据的时候使用这个标志)
// MSG_WAITALL: 等待任何数据 (recv()使用；表示等到所有的信息到达时才返回，使用这个标志的时候，recv返回一直阻塞，直到指定的条件满足时，或者是发生了错误)
    
int write(int fd,void *buf,int len)		// 把buf中的len字节内容写入fd(file descriptor)
int read (int fd,void *buf,int len)		// 从fd(file descriptor)中读取内容
```

区别：

- 头文件不同：`send(), recv()` 头文件是：`<sys/socket.h>`; `write(), read()` 头文件是：`<unistd.h>`; 
- 参数不同：`send(), recv()` 多了第4个参数，用来控制读写操作；当flags=0时，功能同`write(), read()`一样；

`send(), recv(), write(), read()`用于TCP连接，在UDP中使用`sendto(), recvfrom()`；

## recv() & recvfrom()

参考：[recv(), recvfrom()](https://www.gta.ufrj.br/ensino/eel878/sockets/recvman.html); 

```c++
#include <sys/types.h>
#include <sys/socket.h>

ssize_t recv(int s, void *buf, size_t len, int flags);
ssize_t recvfrom(int s, void *buf, size_t len, int flags,
                 struct sockaddr *from, socklen_t *fromlen);
```

Once you have a socket up and connected, you can read incoming data from the remote side using the `recv()` (for TCP `SOCK_STREAM` sockets) and `recvfrom()` (for UDP `SOCK_DGRAM` sockets).

Both functions take the socket descriptor `s`, a pointer to the buffer `buf`, the size (in bytes) of the buffer `len`, and a set of `flags` that control how the functions work.

Additionally, the `recvfrom()` takes a `struct sockaddr*`, `from` that will tell you where the data came from, and will fill in `fromlen` with the size of `struct sockaddr`. (You must also initialize `fromlen` to be the size of `from` or `struct sockaddr`.)

So what wonderous flags can you pass into this function? Here are some of them, but you should check your local man pages for more information and what is actually supported on your system. You bitwise-or these together, or just set `flags` to `0` if you want it to be a regular vanilla `recv()`.

|               |                                                              |
| ------------- | ------------------------------------------------------------ |
| `MSG_OOB`     | Recieve Out of Band data. This is how to get data that has been sent to you with the `MSG_OOB` flag in `send()`. As the recieving side, you will have had signal `SIGURG` raised telling you there is urgent data. In your handler for that signal, you could call `recv()` with this `MSG_OOB` flag. |
| `MSG_PEEK`    | If you want to call `recv()` "just for pretend", you can call it with this flag. This will tell you what's waiting in the buffer for when you call `recv()` "for real" (i.e. *without* the `MSG_PEEK` flag. It's like a sneak preview into the next `recv()` call. |
| `MSG_WAITALL` | Tell `recv()` to not return until all the data you specified in the `len` parameter. It will ignore your wishes in extreme circumstances, however, like if a signal interrupts the call or if some error occurs or if the remote side closes the connection, etc. Don't be mad with it. |

Return Value:

Returns the number of bytes actually recieved (which might be less than you requested in the `len` paramter), or `-1` on error (and `errno` will be set accordingly.)

If the remote side has closed the connection, `recv()` will return `0`. This is the normal method for determining if the remote side has closed the connection. Normality is good, rebel! `if(recv() == 0) { // the connection has been closed}`

> Sean注：常使用 `recv() == 0` 来判断 connection 是否已关闭；

示例：

```c++
int s1, s2;
int byte_count, fromlen;
struct sockaddr_in addr;
char buf[512];

// show example with a TCP stream socket first
s1 = socket(PF_INET, SOCK_STREAM, 0);

// info about the server
addr.sin_family = AF_INET;
addr.sin_port = htons(3490);
inet_aton("10.9.8.7", &addr.sin_addr);

connect(s1, &addr, sizeof(addr)); // connect to server

// all right!  now that we're connected, we can recieve some data!
byte_count = recv(s1, buf, sizeof(buf), 0);
printf("recv()'d %d bytes of data in buf\n", byte_count);

// now demo for UDP datagram sockets:
s2 = socket(PF_INET, SOCK_DGRAM, 0);

fromlen = sizeof(addr);
byte_count = recvfrom(s2, buf, sizeof(buf), 0, &addr, &fromlen);
printf("recv()'d %d bytes of data in buf\n", byte_count);
printf("from IP address %s\n", inet_ntoa(addr.sin_addr));
```

## 短连接&长连接

短连接：连接 -> 传输数据 -> 关闭连接；

长连接：连接 -> 传输数据 -> 保持连接 -> 传输数据 ... -> 关闭连接；

## 同步&异步

同步：客户端提交请求 -> 等待服务器处理 -> 服务器处理完毕返回，这个期间客户端不能做别的事；

异步：客户端提交请求后，不必等到服务器回应就可以继续做别的事；

单工（Simplex）：通信双方发送方和接收方分工明确，只能由发送方向接收方单一方向传送数据；即单向通信，角色不能互换；单行道；

半双工（Half Duplex）：通信双方即是发送方又是接收方，可以互相传送数据，但某一时刻只能单方向传送数据；如：踢足球时只能一个人传给另一个人，因为球（信道）只有一个；双向单车道；

全双工（Full Duplex）：通信双方即是发送方又是接收方，可以同时互相传送数据；双向多车道；

# I/O Models

## 一次`I/O`操作的流程

以读写文件为例：

<img src="https://bigbyto.gitee.io//assets/images/io-model/ioflow.png" alt="pic" style="zoom: 67%;" />

上述过程大致是：

1. 程序进行读操作，向`kernel`发起`system call read()`; 
2. 操作系统发生上下文切换（context switch），从用户空间（user space）切换到内核空间（kernel space），把数据读取到`kernel space buffer`中；
3. `kernel`把数据从`kernel space`**复制**到`user space`，同时从内核空间切换回用户空间；
4. 程序进行写操作，发起`system call write()`，发生上下文切换，由`kernel`把数据复制到`kernel space buffer`中，`kernel`再把数据写入磁盘文件中。

可见，一次I/O操作包括两部分：

1. 数据在`Hardware`和`Kernel space buffer`（内核缓冲区）之间流转；
2. 数据在`user space`和`kernel space`间进行**复制**；

## 5种I/O模型

在Linux中共有5种I/O模型：

1. `Blocking I/O`; 	(阻塞 I/O)
2. `Non-Blocking I/O`;   (非阻塞 I/O)
3. `I/O multiplexing (select, poll, epoll)`;   (I/O 多路复用，又分为`select, poll, epoll`3种方式)
4. `Signal Driven I/O (SIGIO)`;  (信号驱动 I/O)
5. `Asynchronous I/O (the POSIX aio_ function)`;  (异步 I/O)

在介绍不同`I/O`模型前，结合上文可见，(以网络上通过socket获取数据为例)，`application`从`socket`中获取到数据可分为2步：

- 第一步：**Load data into kernel.** （即：把数据从**硬件**（CPU, Hard Disk, Network Adapter等）load into `Kernel`；若数据未准备好，就一直等待数据就绪）
- 第二步：**Copy data from kernel to user**.

下文所说“阻塞”，“非阻塞”等都涉及到这两步。

### Blocking I/O

![~replace~/assets/images/io-model/bio.png](https://wiyi.org/assets/images/io-model/bio.png)

`Blocking I/O`发起`system call recvfrom()`时，进程将一直阻塞直到：1.另一端Socket的数据到来；2. 数据从`kernel`复制到`user`。在这种I/O模型下，我们不得不为每一个Socket都分配一个线程，这会造成很大的资源浪费。

此处的`Blocking`是指：两个步骤都阻塞；

优点：简单易用，对于本地I/O而言性能很高；

缺点：处理网络I/O时，造成进程阻塞空等，浪费资源。

### Non-Blocking I/O

`Non-Blocking I/O`每隔一段时间就发起`system call`看数据是否`ready`. 若数据就绪（第一步完成），则进行第二步的`copy`操作；若数据未就绪，`kernel`会立刻返回`EWOULDBLOCK`错误。

![~replace~/assets/images/io-model/nio.png](https://bigbyto.gitee.io//assets/images/io-model/nio.png)

The system keeps passing **recvfrom()** Poll until the first step is completed, and then block the data from kernel state to user state in the second step.
The `Non-Blocking I/O` mode here mainly **refers to the first step**, loading data to the kernel state. This process is non blocking, and polling is used to determine whether the data is ready in the kernel.

疑问：同样是`recvfrom()`，为何在`Blocking I/O`第一步中会阻塞，在`Non-Blocking I/O`第一步中就不会阻塞？

原因：查看`recvfrom()`的定义：

```c++
ssize_t recvfrom(int s, void *buf, size_t len, int flags,
                 struct sockaddr *from, socklen_t *fromlen);
```

其中，`flags`参数可设置是否阻塞，默认阻塞；详见[recvfrom()](https://linux.die.net/man/2/recvfrom); 

### I/O Multiplexing

疑问：为何需要 I/O 多路复用？

![应用场景](https://pic4.zhimg.com/80/v2-529734ac694c4da96ac78eeebd7deb6b_720w.jpg)

场景思考：应用B从TCP缓冲区读取数据。在并发环境下，可能会有`n`个人向应用B发送数据，此时应用B需要创建多个线程去读取数据，每个线程都会调用`recv()`去读取数据。

并发情况下，B所在服务器很可能会瞬间收到上百万个请求消息，此时应用B就需要创建上百万个线程去读取数据；同时应用B不知道何时会有数据就绪，这些线程都会不断向内核发送`recv()`请求来读取数据。

那么问题来了：如此多的线程不停调用`recv()`请求数据，一来服务器可能扛不住，二来过于浪费线程资源。

对应解决思路：能否由一个线程监控多个网络请求（即`fd`,一个网络socket对应一个fd），这样就只需一个或少数线程就可以完成数据状态询问的操作，当有数据准备就绪后再分配对应线程去读取数据；这样可以节省大量线程资源，这就是I/O多路复用的思路。

![img](https://pic3.zhimg.com/80/v2-2c65fd3534e58d3a54cdeae778a31446_720w.jpg)

**I/O多路复用思路**：

系统提供一种函数，可以同时监控多个`fd`操作，这种函数就是常说的`select, poll, epoll`函数。一旦这种函数所监控的`fd`中有任何一个`fd`的数据准备就绪了，这种函数就会返回`readable`状态，应用程序再调用`recv()`读取数据。

IO多路复用的基本思路是**事件驱动**。服务器可以保持多个客户端IO连接，当某个IO上有可读或可写事件发生时，表示该IO对应的客户端在请求服务器的某项服务，此时服务器响应该服务。[[高并发还得用epoll](https://github.com/yuesong-feng/30dayMakeCppServer/blob/main/day03-%E9%AB%98%E5%B9%B6%E5%8F%91%E8%BF%98%E5%BE%97%E7%94%A8epoll.md)]

> 注：IO多路复用和多线程相似，但是不同的概念。前者针对IO接口，后者针对CPU。

> 小结：通过`select, poll或epoll` 来监控多个`fd`, 而不用为每个`fd`创建一个对应的监控线程，从而节省线程资源。

![Five IO models of UNIX](https://imgs.developpaper.com/imgs/1219482591-5ebe4c2caa4c8_articlex.jpg)
The system first checks whether the kernel data is ready through `select`.
When the kernel data is loaded, the system calls **recvfrom** to load kernel state data into user state.
It looks like the first and second steps are blocking operations, but `select` can handle multiple file handles (including sockets) at the same time at a very low cost.

在Linux中三种可以让内核监控`fd`的`system call`分别是：`select, poll`和`epoll`;

注意：

- 不论是`select, poll`还是`epoll`，都地导致进程**阻塞**；发起真正的I/O操作（如`recv()`）时，进程也会阻塞。

- `/O Multiplexing`的**优点**在于：一次性可以监听大量的`fd`; 
- `select, poll, epoll`并不是I/O操作，`read, recvfrom`这些才是。

#### select

```c++
#include <sys/select.h>
#include <sys/time.h>

int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

typedef struct fd_set {
        u_int   fd_count;               /* how many are SET? */
        SOCKET  fd_array[FD_SETSIZE];   /* an array of SOCKETs */
} fd_set;
```

[select()](https://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch06lev1sec3.html) 可以传入3个`fd_set`参数，分别对应不同事件的`fd`。

`int nfds`（或者`int maxfdp1`）：指定特监控的文件描述符(fd)的个数，它的值是待监控的最大描述符加1；

`fd_set`：即`fd`的一个集合；可以指定要让内核监控哪些`fd`集合，可以监控“读”，“写”和“异常”的`fd`集合；如果对某一个条件不感兴趣，可以把它设为空指针；该结构实际是个**数组**，每个数组元素跟一个打开的`fd`相联系；当调用`select()`时，内核根据I/O状态修改`fd_set`中的内容，由此通知哪个`fd`可操作。

`timeout`：指定内核监控指定的`fd`集合的超时时间; 

返回值：返回一个`int`值，代表了就绪的`fd`数量，这个数量是3个`fd_set`中就绪的`fd`数量之和。若超时，返回`0`; 若出错，返回`-1`。

程序发起`system call select()`时的流程：

1. 程序阻塞等待`kernel`返回；（需要把待监控的`fd_set`集合从`User space`复制到`Kernel space`）
2. `kernel`发现有`fd`就绪，返回就绪的`fd`的数量；
3. 程序轮询3个`fd_set`寻找就绪的`fd`; 
4. 向就绪的`fd`发起真正的I/O操作（`read(), recvfrom()`等）；

优点：

1. 几乎所有系统都支持`select`; 
2. 可以一次在`kernel`上监控多个`fd`; 

缺点：

1. 内核对被监控的的`fd_set`集合大小做了限制（`FD_SETSIZE`，默认是1024）；
2. 每次调用`select()`时，需要把待监控的`fd_set`集合从`User space`复制到`Kernel space`，当`fd_set`集合很大时，开销也大；
3. `kernel`返回后，需要**轮询**所有`fd`找出就绪的`fd`，随着`fd`数量的增加，性能会逐渐下降（因为`fd_set`本质是数组，轮询时相当于对数组的遍历）；

#### poll

[poll()](https://man7.org/linux/man-pages/man2/poll.2.html) 仅Linux支持；

```c++
int poll(struct pollfd *fds, nfds_t nfds, int timeout);

//pollfd结构体如下
struct pollfd {
  int   fd;         /* file descriptor */	// 需要被监控的fd
  short events;     /* requested events */	// 对fd上感兴趣的事件
  short revents;    /* returned events */	// fd上当前实际发生的事件
};
```

`poll()`相比`select()`只是没有了最大`fd`的限制，其他方面基本类似。（`select()`里是数组，`poll()`是**链表**）

`pollfd *fds`: 存放待监控的`fd`；`events`是被监控`fd`的事件掩码，由用户来设置；`revents`是`fd`的操作结果事件掩码，内核在调用返回时设置；

`nfds_t nfds`: 记录`fds`中待监控的`fd`的总数量；

返回值：返回`fds`集合中就绪的读、写和异常的`fd`总数量；返回`0`表示超时，返回`-1`表示出错；

> Sean 注：
>
> 类比“投资赢家”中的结构体，它里面有柔性数组，前面会有变量表示实际数据的长度；

优点：相比`select()`没有了最大`fd`的限制；

缺点：

1. 每次调用`poll()`时，需要把`nfds`个待监控的`fd`(即`fds`)从`User space`复制到`Kernel space`，当`fd`数量很大时，开销也大；
2. `kernel`返回后，需要**轮询**所有`fd`找出就绪的`fd`，随着`fd`数量的增加，性能会逐渐下降（因为`fds`是链表，轮询时相当于对链表的遍历）；

注：对于数据已就绪的`fd`，如果没有被处理，下次`poll()`时还会继续报告该`fd`; 这一点跟`epoll()`不一样。

示例程序1：(参考：[CSocket](https://github.com/MrCoderStack/CSocket);)

server_poll.cpp

```c++
#include <arpa/inet.h>			// inet_ntoa()
#include <cstdio>				// printf, perror()
#include <cstring>				// memeset()
#include <iostream>
#include <sys/socket.h>			// socket()
#include <sys/types.h>
#include <sys/poll.h>           // poll()
#include <unistd.h>				// read(), write(), close()

const int kBacklog = 100;       // 最大积压数

int main()
{
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    int connection_fd[kBacklog] = { 0 };

    sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    inet_aton("0.0.0.0", &server_addr.sin_addr);
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(8888);

    int option = 1;
    setsockopt(listen_fd, SOL_SOCKET, SO_REUSEADDR, &option, sizeof(option));

    bind(listen_fd, (sockaddr *)&server_addr, sizeof(server_addr));
    listen(listen_fd, 100);

    int flag = -1;
    pollfd fds[kBacklog + 1];       // 待监听的fd: kBacklog 个通信socket 和 1个监听socket
    memset(fds, flag, sizeof(fds));
    fds[0].fd = listen_fd;
    fds[0].events = POLLIN;

    int nfds = 1;                   // fds 中待监听的fd总数
    int timeout = -1;               // 永不超时
    
    for (int i = 1; i <= kBacklog; ++i)
    {
        int result = poll(fds, nfds, timeout);
        if (result < 0)
        {
            printf("poll error, result %d\n", result);
            continue;
        }

        if (fds[0].revents & POLLIN)        // listen_fd 检测到有新客户端连接过来
        {
            connection_fd[i - 1] = accept(listen_fd, nullptr, nullptr);
            fds[i].fd = connection_fd[i - 1];
            fds[i].events = POLLIN;
            nfds++;
            printf("New client came, local fd is [%d]\n", connection_fd[i - 1]);
        }

        for (int j = 0; j < kBacklog; ++j)
        {
            char buffer[1024] = { 0 };
            if (connection_fd[j] > 0 && (fds[j + 1].revents & POLLIN))  // 检测已连接的socket的读事件
            {
                int recv_len = recv(connection_fd[j], buffer, sizeof(buffer) - 1, 0);
                if (recv_len > 0)
                {
                    printf("recv data [%s] from local fd [%d] \n", buffer, connection_fd[j]);
                }
                else if (recv_len == 0)
                {
                    close(connection_fd[j]);
                    memset(&fds[j + 1], flag, sizeof(pollfd));
                    printf("Connection closed fd [%d]\n", connection_fd[j]);
                    connection_fd[j] = 0;
                }
                else
                {
                    
                    close(connection_fd[j]);
                    memset(&fds[j + 1], flag, sizeof(pollfd));
                    printf("recv error; recv_len is %d; %s; errno:%d\n", recv_len, strerror(errno), errno);
                    connection_fd[j] = 0;
                }
            }
        } // for j
    } // for i

    close(listen_fd);
    return 0;
}
```

client_poll.cpp

```c++
// client
#include <arpa/inet.h>			// inet_ntoa()
#include <cstdio>
#include <cstdlib>				// atoi()
#include <cstring>				// memeset()
#include <netinet/in.h>			// htons()
#include <netdb.h>				// gethostbyname()
#include <sys/socket.h>			// socket()
#include <sys/types.h>
#include <unistd.h>				// read(), write(), close()

int main()
{
    int client_fd = socket(AF_INET, SOCK_STREAM, 0);

    sockaddr_in server_addr;
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(8888);

    connect(client_fd, (const sockaddr *)&server_addr, sizeof(sockaddr_in));
    char send_buffer[100] = "This is demo.";
    int index = 1;
    while (true)
    {
        send(client_fd, send_buffer, strlen(send_buffer) + 1, 0);
        scanf("%s", send_buffer);
    }

    close(client_fd);
    return 0;
}
```

输出：

```c++
// client 端
sean@sean-virtual-machine:~/xxxx/NetDemo$ ./client_poll 
hi
Let's do it
close
    
// server 端
sean@sean-virtual-machine:~/xxxx/NetDemo$ ./server_poll 
New client came, local fd is [4]
recv data [This is demo.] from local fd [4] 
recv data [hi] from local fd [4] 
recv data [Let's] from local fd [4] 
recv data [do] from local fd [4] 
recv data [it] from local fd [4] 
recv data [close] from local fd [4]
```

另一个例子可参考：[poll](https://github.com/yuanrw/tcp-server-client/tree/master/poll); 

#### epoll

仅Linux支持；

可以发现，`select()`和`poll()`有两个共同的缺点：1. 大量的`fd`在`User space`和`Kernel space`之间进行复制，开销较大；2. `kernel`返回后，需要轮询找出就绪的`fd`, 效率不高。

可不可以当某个`fd`就绪后，`kernel`直接返回就绪的`fd`，而无须轮询呢？这就是`epoll()`. 此外，`epoll()`通过使用`mmap()`减少了`User space`和`Kernel space`之间的复制开销。

`epoll()`是一种事件通知机制（基于事件驱动的I/O方式），内部结构由红黑树实现，用于监听程序注册的`fd`。通过`epoll_ctl()`注册`fd`，一旦该`fd`就绪，内核就会采用类似`callback`的回调机制来激活该`fd`,`epoll_wait()`便可以收到通知。相关函数有：

- `epoll_create1()`: 创建一个`epoll`实例并返回它的`fd`; 

  ```c++
  #include <sys/epoll.h>
  int epoll_create1(int flags);
  
  // int epfd = epoll_create(1024); // 参数表示监听事件的大小，如超过内核会自动调整；已经被舍弃;无实际意义,传入一个大于0的数即可
  int epfd = epoll_create1(0); // 参数flags一般设为0，详细参考 man epoll
  ```

- `epoll_ctl()`: 操作`epoll`实例监听的`fd`，可往里面新增、删除`fd`; （即注册`fd`）

  ```c++
  int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
  
  typedef union epoll_data {
    void        *ptr;
    int          fd;
    uint32_t     u32;
    uint64_t     u64;
  } epoll_data_t;
  
  struct epoll_event {
    uint32_t     events;      /* Epoll events; 如:EPOLLIN */
    epoll_data_t data;        /* User data variable */
  };
  ```

  - `epfd`: 即`epoll_create1()`返回的`fd`; 
  - `op`: `operation`的缩写，可以是`EPOLL_CTL_ADD, EPOLL_CTL_MOD, EPOLL_CTL_DEL`; 
  - `fd`: 即要监听的`fd`; 
  - `event`: 即要注册的事件集合（可读，可写等; ）；

  `epoll`监听事件的`fd`会放在一颗红黑树上，将要监听的IO接口放入`epoll`红黑树中，就可以监听该IO上的事件。

  ```c++
  epoll_ctl(epfd, EPOLL_CTL_ADD, sockfd, &ev);		// 添加事件到epoll
  epoll_ctl(epfd, EPOLL_CTL_MOD, sockfd, &ev);		// 修改epoll红黑树上的事件
  epoll_ctl(epfd, EPOLL_CTL_DEL, sockfd, nullptr);	// 删除事件
  ```

  

- `epoll_wait()`: 用来获取有事件发生的`fd`；等待`epoll`实例中有`fd`就绪，且返回就绪的`fd`的数量.

  ```c++
  int epoll_wait(int epfd, struct epoll_event *events,int maxevents, int timeout);
  ```

  调用`epoll_wait()`会阻塞，直到`epoll`实例监听的`fd`中有一个或多个`fd`就绪或超时；就绪的`fd`就会被`kernel`添加到`events`中；

  - `epfd`: 即`epoll_create1()`返回的`fd`; 
  - `events`: 就绪的`fd`会放在这里；
  - `maxevents`: 可供返回的最大的事件数量，一般是`events`的最大数量；
  - `timeout`: 超时时间，设置为-1表示一直等特；

**水平触发&边缘触发**：

`epoll`有`LT`和`ET`两种模式；

- 水平触发（LevelTriggered, **LT**): **默认**工作模式；`socket`缓冲区`有数据可读`/`没满可写`时，`可读`/`可写`事件一直触发，`epoll_wait()`会一直触发返回（直到缓冲区数据被读取完毕/写满，或者`socket`不再处于`readable/writeable`状态）。（即当`epoll_wait()`监听到某`fd`事件就绪并通知应用程序时，应用程序可以不立即处理该事件；下次调用`epoll_wait()`时，会再次通知此事件。）

  > `select()`和`poll()`属于水平触发；
  >
  > 对于`listenfd`默认使用LT模式；

- 边缘触发（Edge Triggered, **ET**): 只在状态改变（**状态从未就绪变为就绪**）时才触发事件（可理解为，只有新的数据到来时才会触发）；即当`epoll_wait()`监听到某`fd`事件就绪并通知应用程序时，应用程序必须立即处理该事件；如果不处理，下次调用`epoll_wait()`时，也不会再通知此事件。故：该模式下，有数据时必须循环读取完全部数据，直到`read`返回值小于请求值，或遇到`EAGAIN`错误。

  优缺点：高效，提高并发度；程序逻辑必须一次性处理好该`fd`上的事件；
  
  ET模式必须搭配**非阻塞**式`socket`使用。
  
  > 例子：当发送方速率很快，一次可能发送`n`个`msg1`消息包，但接收方只会触发一次`EPOLLIN`(The associated file is available for read operations.) ; 这时，接收方如果不把这`n`个`msg1`消息都处理完毕，那么当发送方再发送另一个`msg2`消息时，接收方触发后处理的还是`msg1`，而`msg2`的消息就无法及时处理而造成**消息处理延迟**；因此，**ET**模式要求，每次触发时都要**全部读取**。

> LT和ET原本用于脉冲信号。Level和Edge指触发点；Level指：只要处于水平，就**一直触发**；Edge指：只有是上升沿和下降沿的时候才触发；比如：0 -> 1就是Edge, 1->1就是Level。（仅用于帮助理解）

ET模式可以大大减少了`epoll`事件的触发次数，效率较高。

`epoll`示例程序：参考下文的“[epoll版 Echo Server](#epoll 版 Echo Server)"; 或参考[epoll](https://github.com/yuanrw/tcp-server-client/tree/master/epoll); 

#### 小结

|            | select                                             | poll                                               | epoll                    |
| ---------- | -------------------------------------------------- | -------------------------------------------------- | ------------------------ |
| 操作方式   | 遍历                                               | 遍历                                               | 遍历                     |
| 底层实现   | 数组                                               | 链表                                               | 红黑树                   |
| I/O效率    | 每次调用轮询（线性遍历）O(N)                       | 每次调用轮询（线性遍历） O(N)                      | 事件驱动 O(1)            |
| 最大连接数 | 1024(x86) or 2048(x64)                             | 无上限                                             | 无上限                   |
| `fd`复制   | 需把待监控的`fd`从`User space`复制到`Kernel space` | 需把待监控的`fd`从`User space`复制到`Kernel space` | 使用`mmap`减少了复制开销 |

### Signal Driven I/O

![Five IO models of UNIX](https://imgs.developpaper.com/imgs/1352280258-5ebe4c366c67c_articlex.jpg)
The first step is to register a callback function and notify me when the kernel data is ready.
At this time, the system can do other things **without blocking waiting** for kernel data.
Step 2, blocking call **recvfrom**, load the kernel’s data into the user state.

这种模式不常用。

### Asynchronous I/O

```c++
int aio_read(struct aiocb *aiocbp);

//aiocb
struct aiocb {
  /* The order of these fields is implementation-dependent */

  int             aio_fildes;     /* File descriptor */
  off_t           aio_offset;     /* File offset */
  volatile void  *aio_buf;        /* Location of buffer */
  size_t          aio_nbytes;     /* Length of transfer */
  int             aio_reqprio;    /* Request priority */
  struct sigevent aio_sigevent;   /* Notification method */
  int             aio_lio_opcode; /* Operation to be performed;
                                       lio_listio() only */
  /* Various implementation-internal fields not shown */
};
```

发起`system call aio_read`后，`kernel`会立即返回；等数据就绪，`kernel`会发送一个`signal`，以便让`application`处理数据；应用程序只发起了一次`system call`，且整个过程没有任何阻塞。

![Five IO models of UNIX](https://imgs.developpaper.com/imgs/644623516-5ebe4c3f78028_articlex.jpg)
In theory, this model is a real asynchronous model, because in the above four models, in the second step: data loading from kernel state to user state is synchronous operation.
In this model, when the system loads the file, it only needs to pass `AIO_Read` registers a callback to notify the current system when the file is loaded in kernel state or user state.
In this process, the system can perform other operation tasks without waiting.

## Summary and comparison

![Five IO models of UNIX](https://imgs.developpaper.com/imgs/1334357830-5ebe4c5d46748_articlex.jpg)
This figure is a summary and comparison of the above five IO models. Generally speaking, the later the model is, the more efficient in theory.
The first four IO models [blocking IO, non blocking IO, IO multiplexing, signal driven IO] are all synchronous IO, only the last one is truly asynchronous [asynchronous IO].

**同步I/O & 异步I/O**：

POSIX对这两个术语定义如下:

- 同步I/O操作将会造成请求进程阻塞，直到I/O操作完成; 
- 异步I/O操作不会造成进程阻塞; 

根据上面的定义我们可以看出，前面4种I/O模型都是同步I/O，因为它们的I/O操作(`recvfrom`)都会造成进程阻塞。只有最后一个I/O模型匹配异步I/O的定义。

> POSIX defines these two terms as follows:
>
> - A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes.
> - An asynchronous I/O operation does not cause the requesting process to be blocked.
>
> Using these definitions, the first four I/O models—blocking, nonblocking, I/O multiplexing, and signal-driven I/O—are all synchronous because the actual I/O operation (recvfrom) blocks the process. Only the asynchronous I/O model matches the asynchronous I/O definition.

参考资料：

- [100%弄明白5种I/O模型](https://zhuanlan.zhihu.com/p/115912936); 
- [一篇文章读懂阻塞，非阻塞，同步，异步](https://www.jianshu.com/p/b8203d46895c); 
- [IO多路复用的三种机制Select，Poll，Epoll](https://www.jianshu.com/p/397449cadc9a); 
- [Blocking vs. non-blocking sockets](https://www.scottklement.com/rpg/socktut/nonblocking.html); 
- [带你彻底理解Linux五种I/O模型](https://wiyi.org/linux-io-model.html); 
- [理解Linux中的file descriptor(文件描述符)](https://wiyi.org/linux-file-descriptor.html); 
- [Five IO models of Unix](https://developpaper.com/five-io-models-of-unix/);
- [EPOLLET VS EPOLLLT](https://zhuanlan.zhihu.com/p/36977578); 

# Programming Practice

在编程实践中学习并总结相关知识点；

参考资料：

- [Github - 30天自制C++服务器](https://github.com/yuesong-feng/30dayMakeCppServer); 

## 异常处理

封装一个错误处理函数：

```c++
#include <cstdio>		// perror()
#include <cstdlib>		// exit()

// 如果 condition 为真,则打印错误信息
void ErrorIf(bool condition, const char *error_msg)
{
    if (condition)
    {
        perror(error_msg);
        exit(EXIT_FAILURE);
    }
}

// 使用示例
int listening_sockfd = socket(AF_INET, SOCK_STREAM, 0);
ErrorIf(listening_sockfd == -1, "socket create error");
```

## Echo Server

编写一个简单的回声服务器，即服务端将客户端发来的数据原封不动地发送回去。

util.h, util.cpp:

```c++
// 名称：util.h
// 版权：仅供学习
// 作者：Sean (eppesh@163.com)
// 环境：VS2019
// 时间：04/26/2022
// 说明：工具类,包含常用辅助函数;

#ifndef UTIL_H_
#define UTIL_H_

#include <cstdio>
#include <cstdlib>

// 条件condition为true时输出错误信息error_msg
void ErrorIf(bool condition, const char *error_msg);

#endif

///////////////////////////////
// util.cpp
#include "util.h"

void ErrorIf(bool condition, const char *error_msg)
{
    if (condition)
    {
        perror(error_msg);
        exit(EXIT_FAILURE);
    }
}
```

Makefile文件：

```shell
.PHONY : clean 
server : 
	g++ util.cpp client.cpp -o client && \
	g++ util.cpp server.cpp -o server

clean :
	rm server && rm client
```

server.cpp:

```c++
// 名称：server.cpp
// 版权：仅供学习
// 作者：Sean (eppesh@163.com)
// 环境：VS2019
// 时间：04/26/2022
// 说明：服务端

#include <arpa/inet.h>
#include <cstring>
#include <netinet/in.h>
#include <sys/socket.h>
#include <unistd.h>

#include "util.h"

int main()
{
    int listening_sockfd = socket(AF_INET, SOCK_STREAM, 0);
    ErrorIf((listening_sockfd == -1), "socket create error");

    sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    server_addr.sin_port = htons(8888);

    ErrorIf((bind(listening_sockfd, (sockaddr *)&server_addr, sizeof(server_addr)) == -1), "socket bind error");

    ErrorIf((listen(listening_sockfd, SOMAXCONN) == -1), "socket listen error");

    sockaddr_in client_addr;
    memset(&client_addr, 0, sizeof(client_addr));
    socklen_t client_addr_len = sizeof(client_addr);
    
    int client_sockfd = accept(listening_sockfd, (sockaddr *)&client_addr, &client_addr_len);
    ErrorIf((client_sockfd == -1), "socket accept error");

    printf("new client fd[%d]! IP: %s Port: %d\n", client_sockfd, inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

    while (true)
    {
        char buf[1024] = { 0 };
        ssize_t read_bytes = read(client_sockfd, buf, sizeof(buf));
        if (read_bytes > 0)
        {
            printf("message from client fd[%d]: %s\n", client_sockfd, buf);
            write(client_sockfd, buf, sizeof(buf));
        }
        else if (read_bytes == 0)
        {
            printf("client fd[%d] disconnected\n", client_sockfd);
            close(client_sockfd);
            break;
        }
        else if (read_bytes == -1)
        {
            close(client_sockfd);
            ErrorIf(true, "socket read error");
        }
    }

    close(listening_sockfd);
    return 0;
}
```

client.cpp:

```c++
// 名称：client.cpp
// 版权：仅供学习
// 作者：Sean (eppesh@163.com)
// 环境：VS2019
// 时间：04/26/2022
// 说明：客户端

#include <arpa/inet.h>
#include <cstring>
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <unistd.h>

#include "util.h"

int main()
{
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    ErrorIf((sockfd == -1), "socket create error");

    sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    server_addr.sin_port = htons(8888);

    ErrorIf((connect(sockfd, (sockaddr *)&server_addr, sizeof(server_addr)) == -1), "socket connect error");

    while (true)
    {
        char buf[1024] = { 0 };
        scanf("%s", buf);
        ssize_t write_bytes = write(sockfd, buf, sizeof(buf));
        if (write_bytes == -1)
        {
            printf("socket already disconnected, can't write any more!\n");
            break;
        }

        memset(buf, 0, sizeof(buf));
        ssize_t read_bytes = read(sockfd, buf, sizeof(buf));
        if (read_bytes > 0)
        {
            printf("message from server: %s\n", buf);
        }
        else if (read_bytes == 0)
        {
            printf("server socket disconnected!\n");
            break;
        }
        else if (read_bytes == -1)
        {
            close(sockfd);
            ErrorIf(true, "socket read error");
        }
    }

    close(sockfd);
    return 0;
}
```

## epoll 版 Echo Server

使服务器可以接收**多个客户端**连接。

改进思路：

- 创建完服务器的监听`fd`-`listening_sockfd`后，将之添加到`epoll`中，只要这个`fd`上发生可读事件，表示有一个新的客户端连接。
- 每次`accept`一个客户端就把该客户端的`fd` - `client_sockfd`添加到`epoll`，`epoll`会监听`client_sockfd`是否有事件发生，如果发生则处理事件。

`fcntl`是Linux上的系统调用，用来对已打开的文件描述符`fd`进行各种控制操作，用以改变已打开文件的各种属性（如复制文件描述符，设置非阻塞，加读写锁等）。经常用来**设置非阻塞、加读写锁**。

```c++
#include <fcntl.h>
int fcntl(int fd, int cmd, ... /* arg */);
```

`cmd`命令：

- `F_DUPFD`: 复制文件描述符；与`dup`函数功能一样，复制由`fd`指向的文件描述符，调用成功后返回新的文件描述符，与旧的`fd`共同指向同一个文件。
- `F_GETFD`: 获取`fd`的标志（如`close-on-exec`标志）；[[CSDN-fd标志和fd状态标志](https://blog.csdn.net/kyang_823/article/details/79496362)]
- `F_SETFD`: 设置`fd`的标志（如将文件描述符`close-on-exec`标志设置为第三个参数`arg`的最后一位）；
- `F-GETFL`: 获取`fd`的状态标志；
- `F_SETFL`: 设置`fd`的状态标志；
- `F_GETLK`: 获取文件锁；
- `F_SETLK`: 设置文件锁；

**设置非阻塞**：先把`fd`对应的标志读出来，再加上非阻塞标志`O_NONBLOCK`，再设置回去。

添加标志用**或运算(`|`)**，清除标志用**按位取反后的与运算**；

```c++
int flags = fcntl(fd, F_GETFL);
// 设置非阻塞
flags |= O_NONBLOCK;
fcntl(fd, F_SETFL, flags);
printf("flags: %#x \n", flags);
// 清除非阻塞标志
flags &= ~O_NONBLOCK;
fcntl(fd, F_SETFL, flags);
printf("flags: %#x \n", flags);
```

server.cpp:

```c++
// 名称：server.cpp
// 版权：仅供学习
// 作者：Sean (eppesh@163.com)
// 环境：VS2019
// 时间：04/26/2022
// 说明：服务端

#include <arpa/inet.h>
#include <cstring>
#include <cerrno>
#include <fcntl.h>				// fcntl
#include <netinet/in.h>
#include <sys/epoll.h>          // epoll
#include <sys/socket.h>
#include <unistd.h>

#include "util.h"

const int kMaxEvents = 1024;
const int kReadBuffer = 1024;

void SetNonBlocking(int fd)
{
    fcntl(fd, F_SETFL, fcntl(fd, F_GETFL) | O_NONBLOCK);
}

int main()
{
    int listening_sockfd = socket(AF_INET, SOCK_STREAM, 0);
    ErrorIf((listening_sockfd == -1), "socket create error");

    sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    server_addr.sin_port = htons(8888);

    ErrorIf((bind(listening_sockfd, (sockaddr *)&server_addr, sizeof(server_addr)) == -1), "socket bind error");
    ErrorIf((listen(listening_sockfd, SOMAXCONN) == -1), "socket listen error");

    int epfd = epoll_create1(0);
    ErrorIf((epfd == -1), "epoll create error");

    epoll_event events[kMaxEvents];
    epoll_event ev;
    memset(&events, 0, sizeof(events));
    memset(&ev, 0, sizeof(ev));
    ev.data.fd = listening_sockfd;
    ev.events = EPOLLIN | EPOLLET;      // 此处使用了ET模式，且未处理错误；后续会修复，因为接受连接最好不用ET模式
    SetNonBlocking(listening_sockfd);
    epoll_ctl(epfd, EPOLL_CTL_ADD, listening_sockfd, &ev);

    while (true)    // 不断监听epoll上的事件并处理
    {
        int nfds = epoll_wait(epfd, events, kMaxEvents, -1);        // 有nfds个fd发生事件
        ErrorIf((nfds == -1), "epoll wait error");

        for (int i = 0; i < nfds; ++i)  // 处理这nfds个事件
        {
            if (events[i].data.fd == listening_sockfd)  // 发生事件的fd是服务器的监听fd(listening_sockfd)，表示有新客户端连接
            {
                sockaddr_in client_addr;
                memset(&client_addr, 0, sizeof(client_addr));
                socklen_t client_addr_len = sizeof(client_addr);

                int client_sockfd = accept(listening_sockfd, (sockaddr *)&client_addr, &client_addr_len);
                ErrorIf((client_sockfd == -1), "socket accept error");
                printf("new client fd[%d]! IP: %s Port: %d\n", client_sockfd, inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

                memset(&ev, 0, sizeof(ev));
                ev.data.fd = client_sockfd;
                ev.events = EPOLLIN | EPOLLET;                              // 对于客户端连接，使用ET模式，可以让epoll更高效，支持更多并发
                SetNonBlocking(client_sockfd);                              // ET模式要搭配非阻塞socket使用
                epoll_ctl(epfd, EPOLL_CTL_ADD, client_sockfd, &ev);         // 将该客户端fd添加到epoll
            }
            else if (events[i].events & EPOLLIN)    // 发生事件的是客户端，并且是可读事件
            {
                char buf[kReadBuffer] = { 0 };
                while (true)    // 由于使用非阻塞IO，读取客户端buffer时，需要不断读取，一次读取buf大小数据，直到全部读取完毕
                {
                    memset(buf, 0, sizeof(buf));
                    ssize_t read_bytes = read(events[i].data.fd, buf, sizeof(buf));
                    if (read_bytes > 0)
                    {
                        printf("message from client fd[%d]: %s\n", events[i].data.fd, buf);
                        write(events[i].data.fd, buf, sizeof(buf));
                    }
                    else if (read_bytes == -1 && errno == EINTR)        // 客户端正常中断，继续读取
                    {
                        printf("continue reading");
                        continue;
                    }
                    else if (read_bytes == -1 && ((errno == EAGAIN) || (errno == EWOULDBLOCK)))         // 非阻塞IO，这个条件表示数据全部读取完毕
                    {
                        printf("finish reading once, errno: %d\n", errno);
                        break;
                    }
                    else if (read_bytes == 0)                           // EOF, 客户端断开连接
                    {
                        printf("EOF, client fd[%d] disconnected\n", events[i].data.fd);
                        close(events[i].data.fd);       // 关闭socket会自动将文件描述符从epoll树上移除
                        break;
                    }
                    else
                    {
                        // 其他 read_bytes == -1 的情况处理
                    }
                }// while
            }
            else    // 其他事件
            {
                printf("something else happened\n");
            }
        } // end of for
    } // end of while

    close(listening_sockfd);
    return 0;
}
```

client.cpp: 基本没变化，只是新增了一个全局变量替换之前的1024.

```c++
const int kBufferSize = 1024;
char buf[kBufferSize] = { 0 };
```

client端需要注意：发送的`buf`大小必须**大于等于**服务器端`buf`大小，不然会出错（因为服务器端使用`epoll`的ET模式与客户端连接，从客户端读取数据时需要一次性读取完;如果当前客户端的`buf`小于服务器端的`buf`大小，服务器读取时就会越界）。

## 封装epoll版Echo Server

将上面`epoll`版的`Echo Server`封装成相应类。

- 更新`Makefile`文件：

```shell
.PHONY : clean rebuild
server : 
	g++ -o client util.cpp client.cpp && \
	g++ -o server util.cpp server.cpp epoll.cpp inet_address.cpp socket.cpp
 
clean :
	rm server && rm client

rebuild :
	clean server
```

添加了`rebuild`，这样输入`make rebuild`命令就可以先清除旧文件再重新编译构建。

- 新建`Epoll`类，位于`epoll.h, epoll.cpp`中；

```c++
// epoll.h
#ifndef EPOLL_H_
#define EPOLL_H_
#include <cstring>
#include <sys/epoll.h>
#include <unistd.h>
#include <vector>
#include "util.h"
const int kMaxEvents = 1000;
class Epoll
{
private:
    int epfd_;                      // epoll 实例
    epoll_event *events_;
public:
    Epoll();
    ~Epoll();
    void AddFd(int fd, uint32_t operation);                 // 添加fd
    std::vector<epoll_event> Poll(int timeout = -1);        // 返回就绪的fd
};
#endif

// epoll.cpp
#include "epoll.h"
Epoll::Epoll() :epfd_(-1), events_(nullptr)
{
    epfd_ = epoll_create1(0);
    ErrorIf((epfd_ == -1), "epoll create error");
    events_ = new epoll_event[kMaxEvents];
    ErrorIf((events_ == nullptr), "new epoll_event error");
    memset(events_, 0, sizeof(*events_) * kMaxEvents);
}
Epoll::~Epoll()
{
    if (epfd_ != -1)
    {
        close(epfd_);
        epfd_ = -1;
    }
    if (events_ != nullptr)
    {
        delete[] events_;
        events_ = nullptr;
    }
}
void Epoll::AddFd(int fd, uint32_t operation)
{
    epoll_event ev;
    memset(&ev, 0, sizeof(ev));
    ev.data.fd = fd;
    ev.events = operation;
    ErrorIf((epoll_ctl(epfd_, EPOLL_CTL_ADD, fd, &ev) == -1), "epoll add event error");
}
std::vector<epoll_event> Epoll::Poll(int timeout /* = -1 */)
{
    std::vector<epoll_event> active_events;
    int nfds = epoll_wait(epfd_, events_, kMaxEvents, timeout);
    ErrorIf((nfds == -1), "epoll wait error");

    for (int i = 0; i < nfds; ++i)
    {
        active_events.push_back(events_[i]);
    }
    return active_events;
}
```

- 新建`Socket`类，位于`socket.h, socket.cpp`中；

```c++
// socket.h
#ifndef SOCKET_H_
#define SOCKET_H_

#include <cstdio>
#include <fcntl.h>
#include <sys/socket.h>
#include <unistd.h>

#include "inet_address.h"
#include "util.h"

class InetAddress;

class Socket
{
private:
    int sockfd_;

public:
    Socket();
    Socket(int sockfd);
    ~Socket();

    void Bind(InetAddress *address);                // 绑定IP和Port
    void Listen();                                  // 监听
    void SetNonBlocking();                          // 设置为非阻塞

    int Accept(InetAddress *address);               // 接受连接,返回客户端fd
    int GetFd();                                    // 获取fd
};

#endif

// socket.cpp
#include "socket.h"

Socket::Socket() :sockfd_(-1)
{
    sockfd_ = socket(AF_INET, SOCK_STREAM, 0);
    ErrorIf((sockfd_ == -1), "socket create error");
}

Socket::Socket(int sockfd) :sockfd_(sockfd)
{
    ErrorIf((sockfd_ == -1), "socket create error");
}

Socket::~Socket()
{
    if (sockfd_ != -1)
    {
        close(sockfd_);
        sockfd_ = -1;
    }
}

void Socket::Bind(InetAddress *address)
{
    ErrorIf(bind(sockfd_, (sockaddr *)&address->address_, address->address_len_) == -1, "socket bind error");
}

void Socket::Listen()
{
    ErrorIf(listen(sockfd_, SOMAXCONN) == -1, "socket listen error");
}

void Socket::SetNonBlocking()
{
    fcntl(sockfd_, F_SETFL, fcntl(sockfd_, F_GETFL) | O_NONBLOCK);
}

int Socket::Accept(InetAddress *address)
{
    int client_sockfd = accept(sockfd_, (sockaddr *)&address->address_, &address->address_len_);
    ErrorIf((client_sockfd == -1), "socket accept error");
    return client_sockfd;
}

int Socket::GetFd()
{
    return sockfd_;
}
```

- 新建`InetAddress`类，位于`inet_address.h, inet_address.cpp`中；

```c++
// inet_address.h
#ifndef INET_ADDRESS_H_
#define INET_ADDRESS_H_

#include <arpa/inet.h>
#include <cstring>

class InetAddress
{
public:
    sockaddr_in address_;
    socklen_t address_len_;

public:
    InetAddress();
    InetAddress(const char *ip, uint16_t port);
    ~InetAddress();
};

#endif

// inet_address.cpp
#include "inet_address.h"

InetAddress::InetAddress() :address_len_(sizeof(address_))
{
    memset(&address_, 0, sizeof(address_));
}

InetAddress::InetAddress(const char *ip, uint16_t port) : address_len_(sizeof(address_))
{
    memset(&address_, 0, sizeof(address_));
    address_.sin_family = AF_INET;
    address_.sin_addr.s_addr = inet_addr(ip);
    address_.sin_port = htons(port);
}

InetAddress::~InetAddress()
{

}
```

- client.cpp未改动；server.cpp有变化:

```c++
#include <arpa/inet.h>
#include <cstring>
#include <cerrno>
#include <fcntl.h>
#include <netinet/in.h>
#include <sys/epoll.h>          // epoll
#include <sys/socket.h>
#include <unistd.h>

#include "epoll.h"
#include "inet_address.h"
#include "socket.h"
#include "util.h"

const int kReadBuffer = 1024;

void SetNonBlocking(int fd)
{
    fcntl(fd, F_SETFL, fcntl(fd, F_GETFL) | O_NONBLOCK);
}

void HandleReadEvent(int sockfd);

int main()
{
    Socket *server_socket = new Socket();
    InetAddress *server_address = new InetAddress("127.0.0.1", 8888);
    server_socket->Bind(server_address);
    server_socket->Listen();

    Epoll *ep = new Epoll();
    server_socket->SetNonBlocking();
    ep->AddFd(server_socket->GetFd(), EPOLLIN | EPOLLET);

    while (true)
    {
        std::vector<epoll_event> events = ep->Poll();
        int nfds = events.size();
        for (int i = 0; i < nfds; ++i)
        {
            if (events[i].data.fd == server_socket->GetFd())    // 发生事件的fd是服务器的监听fd(listening_sockfd)，表示有新客户端连接
            {
                InetAddress *client_addr = new InetAddress();                           // TODO: 记得内存释放
                Socket *client_socket = new Socket(server_socket->Accept(client_addr)); // TODO: 记得内存释放
                printf("new client fd[%d]! IP: %s Port: %d\n", client_socket->GetFd(), inet_ntoa(client_addr->address_.sin_addr), ntohs(client_addr->address_.sin_port));
                
                client_socket->SetNonBlocking();
                ep->AddFd(client_socket->GetFd(), EPOLLIN | EPOLLET);
            }
            else if (events[i].events & EPOLLIN)        // 发生事件的是客户端，并且是可读事件
            {
                HandleReadEvent(events[i].data.fd);
            }
            else
            {
                printf("something else happened\n");
            }
        }
    } // end of while

    if (server_socket != nullptr)
    {
        delete server_socket;
        server_socket = nullptr;
    }
    if (server_address != nullptr)
    {
        delete server_address;
        server_address = nullptr;
    }
    if (ep != nullptr)
    {
        delete ep;
        ep = nullptr;
    }
    return 0;
}

void HandleReadEvent(int sockfd)
{
    char buf[kReadBuffer] = { 0 };
    while (true)    // 由于使用非阻塞IO，读取客户端buffer时，需要不断读取，一次读取buf大小数据，直到全部读取完毕
    {
        memset(buf, 0, sizeof(buf));
        ssize_t read_bytes = read(sockfd, buf, sizeof(buf));
        if (read_bytes > 0)
        {
            printf("message from client fd[%d]: %s\n", sockfd, buf);
            write(sockfd, buf, sizeof(buf));
        }
        else if (read_bytes == -1 && errno == EINTR)        // 客户端正常中断，继续读取
        {
            printf("continue reading");
            continue;
        }
        else if (read_bytes == -1 && ((errno == EAGAIN) || (errno == EWOULDBLOCK)))         // 非阻塞IO，这个条件表示数据全部读取完毕
        {
            printf("finish reading once, errno: %d\n", errno);
            break;
        }
        else if (read_bytes == 0)                           // EOF, 客户端断开连接
        {
            printf("EOF, client fd[%d] disconnected\n", sockfd);
            close(sockfd);       // 关闭socket会自动将文件描述符从epoll树上移除
            break;
        }
        else
        {
            // 其他 read_bytes == -1 的情况处理
        }
    }
}
```

## epoll高级用法--Channel

回顾`epoll`的用法：我们把关心的`fd`注册到内核中`epfd`标识的事件表（红黑树）中，当该`fd`上有我们关心的事件发生时，拿到该`fd`并处理就绪的事件。

试想这样一个情景：一台服务器可以同时提供不同类型的服务，如既可以提供HTTP服务，也能提供FTP服务等。当一个`clientfd`上发生了可读事件，我们虽然拿到了该`clientfd`的`fd`，但是后面应该按照”HTTP服务“的应答逻辑去处理该事件呢，还是按照”FTP服务“的应答逻辑去处理该事件？因为不同的连接类型（这条`socket`连接可能是HTTP服务的连接，也可能是FTP服务的连接）常常对应不同的处理逻辑，而我们仅仅依靠`epoll_wait`返回的`clientfd`（一个`int`型的值）是不足以区分该`fd`属于什么连接类型的，因此我们希望从`epoll_wait`的返回结果中获取更多关于该`clientfd`的信息（如：知道这个`fd`对应的是什么类型的连接等）。

关于`epoll`的一些结构体：

```c++
typedef union epoll_data 
{
	void        *ptr;
  	int          fd;
  	uint32_t     u32;
  	uint64_t     u64;
} epoll_data_t;

struct epoll_event 
{
	uint32_t     events;      /* Epoll events; 如:EPOLLIN */
  	epoll_data_t data;        /* User data variable */
};

int epfd = epoll_create1(0);
int epoll_ctl(int epfd, int operation, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event *events,int maxevents, int timeout);
```

