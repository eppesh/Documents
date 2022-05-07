[toc]

# Basic Knowledge

## 内核缓冲区的个人理解

TCP协议是针对端对端的数据传送，即发送端和接收端。
在操作系统中有用户空间(`user space`)和内核空间(`kernel space`)；
每个TCP连接的每一端，在内核中都有一个**发送缓冲区**或**接收缓冲区**，可理解为 TCP缓冲区。

<img src="https://bigbyto.gitee.io//assets/images/io-model/ioflow.png" alt="pic" style="zoom: 67%;" />

**IO操作例1：网络通信**

- 客户端向服务端发送数据：
  客户端调用`send()`，只是把应用层(位于user sapce)的数据**拷贝**到`connfd`的内核的**发送缓冲区**中就完事了；之后是TCP把数据真正发送到对端。
- 服务端接收客户端的数据：

1. 服务端`clienfd`的内核的**接收缓冲区**前提是得有数据（有数据时系统会把数据从硬件--网络适配器放到内核的接收缓冲区中）；
2. 服务端调用`recv()`，把客户端发送来的数据从`clientfd`的内核的**接收缓冲区拷贝**给应用程序（user sapce中）；

**IO操作例2：文件读写**

- 应用程序A往磁盘**写入**数据：
  A通过系统调用`write()`把待写入的数据从用户空间**拷贝**到内核的**缓冲区**；之后系统会把数据写入磁盘；
- 应用程序A从磁盘**读取**数据：

1. 内核的**缓冲区**前提是得有数据（有数据时系统会把数据从硬件--磁盘放到内核的缓冲区中）；
2. A通过系统调用`read()`，把要读取的数据从内核的**缓冲区拷贝**到用户空间供应用A使用。

结合上面的例子，可知：
对于处于用户空间的应用程序A来说：

- 只有内核的**接收缓冲区**中有数据时，才可以**读取/接收**(`read(),recv() `)数据；
- 只有内核的**发送缓冲区**中有数据时，才可以**写入/发送**(`write(),send()`)数据；

缓冲区**有数据**可以**读取/接收**，称为**可读事件**；
缓冲区**有空间**可以**写入/发送**，称为**可写事件**；
**事件分发**可理解为对于发生的不同事件，采用不同方式的处理。如发生了可读事件，就进行A处理；发生了可写事件，就进行B处理。

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

### socket的个人理解

`socket`称为“套接字”，在Linux中它本质是一个**文件描述符(File Discriptor**,简称**fd**). 
1. 网络通信可以把网线理解为一根线段`AB`，`A`和`B`分别是线段的两头（Endpoint）。每个`socketfd`对应一头，要么是`A`头，要么是`B`头，每一头/每个端点(Endpoint)都对应着一个IP和一个Port。
2. 指定三类套接字`socketfd`，方便说明。
`listenfd`: **服务端**用于**监听**的`socketfd`；
`clientfd`: **服务端**获取到的**已连接成功的**客户端`socketfd`;(即`accept()`一个客户端后新产生的那个`socketfd`)；
`connfd`: **客户端**调用`connect()`与服务端建立连接后返回的`socketfd`；

可以想像，有`N`个客户端和`1`个服务端，并且这`N`个客户端都成功连接上服务端的场景：
- 服务端有`1`个`listenfd`用来监听；
- 服务端有`N`个`clientfd`一一对应连接上的客户端；
- 客户端有`N`个`connfd`对应着服务端上的`N`个`clientfd`，此时`connfd`就相当于线段`AB`中的`A`端，`clientfd`相当于线段`AB`中的`B`端，通信就在这两个端头进行着，直到连接断开。

注意：

- `listen()`的第二个参数是告诉内核设置**连接队列**的长度。

  内核为每个`listen`状态的`socket`设置两个队列：未完成连接队列(incomplete connection queue)和已完成连接队列(completed connection queue)，这两个队列共用`listen()`设置的连接队列的长度。

  当客户端发送`SYN`报文的时候（第1次握手），服务器检测未完成连接队列是否已满，如果满了就丢弃该`SYN`，如果没满就把这个连接放入未完成队列，并发送`ACK + SYN`（第2次握手）； 当客户端收到`ACK + SYN`后，会发送`ACK`报文（第3次握手），并从`connect()`返回；服务器收到这个`ACK`后把连接从未完成连接队列中取出放入已完成连接队列中，等待`accept()`把这个连接取走。此时，三次握手全部完成，两端连接都是`ESTABLISH`状态。

- `accept()`只会从`listen()`的已完成的连接队列中取连接出来（拿**现成的**、已连接上的连接，而不是字面意义上的”接收“），因此，三次握手发生在`accept()`之前；即，`accept()`返回时，连接已经是建立好了的。

  > The backlog can be reached if the completed connection queue fills (i.e., the server process or the server host is so busy that the process cannot call `accept` fast enough to take the completed entries off the queue) or if the incomplete connection queue fills. The latter is the problem that HTTP servers face, when the round-trip time between the client and server is long, compared to the arrival rate of new connection requests, because a new SYN occupies an entry on this queue for one round-trip time. […]
  >
  > The completed connection queue is almost always empty because when an entry is placed on this queue, the server's call to `accept` returns, and the server takes the completed connection off the queue.

参考：

- [How TCP backlog works in Linux](http://veithen.io/2014/01/01/how-tcp-backlog-works-in-linux.html)；
- [connect accept listen 与三次握手的关系](https://blog.csdn.net/qq_33113661/article/details/88837538); 

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

## 参考资料

- [100%弄明白5种I/O模型](https://zhuanlan.zhihu.com/p/115912936); 
- [一篇文章读懂阻塞，非阻塞，同步，异步](https://www.jianshu.com/p/b8203d46895c); 
- [IO多路复用的三种机制Select，Poll，Epoll](https://www.jianshu.com/p/397449cadc9a); 
- [Blocking vs. non-blocking sockets](https://www.scottklement.com/rpg/socktut/nonblocking.html); 
- [带你彻底理解Linux五种I/O模型](https://wiyi.org/linux-io-model.html); 
- [理解Linux中的file descriptor(文件描述符)](https://wiyi.org/linux-file-descriptor.html); 
- [Five IO models of Unix](https://developpaper.com/five-io-models-of-unix/);
- [EPOLLET VS EPOLLLT](https://zhuanlan.zhihu.com/p/36977578); 
- [CSDN-C++网络编程实战项目--Sinetlib网络库（2）——I/O复用与事件分发](https://blog.csdn.net/silence1772/article/details/83307716);
- [CSDN-478-手写C++muduo库(Channel)](https://blog.csdn.net/LINZEYU666/article/details/119908766);
- [Github-day05-epoll高级用法-Channel登场](https://github.com/yuesong-feng/30dayMakeCppServer/blob/main/day05-epoll%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95-Channel%E7%99%BB%E5%9C%BA.md);
- [知乎-Reactor模式介绍](https://zhuanlan.zhihu.com/p/428693405);

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

  该类中的`sockfd_`既可以是`listenfd`，也可以是`clientfd`; 前者调用默认构造函数创建，并可使用所有方法；后者通过带参构造函数来赋值，只使用`SetNonBlocking()`和`GetFd()`两个方法。

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
    Socket();                                       // 默认构造函数表示是新建listenfd,并赋值给sockfd_
    Socket(int sockfd);                             // 带参构造函数表示是客户端的clientfd,并赋值给sockfd_
    ~Socket();

    // listenfd 使用下面5个方法
    void Bind(InetAddress *address);                // 绑定IP和Port
    void Listen();                                  // 监听
    int Accept(InetAddress *address);               // 接受连接,返回客户端fd

    // clientfd 只使用下列2个方法
    void SetNonBlocking();                          // 设置为非阻塞
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

Sean注：这里使用`Channel`类来封装`fd`，类似`ClockIn`项目中的`GridInfo`结构体，其实也就是`fd`与`Channel`一个对应关系；

> 可以结合这个 [EventBase类](https://blog.csdn.net/silence1772/article/details/83307716) 来理解，这个`EventBase`就是每个`fd`对应的相关事件和事件处理函数，在`Epoll`类中用一个`map`把`fd`和`EventBase`映射起来。

在`epoll_wait()`中返回的`events`就是已就绪的事件集(可以记为`active_events`，包括事件名`active_events[i].events`和数据`active_events[i].data`)。

没使用`Channel`类前，我们是返回一个`std::vector<epoll_event> active_events`，主要使用`active_events[i].data.fd`这个`int`型的值; 

而使用`Channel`类后，是返回一个`Channel *active_events=new Channel[kMaxEvens]`数组指针，这次主要使用`active_events[i].data.ptr`这个指针。

> 疑问：明白这里想用Channel来代替之前的fd，但`active_events[i].data.ptr`指向的内容是什么，有多长，可以直接转换成`Channel *`吗？需要读内存验证下。
>
> 为何不直接做一个类似`std::map<int fd, Channel channel>`的映射？

除下面这些文件有改动外，其他文件未做修改：

- 更新`Makefile`：添加`channel.cpp`；

- 添加`Channel`类，位于`channel.h, channel.cpp`文件中；

  ```c++
  // 名称：channel.h
  // 版权：仅供学习
  // 作者：Sean (eppesh@163.com)
  // 环境：VS2019
  // 时间：04/28/2022
  // 说明：Channel类; 把fd封装成Channel类;一个fd对应一个Channel类(之前是将fd添加到epoll的事件表/红黑树中,现在是把Channel添加进去)；该类中包含该fd关心的事件以及事件处理函数
  
  #ifndef CHANNEL_H_
  #define CHANNEL_H_
  
  #include <sys/epoll.h>
  
  #include "epoll.h"
  
  class Epoll;
  
  class Channel
  {
  private:
      Epoll *epfd_;                                           // 内核中 epoll 事件表的标识
      int fd_;                                                // 该socket对应的fd
      uint32_t events_;                                       // 关注的事件
      uint32_t revents_;                                      // 返回的活跃事件; 当有事件发生时，发生的事件类型会写到revents_中,然后就可以根据这个值来决定执行哪个处理函数
      bool in_epoll_;                                         // 当前Channel是否已经在epoll的事件表（红黑树）中;(可理解为该Channel一一对应的fd是否已经在epoll的事件表中)
  
  public:
      Channel(Epoll *epfd, int fd);
      ~Channel();
  
      void EnableReadEvents();                                // 关注可读事件
  
      int GetFd();
      uint32_t GetEvents();
      uint32_t GetRevents();
      void SetRevents(uint32_t revents);
      bool GetInEpoll();
      void SetInEpoll();
  };
  #endif
  
  // channel.cpp
  #include "channel.h"
  
  Channel::Channel(Epoll *epfd, int fd) :epfd_(epfd), fd_(fd), events_(0), revents_(0), in_epoll_(false)
  {
  }
  
  Channel::~Channel()
  {
  }
  
  void Channel::EnableReadEvents()
  {
      events_ = EPOLLIN | EPOLLET;    // events_ |= (EPOLLIN | EPOLLET); 是否更合理？
      epfd_->UpdateChannel(this);
  }
  
  int Channel::GetFd()
  {
      return fd_;
  }
  
  uint32_t Channel::GetEvents()
  {
      return events_;
  }
  
  uint32_t Channel::GetRevents()
  {
      return revents_;
  }
  
  void Channel::SetRevents(uint32_t ev)
  {
      revents_ = ev;
  }
  
  bool Channel::GetInEpoll()
  {
      return in_epoll_;
  }
  
  void Channel::SetInEpoll()
  {
      in_epoll_ = true;
  }
  ```

- 更新`Epoll`类，包括`epoll.h, epoll.cpp`文件：

  ```c++
  // epoll.h
  #ifndef EPOLL_H_
  #define EPOLL_H_
  #include <cstring>
  #include <sys/epoll.h>
  #include <unistd.h>
  #include <vector>
  #include "channel.h"
  #include "util.h"
  const int kMaxEvents = 1000;
  class Channel;
  class Epoll
  {
  private:
      int epoll_fd_;                                          // epoll自身的fd，用来唯一标识内核中的epoll事件表(红黑树)
      epoll_event *active_events_;                            // 传给epoll_wait()的参数,epoll_wait()将会把就绪的事件写入到该参数中返回给用户;此处用的是epoll_event数组的指针,也可以用std::vector<epoll_event> active_events_;
  
  public:
      Epoll();
      ~Epoll();
  
      void AddFd(int fd, uint32_t events);                    // 向事件表中注册/添加关心的fd和事件
      void UpdateChannel(Channel *channel);                   // 向事件表中add,modify关心的Channel(即fd)
      //std::vector<epoll_event> Poll(int timeout = -1);      // 返回就绪的fd
      std::vector<Channel *> Poll(int timeout = -1);          // 轮询并返回就绪的Channel
  };
  #endif
  
  // epoll.cpp
  #include "epoll.h"
  // 修改部分如下
  std::vector<Channel *> Epoll::Poll(int timeout /* = -1 */)
  {
      std::vector<Channel *> active_channels;
      int nfds = epoll_wait(epoll_fd_, active_events_, kMaxEvents, timeout);
      ErrorIf((nfds == -1), "epoll wait error");
  
      for (int i = 0; i < nfds; ++i)
      {
          Channel *channel = (Channel *)active_events_[i].data.ptr;
          channel->SetRevents(active_events_[i].events);
          active_channels.push_back(channel);
      }
      return active_channels;
  }
  
  void Epoll::UpdateChannel(Channel *channel)
  {
      int fd = channel->GetFd();
      epoll_event ev;
      memset(&ev, 0, sizeof(ev));
      ev.data.ptr = channel;
      ev.events = channel->GetEvents();
      if (!channel->GetInEpoll())         // 该channel/fd还不在epoll的事件表中(还未注册)
      {
          ErrorIf((epoll_ctl(epoll_fd_, EPOLL_CTL_ADD, fd, &ev) == -1), "epoll add error");
          channel->SetInEpoll();
      }
      else
      {
          ErrorIf((epoll_ctl(epoll_fd_, EPOLL_CTL_MOD, fd, &ev) == -1), "epoll modify error");
      }
  }
  ```

- 更新`server.cpp`: (只有`main()`做了相应修改)

  ```c++
  #include "channel.h"
  int main()
  {
      Socket *server_socket = new Socket();
      InetAddress *server_address = new InetAddress("127.0.0.1", 8888);
      server_socket->Bind(server_address);
      server_socket->Listen();
  
      Epoll *epoll_fd = new Epoll();
      server_socket->SetNonBlocking();
  
      Channel *listen_channel = new Channel(epoll_fd, server_socket->GetFd());
      listen_channel->EnableReadEvents(); // 关注可读事件
  
      while (true)
      {
          std::vector<Channel *> active_channels = epoll_fd->Poll();
          int nfds = active_channels.size();
          for (int i = 0; i < nfds; ++i)
          {
              int channel_fd = active_channels[i]->GetFd();
              if (channel_fd == server_socket->GetFd())    // 发生事件的fd是服务器的监听fd(listening_sockfd)，表示有新客户端连接
              {
                  InetAddress *client_addr = new InetAddress();                               // TODO: 记得内存释放
                  Socket *client_socket = new Socket(server_socket->Accept(client_addr));     // TODO: 记得内存释放
                  printf("new client fd[%d]! IP: %s Port: %d\n", client_socket->GetFd(), inet_ntoa(client_addr->address_.sin_addr), ntohs(client_addr->address_.sin_port));
                  
                  client_socket->SetNonBlocking();
                  Channel *client_channel = new Channel(epoll_fd, client_socket->GetFd());
                  client_channel->EnableReadEvents();
              }
              else if (active_channels[i]->GetRevents() & EPOLLIN)        // 就绪事件的是客户端，并且是可读事件
              {
                  HandleReadEvent(channel_fd);
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
      if (epoll_fd != nullptr)
      {
          delete epoll_fd;
          epoll_fd = nullptr;
      }
      return 0;
  }
  ```

## 事件驱动核心类

将之前的过程式编程向着面向对象编程修改。本质是事件驱动。[如何深刻理解Reactor和Proactor？小林coding的回答](https://www.zhihu.com/question/26943938/answer/1856426252) 这篇对Reactor模式解释地蛮清楚的。这一小篇章用来将之前的内容分离成**服务器模块**和**事件驱动模块**，从而把服务器逐渐改造成Reactor模式。

- 服务器模块：核心是`Server`类，负责与客户端建立连接并进行业务处理；
- 事件驱动模块：核心是`EventLoop`类和`Channel`类；
  - `EventLoop`类：负责`sockfd`及事件的监听和事件分发；
  - `Channel`类：负责事件的注册，以及绑定事件处理的回调函数；

一个`EventLoop`对应一个内核中事件表`epoll_fd`（供全局使用）；

一个`Channel`对应一个关心的`sockfd`，以及该`sockfd`上关心的事件和事件对应的处理方式（该处理方式可以通过设置关心的事件时，设置该事件的处理方式的回调函数）；

一个`Server`负责建立连接（在新建`Channel`时根据不同的`sockfd`绑定不同的回调函数：是`listen_sockfd`则绑定`NewConnection()`用来新建连接，是`client_sockfd`则绑定`HandleReadEvent()`读写数据），以及进行业务处理（read->业务处理->send）（目前版本是这样）；

新建的`EventLoop`类相当于一个`Reactor`（即分发器`Dispatcher`），这个类首先生成内核中的事件表实例`epoll_fd`（通过`Epoll`类实现，并通过`Channel`类对关心的`sockfd`和相应事件的注册/修改、以及绑定事件处理的回调函数），然后循环不停地监听，一旦有事件就绪就**分发**给`Handler`处理（具体的`Handler`即每个`Channel`绑定的回调处理函数；

需要修改的文件：

- 把代码放到`src`目录下, `Makefile`做相应修改；

- 新增`Server`类，位于`src/server.h, src/server.cpp`文件中：

  ```c++
  // server.h
  #ifndef SERVER_H_
  #define SERVER_H_
  
  #include <cstring>
  #include <functional>
  #include <sys/epoll.h>
  #include <unistd.h>
  
  #include "channel.h"
  #include "event_loop.h"
  #include "inet_address.h"
  #include "socket.h"
  
  const int kReadBuffer = 1024;
  
  class EventLoop;
  class Socket;
  
  class Server
  {
  private:
      EventLoop *event_loop_;
  
  public:
      Server(EventLoop *event_loop);
      ~Server();
  
      void HandleReadEvent(int sockfd);
      void NewConnection(Socket *server_socket);
  };
  #endif
  
  // server.cpp
  #include "server.h"
  
  Server::Server(EventLoop *event_loop) :event_loop_(event_loop)
  {
      Socket *server_socket = new Socket();
      InetAddress *server_addr = new InetAddress("127.0.0.1", 8888);
      server_socket->Bind(server_addr);
      server_socket->Listen();
      server_socket->SetNonBlocking();
  
      Channel *server_channel = new Channel(event_loop, server_socket->GetFd());
      std::function<void()> callback = std::bind(&Server::NewConnection, this, server_socket);
      server_channel->SetCallback(callback);
      server_channel->EnableReadEvents();
  }
  
  Server::~Server()
  {
  }
  
  void Server::HandleReadEvent(int sockfd)
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
  
  void Server::NewConnection(Socket *server_socket)
  {
      InetAddress *client_addr = new InetAddress();                               // TODO: 记得内存释放
      Socket *client_socket = new Socket(server_socket->Accept(client_addr));     // TODO: 记得内存释放
      printf("new client fd[%d]! IP: %s Port: %d\n", client_socket->GetFd(), inet_ntoa(client_addr->address_.sin_addr), ntohs(client_addr->address_.sin_port));
  
      client_socket->SetNonBlocking();
      Channel *client_channel = new Channel(event_loop_, client_socket->GetFd());
      std::function<void()> callback = std::bind(&Server::HandleReadEvent, this, client_socket->GetFd());
      client_channel->SetCallback(callback);
      client_channel->EnableReadEvents();
  }
  ```

- 新增`EventLoop`类，位于`src/event_loop.h, src/event_loop.cpp`文件国：

  ```c++
  #ifndef EVENT_LOOP_H_
  #define EVENT_LOOP_H_
  
  #include <sys/epoll.h>
  #include <vector>
  
  #include "channel.h"
  #include "epoll.h"
  
  class Channel;
  class Epoll;
  
  class EventLoop
  {
  private:
      Epoll *epoll_fd_;
      bool is_quit_;
  
  public:
      EventLoop();
      ~EventLoop();
  
      void Loop();
      void UpdateChannel(Channel *channel);
  };
  
  #endif
  
  // event_loop.cpp
  #include "event_loop.h"
  
  EventLoop::EventLoop()
  {
      epoll_fd_ = new Epoll();
      ErrorIf((epoll_fd_ == nullptr), "epoll new error");
  }
  
  EventLoop::~EventLoop()
  {
      if (epoll_fd_ != nullptr)
      {
          delete epoll_fd_;
          epoll_fd_ = nullptr;
      }
  }
  
  void EventLoop::Loop()
  {
      while (!is_quit_)
      {
          std::vector<Channel *> channels;
          channels = epoll_fd_->Poll();
          for (auto it = channels.begin(); it != channels.end(); ++it)
          {
              (*it)->HandleEvents();
          }
      }
  }
  
  void EventLoop::UpdateChannel(Channel *channel)
  {
      epoll_fd_->UpdateChannel(channel);
  }
  ```

- 修改`Channel`类：

  ```c++
  #ifndef CHANNEL_H_
  #define CHANNEL_H_
  
  #include <sys/epoll.h>
  #include <functional>
  
  #include "epoll.h"
  #include "event_loop.h"
  
  class Epoll;
  class EventLoop;
  
  class Channel
  {
  private:
      //Epoll *epfd_;                                           // 内核中 epoll 事件表的标识
      EventLoop *event_loop_;
      int fd_;                                                // 该socket对应的fd
      uint32_t events_;                                       // 关注的事件
      uint32_t revents_;                                      // 返回的活跃事件; 当有事件发生时，发生的事件类型会写到revents_中,然后就可以根据这个值来决定执行哪个处理函数
      bool in_epoll_;                                         // 当前Channel是否已经在epoll的事件表（红黑树）中;(可理解为该Channel一一对应的fd是否已经在epoll的事件表中)
      std::function<void()> callback_;                        // 回调函数
  
  public:
      //Channel(Epoll *epfd, int fd);
      Channel(EventLoop *event_loop, int fd);
      ~Channel();
  
      void EnableReadEvents();                                // 关注可读事件
      void HandleEvents();
  
      int GetFd();
      uint32_t GetEvents();
      uint32_t GetRevents();
      void SetRevents(uint32_t revents);
      bool GetInEpoll();
      void SetInEpoll();
      void SetCallback(std::function<void()> callback);
  };
  #endif
  
  // channel.cpp
  #include "channel.h"
  
  Channel::Channel(EventLoop *event_loop, int fd) :event_loop_(event_loop), fd_(fd), events_(0), revents_(0), in_epoll_(false)
  {
  }
  
  Channel::~Channel()
  {
  }
  
  void Channel::EnableReadEvents()
  {
      events_ = EPOLLIN | EPOLLET;    // events_ |= (EPOLLIN | EPOLLET); 是否更合理？
      event_loop_->UpdateChannel(this);
  }
  
  void Channel::HandleEvents()
  {
      callback_();
  }
  
  int Channel::GetFd()
  {
      return fd_;
  }
  
  uint32_t Channel::GetEvents()
  {
      return events_;
  }
  
  uint32_t Channel::GetRevents()
  {
      return revents_;
  }
  
  void Channel::SetRevents(uint32_t ev)
  {
      revents_ = ev;
  }
  
  bool Channel::GetInEpoll()
  {
      return in_epoll_;
  }
  
  void Channel::SetInEpoll()
  {
      in_epoll_ = true;
  }
  
  void Channel::SetCallback(std::function<void()> callback)
  {
      callback_ = callback;
  }
  ```

## 给服务器添加一个Acceptor

目前的服务器包括**接受连接**和**echo客户端发来的数据**，现在对其进一步抽象、模块化。

这一小节新建的`Acceptor`类的作用：

1. 使用`listenfd`监听连接请求；
2. 接受并建立连接；【注：这一步的逻辑在`Acceptor`类中，但利用回调函数，其最终的实现代码还位于`Server`类中；】

**疑问**：为何新建连接从逻辑上已经分离到`Acceptor`类中了，但实际工作还是由`Server`类来完成呢？

答：因为新建连接的实际操作不适合放到`Acceptor`类中，会增加其复杂性、不利于模块化，因此把实际操作的工作放到`Server`类中完成。

> 新建连接的实际操作是通过`accept(listenfd,...)`返回一个新的`fd`，即`clientfd`，后续会在该`clientfd`上接收/发送数据。
>
> `Acceptor`类的目的就是把**建立连接**这部分模块化，即它只负责**建立连接**即可，而不用负责对该连接的后续处理。此外，建立连接时的`listenfd`跟建立连接后生成的`clientfd`没有任何关系，因此我们不需要在`Acceptor`类中处理`clientfd`。
>
> 假设我们把新建连接的实际操作也放到`Acceptor`类中，那么新产生的`clientfd`就也需要由`Acceptro`类负责其生命周期；比如，新建连接时`new`出一个`client_socket`对象，那么`Accepter`类需要等`clientfd`数据处理结束后负责`delete`掉这个`client_socket`对象。这显然跟我们的初衷（只负责建立连接）相矛盾。因此，还是应该由`Server`类来实际创建`clientfd`并管理其生命周期更合适。
>
> 另一角度理解：
>
> `Acceptor`类中有一个`Socket`对象，即`listenfd`；后续会把新建连接抽象成一个TCP连接类（即`Connection`类），`Connection`类中也有一个`Socket`对象，即`clientfd`；由于这两个对象没有任何关系，`Acceptor`类和`Connectin`类就像两条平行线，互不相干。如果把`Connection`类放到`Acceptor`类中，显然是不合适的。就像两个级别一样大的人A和B，却要把B交给A管理，是不合理的，应该都交给他们的上级来管理（即由`Server`类这个上级来管理`Acceptor`类和`Connection`类）。

可以发现，对于每一个事件，不论做什么操作，它都位于一个TCP连接上，其前提都是先`accept()`接收这个TCP连接，再把`sockfd`注册到`epoll`事件表中，等后续该TCP连接有就绪事件了再对其处理。因此，可以把**接收连接**这一模块分离出来，添加一个`Acceptor`类。

修改的文件：

- `Makefile`修改：添加`Acceptor`类；

- 新建`Acceptor`类（包含在`acceptor.h, acceptor.cpp`中）；

  ```c++
  #ifndef ACCEPTOR_H_
  #define ACCEPTOR_H_
  
  #include <sys/epoll.h>
  #include <functional>
  
  #include "epoll.h"
  #include "event_loop.h"
  #include "inet_address.h"
  #include "socket.h"
  
  class Epoll;
  class EventLoop;
  class InetAddress;
  class Socket;
  
  class Acceptor
  {
  private:
      EventLoop *event_loop_;
      Socket *socket_;
      InetAddress *addr_;
      Channel *accept_channel_;
  
  public:
      std::function<void(Socket *)> new_connection_callback_;
  
  public:
      Acceptor(EventLoop *event_loop);
      ~Acceptor();
  
      void AcceptConnection();
      void SetNewConnectionCallback(std::function<void(Socket *)> callback);    
  };
  #endif
  
  // acceptor.cpp
  #include "acceptor.h"
  
  Acceptor::Acceptor(EventLoop *event_loop) :event_loop_(event_loop)
  {
      socket_ = new Socket();
      addr_ = new InetAddress("127.0.0.1", 8888);
      socket_->Bind(addr_);
      socket_->Listen();
      socket_->SetNonBlocking();
  
      accept_channel_ = new Channel(event_loop_, socket_->GetFd());
      std::function<void()> callback = std::bind(&Acceptor::AcceptConnection, this);
      accept_channel_->SetCallback(callback);
      accept_channel_->EnableReadEvents();
  }
  
  Acceptor::~Acceptor()
  {
      if (accept_channel_ != nullptr)
      {
          delete accept_channel_;
          accept_channel_ = nullptr;
      }
      if (addr_ != nullptr)
      {
          delete addr_;
          addr_ = nullptr;
      }
      if (socket_ != nullptr)
      {
          delete socket_;
          socket_ = nullptr;
      }
  }
  
  void Acceptor::AcceptConnection()
  {
      new_connection_callback_(socket_);
  }
  
  void Acceptor::SetNewConnectionCallback(std::function<void(Socket * )> callback)
  {
      new_connection_callback_ = callback;
  }
  ```

- 修改`Server`类；

  ```c++
  // server.h中添加了一个private的成员变量,其他不变
      Acceptor *acceptor_;
  // server.cpp 构造函数和析构函数有改动，其他未变
  Server::Server(EventLoop *event_loop) :event_loop_(event_loop), acceptor_(nullptr)
  {
      acceptor_ = new Acceptor(event_loop_);
      std::function<void(Socket *)> callback = std::bind(&Server::NewConnection, this, std::placeholders::_1);
      acceptor_->SetNewConnectionCallback(callback);
  }
  
  Server::~Server()
  {
      if (acceptor_ != nullptr)
      {
          delete acceptor_;
          acceptor_ = nullptr;
      }
  }
  ```
  

### 小结

梳理一下当前的大致流程：（从服务器端的`main()`入口开始）

```c++
// 服务器端 server.cpp (不是本网络库的server类，而是调用正在编写的该网络库的服务来建立这个服务端应用)
int main()
{
    EventLoop *event_loop = new EventLoop();
    Server *server = new Server(event_loop);
    event_loop->Loop();
    return 0;
}
```

1. `new EventLoop()`:

   - `new`一个`Epoll`类的实例`epoll_fd_`，该实例就是内核中`epoll`事务表（红黑树）的唯一标识（由`epoll_create1()`创建）；

     即：这一步在内核中创建了用于IO多路复用的事务表；

2. `new Server()`:

   2.1 `new`一个`Acceptor`：

   - `new`一个`Socket, InetAddress, Channel`，为`Channel::callback_`设置回调函数为`Acceptor::AcceptConnection()`；

     由于在`Channel::HandleEvents()`中才会调用`Channel::callback_`，故调用`Channel::HandleEvents()`时就会调用`Acceptor::AcceptConnection()`; 

     ```c++
     // 有新客户端连接上来时
     // Loop()中监听到listen_sockfd上可读事件就绪, 结合下面的2.2；
     Loop() -> Channel::HandleEvent() 
         -> Acceptor::AcceptConnection() // 上面2.1里对Channel::callback_设置了回调AcceptConnection()
         -> Server::NewConnection();	// 下面2.2里对Acceptor::new_connection_callback_设置了NewConnection()
     ```

   - 向事务表中注册可读事件；

   2.2 为`Acceptor::new_connection_callback_`设置回调函数为`Server::NewConnection()`; 

   由于在`Acceptor::AcceptConnection()`中才会调用`Acceptor::new_connection_callback_`，故调用`Acceptor::AcceptConnection()`时就会调用`Server::NewConnection()`；

3. 开始`EventLoop`中的`Loop()`循环：

   一旦事务表中有就绪事件发生，就调用`Chanel::HandleEvents()`；由于`Chanel::HandleEvents()`中是一个回调函数`Channel::callback_`，因此具体执行要看`Channel::callback_`上绑定哪个回调函数。

4. 在`Server::NewConnection()`被调用后：

   - `new`一个`Socket, InetAddress, Channel`来描述新连接上来的客户端，并为`Channel::callback_`设置回调函数为`Server::HandleReadEvent()`，一旦该客户端上有就绪事件，通过下列流程进行调用：

     ```c++
     Loop() -> Channel::HandleEvent() 
         -> Server::HandleReadEvent();	
     ```

由于该服务的核心是`EventLoop::Loop()`，在这个循环时对就绪事件调用`Channel::HandleEvent()`，其中具体调用的又是`Channel::callback_`上的回调函数，因此，最主要的工作就是在相应逻辑上为`Channel::callback_`设置相应回调函数。

# 建立TCP连接的类

回顾基本的几个关键知识点的伪代码：（可回看上面的[socket的个人理解](#socket的个人理解)]

```c++
// 在服务器端
// step 1: 监听
int listenfd = socket(...);		// 用于监听是否有新连接的fd
bind(listenfd, addr);			// 把该fd跟服务器地址(IP+Port)进行绑定
listen(listenfd, ...);			// 开始监听

// step 2: 建立连接
int clientfd = accept(listenfd, ...);	// 建立连接后返回新连接的fd, listenfd继续去监听
```

目前的`Server`类主要包含有以下几个操作：

- Op1: 在端口上**监听**是否有连接请求（即`listen_sockfd.listen()`)；

  > 这一部分已经在上一节抽象成`Acceptor`类从`Server`类中分离了出去；虽然新建连接的**逻辑**在`Acceptor::AcceptConnection()`中，但新建连接的**实际工作**还是在`Server`类中完成，详见Op3。

- Op2: 对监听到连接请求的客户端**建立连接**（即`int client_sockfd = accept(...)`）；

  > 这一小节就是对“建立连接”这部分再抽象出来一个`Connection`类。

- Op3: 响应操作1(Op1)中最终执行的操作；

  > Op1中监听到连接请求后，最终会调用`Server::NewConnection()`来`accept()`新连接。

- Op4: 响应操作2中最终执行的操作；

  > Op2中建立好每个跟客户端的连接后，最终会调用`Server::HandleReadEvent()`来收发数据、业务处理。

对于`Acceptor`类中新建立的每个TCP连接，我们抽象成一个`Connection`类，它与`Acceptor`类相似，不同之处在于，`Acceptor`类的处理事件（即新建连接）的实际操作位于`Server`类中，而`Connection`类的处理事件的操作不必那样，由自身来完成即可。

对于一个高并发服务器，一般是有一个`Acceptor`用于接受连接（也可以有多个），有成千上万个TCP连接（即成千上万上`Connection`类的实例）。需要把这些TCP连接都保存下来，可以使用`std::map<int, Connection*>`，`key`是`clientfd`，`value`是其对应`Connection`类实例。

修改的地方：

- 修改`Makefile`，添加`Connection`；

- 修改`Server`类：

  ```c++
  class Server
  {
  private:
      EventLoop *event_loop_;
      Acceptor *acceptor_;
      std::map<int, Connection *> connections_;
  
  public:
      Server(EventLoop *event_loop);
      ~Server();
  
      void NewConnection(Socket *listen_socket);          // 新建连接
      void DeleteConnection(Socket *client_socket);       // 删除连接
  };
  // src/server.cpp
  Server::Server(EventLoop *event_loop) :event_loop_(event_loop), acceptor_(nullptr)
  {
      acceptor_ = new Acceptor(event_loop_);
      std::function<void(Socket *)> callback = std::bind(&Server::NewConnection, this, std::placeholders::_1);
      acceptor_->SetNewConnCallback(callback);
  }
  
  Server::~Server()
  {
      if (acceptor_ != nullptr)
      {
          delete acceptor_;
          acceptor_ = nullptr;
      }
  }
  
  void Server::NewConnection(Socket *client_socket)
  {
      Connection *conn = new Connection(event_loop_, client_socket);
      std::function<void(Socket *)> callback = std::bind(&Server::DeleteConnection, this, std::placeholders::_1);
      conn->SetDelConnCallback(callback);
  
      connections_[client_socket->GetFd()] = conn;
  }
  
  void Server::DeleteConnection(Socket *client_socket)
  {
      Connection *conn = connections_[client_socket->GetFd()];
      connections_.erase(client_socket->GetFd());
      if (conn != nullptr)
      {
          delete conn;
          conn = nullptr;
      }
  }
  ```

  

- 新增`Connection`类：

  ```c++
  // 名称：connection.h
  // 版权：仅供学习
  // 作者：Sean (eppesh@163.com)
  // 环境：VS2019
  // 时间：05/05/2022
  // 说明：Connection类,一个TCP连接类
  
  #ifndef CONNECTION_H_
  #define CONNECTION_H_
  
  #include <sys/epoll.h>
  #include <functional>
  
  #include "channel.h"
  #include "event_loop.h"
  #include "socket.h"
  
  const int kReadBuffer = 1024;
  
  class Channel;
  class EventLoop;
  class Socket;
  
  class Connection
  {
  private:
      EventLoop *event_loop_;
      Socket *client_socket_;
      Channel *client_channel_;
      std::function<void(Socket *)> del_conn_callback_;               // 删除连接的回调函数
  
  public:
      Connection(EventLoop *event_loop, Socket *socket);
      ~Connection();
  
      void Echo(int sockfd);                                         // 业务处理函数
      void SetDelConnCallback(std::function<void(Socket *)> callback);    
  };
  #endif
  // connection.cpp
  #include "connection.h"
  
  Connection::Connection(EventLoop *event_loop, Socket *socket) :
      event_loop_(event_loop), client_socket_(socket), client_channel_(nullptr)
  {
      client_channel_ = new Channel(event_loop_, client_socket_->GetFd());
      std::function<void()> callback = std::bind(&Connection::Echo, this, client_socket_->GetFd());
      client_channel_->SetCallback(callback);
      client_channel_->EnableReadEvents();
  }
  
  Connection::~Connection()
  {
      if (client_channel_ != nullptr)
      {
          delete client_channel_;
          client_channel_ = nullptr;
      }
      if (client_socket_ != nullptr)
      {
          delete client_socket_;
          client_socket_ = nullptr;
      }
  }
  
  void Connection::Echo(int sockfd)
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
              //close(sockfd);       // 关闭socket会自动将文件描述符从epoll树上移除
              del_conn_callback_(client_socket_);     // 调用Server::NewConnection()中设置的回调函数,即Server::DeleteConnection();
              break;
          }
          else
          {
              // 其他 read_bytes == -1 的情况处理
          }
      }
  }
  
  void Connection::SetDelConnCallback(std::function<void(Socket *)> callback)
  {
      del_conn_callback_ = callback;
  }
  ```

- 修改`Acceptor`类：

  ```c++
  class Acceptor
  {
  private:
      EventLoop *event_loop_;
      Socket *listen_socket_;
      Channel *accept_channel_;
      std::function<void(Socket *)> new_conn_callback_;
  
  public:
      Acceptor(EventLoop *event_loop);
      ~Acceptor();
  
      void AcceptConnection();
      void SetNewConnCallback(std::function<void(Socket *)> callback);    
  };
  // acceptor.cpp
  Acceptor::Acceptor(EventLoop *event_loop) :event_loop_(event_loop)
  {
      listen_socket_ = new Socket();
      InetAddress *addr = new InetAddress("127.0.0.1", 8888);
      listen_socket_->Bind(addr);
      listen_socket_->Listen();
      listen_socket_->SetNonBlocking();
  
      accept_channel_ = new Channel(event_loop_, listen_socket_->GetFd());
      std::function<void()> callback = std::bind(&Acceptor::AcceptConnection, this);
      accept_channel_->SetCallback(callback);
      accept_channel_->EnableReadEvents();
  }
  
  Acceptor::~Acceptor()
  {
      if (accept_channel_ != nullptr)
      {
          delete accept_channel_;
          accept_channel_ = nullptr;
      }
      if (listen_socket_ != nullptr)
      {
          delete listen_socket_;
          listen_socket_ = nullptr;
      }
  }
  
  void Acceptor::AcceptConnection()
  {    
      InetAddress *client_addr = new InetAddress();
      Socket *client_socket = new Socket(listen_socket_->Accept(client_addr));
      printf("new client fd[%d]! IP: %s Port: %d\n", client_socket->GetFd(), inet_ntoa(client_addr->address_.sin_addr), ntohs(client_addr->address_.sin_port));
      client_socket->SetNonBlocking();
  
      new_conn_callback_(client_socket);
  
      delete client_addr;
      client_addr = nullptr;
  }
  
  void Acceptor::SetNewConnCallback(std::function<void(Socket * )> callback)
  {
      new_conn_callback_ = callback;
  }
  ```

# 使用缓冲区

目前该网络库的组成如下：

- `Server`类：服务器类，核心类；负责管理`Acceptor`类和`Connection`类；
- `EventLoop`类：事件驱动类，核心类；负责维护`epoll`事务表、事件就绪后分发事件；
- `Acceptor`类：连接类；负责建立新的连接；
- `Connection`类：TCP连接类；负责每一个新建立的连接，以及该连接上的业务处理；
- `Socket`类：基础类；包含常用的`socket`操作（`bind, listen, accept`）；
- `Epoll`类：基础类；负责`epoll`相关操作；
- `InetAddress`类：网络地址类，基础类；对网络地址相关操作的封装；
- `Channel`类：基础类；负责把一个`fd`对应的”关心的事件“、”已就绪的事件“、”事件对应的回调函数“等封装整合起来；
- `Util`类：工具类，基础类；包含常用的辅助方法；

本小节为每个`Connection`类分配一个读缓冲区和写缓冲区，从客户端读取来的数据都存放在读缓冲区里。

修改内容如下：

- 修改`Makefile`：添加`src/buffer`; 

- 修改`Connection`类：

  ```c++
  class Connection
  {
  private:
      EventLoop *event_loop_;
      Socket *client_socket_;
      Channel *client_channel_;
      std::function<void(Socket *)> del_conn_callback_;               // 删除连接的回调函数
      std::string *in_buffer_;
      Buffer *read_buffer_;
  
  public:
      Connection(EventLoop *event_loop, Socket *socket);
      ~Connection();
  
      void Echo(int sockfd);                                         // 业务处理函数
      void SetDelConnCallback(std::function<void(Socket *)> callback);    
  };
  // connection.cpp
  Connection::Connection(EventLoop *event_loop, Socket *socket) :
      event_loop_(event_loop), client_socket_(socket), client_channel_(nullptr),
      in_buffer_(new std::string()), read_buffer_(nullptr)
  {
      client_channel_ = new Channel(event_loop_, client_socket_->GetFd());
      std::function<void()> callback = std::bind(&Connection::Echo, this, client_socket_->GetFd());
      client_channel_->SetCallback(callback);
      client_channel_->EnableReadEvents();
      read_buffer_ = new Buffer();
  }
  
  void Connection::Echo(int sockfd)
  {
      char buf[kReadBuffer] = { 0 };
      while (true)    // 由于使用非阻塞IO，读取客户端buffer时，需要不断读取，一次读取buf大小数据，直到全部读取完毕
      {
          memset(buf, 0, sizeof(buf));
          ssize_t read_bytes = read(sockfd, buf, sizeof(buf));
          if (read_bytes > 0)
          {
              read_buffer_->Append(buf, read_bytes);
          }
          else if (read_bytes == -1 && errno == EINTR)        // 客户端正常中断，继续读取
          {
              printf("continue reading");
              continue;
          }
          else if (read_bytes == -1 && ((errno == EAGAIN) || (errno == EWOULDBLOCK)))         // 非阻塞IO，这个条件表示数据全部读取完毕
          {
              printf("finish reading once\n");
              printf("message from client fd[%d]: %s\n", sockfd, read_buffer_->C_str());
              ErrorIf((write(sockfd, read_buffer_->C_str(), read_buffer_->Size()) == -1), "socket write error");
              read_buffer_->Clear();
              break;
          }
          else if (read_bytes == 0)                           // EOF, 客户端断开连接
          {
              printf("EOF, client fd[%d] disconnected\n", sockfd);
              //close(sockfd);       // 关闭socket会自动将文件描述符从epoll树上移除
              del_conn_callback_(client_socket_);     // 调用Server::NewConnection()中设置的回调函数,即Server::DeleteConnection();
              break;
          }
          else
          {
              // 其他 read_bytes == -1 的情况处理
          }
      }
  }
  ```

- 添加`Buffer`类：

  ```c++
  #include <iostream>
  #include <string>
  
  class Buffer
  {
  private:
      std::string buffer_;
  
  public:
      Buffer();
      ~Buffer();
  
      void Append(const char *str, int size);
      ssize_t Size();
      const char *C_str();
      void Clear();
      void Getline();
  };
  // buffer.cpp
  Buffer::Buffer()
  {
  
  }
  
  Buffer::~Buffer()
  {
  
  }
  
  void Buffer::Append(const char *str, int size)
  {
      for (int i = 0; i < size; ++i)
      {
          if (str[i] == '\0')
          {
              break;
          }
          buffer_.push_back(str[i]);
      }
  }
  
  ssize_t Buffer::Size()
  {
      return buffer_.size();
  }
  
  const char *Buffer::C_str()
  {
      return buffer_.c_str();
  }
  
  void Buffer::Clear()
  {
      buffer_.clear();
  }
  
  void Buffer::Getline()
  {
      buffer_.clear();
      std::getline(std::cin, buffer_);
  }
  ```

# 添加线程池

目前是个单线程服务器，所有`fd`上的事件都是在主线程上处理。假设有1000个事件同时到来，每个事件处理需要1秒，那也需要1000秒来传输数据，显然是不合适的。

在`Reactor`模式中，一个`Reactor`应该只负责事件分发而不负责事件处理，因此可以构建一个线程池用于事件处理。

线程池的实现可以有不同思路：

- 思路1：每有一个新任务（就绪的事件），就开一个新线程去处理。

  但这种情况线程数并不固定。假如某一时刻有1000个并发请求，那就需要开1000个线程，但机器未必能支持这么高的并发；而且太多的线程数，线程切换也会耗费大量资源。

- 思路2：采用固定的线程数，可以避免服务器的负载不稳定。具体的工作线程数目，一般是CPU核数（物理支持的最大并发数）。当事件就绪后，将它添加到事件队列中，由工作线程不断主动取出事件队列中的事件进行处理。

  当`Channel`类有事件需要处理时，将这个事件处理添加到线程池，主线程`EventLoop`就可以继续进行事件循环，而不用关心某个`sockfd`上的事件处理。

关于线程池，需注意两点：

1. 多线程环境下对事件队列的读写操作要考虑互斥锁；可使用`std::mutex`; 
2. 当事件队列为空时，CPU应该不再轮询以节省资源；可使用条件变量`std::condition_variable`.

改动的地方较多，不再在这里列举，以实际代码为准。 