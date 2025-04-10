# ch 1 intro
- `linux + c + gcc`

# ch 2 what is a socket
- `unix` 中一切皆文件, `socket` 就是通过网络与外界进行信息交换的中介文件, 由于是文件, 因此程序可以通过 `fd` 对 `socket` 文件进行一些标准的 io 操作, 同时对于 `socket` 还有一些单独的操作
- 操作 `socket`:
	- `socket() system routine` 系统调用来获取 `socket fd`
	- `send() system routine` 系统调用来通过 `fd` 对 `socket file` 写东西 (发送信息)
	- `recv() system routine` 系统调用来通过 `fd` 从 `socket file` 读东西 (接受信息)
	- 作为一个 `unix fd`, 使用 `write(), read()` 直接操作也是可以的
- `socket` 种类
	- `Internet Socket, DARPA Internet addresses` 网络通信使用
	- `Unix Socket, path names on a local node` 单机内进程间通信使用
	- ...


### 2.1 two types of Internet Socket
- 常见的有三种
	- `Stream Socket, SOCK_STREAM`, 面向连接的可靠流式传输 `socket`, 使用 TCP 协议达到这种传输效果
	- `Datagram Socket, SOCK_DGRAM`, 无连接的传输, 以数据包为单位 `socket`, 使用 UDP 协议达到这种传输效果
	- `Raw Socket, SOCK_RAW`
- why stream sockets? 可靠且有序, 很多对数据传输要求很严格的应用需要这种特性, 例如 `telnet, ssh, http`
- why datagram sockets? 不需要维护连接, 直接打包数据就可以通过 `IP` 发送给对方, 速度快, 可拓展性好, 应用层可以自己拓展使其成为可靠传输, 对数据的质量没有太高要求, 例如 `tftp, dhcpcd, multiplayer-games, streaming audio, video conferencing`
```ad-note
If you’re sending chat messages, TCP is great; if you’re sending 40 positional updates per second of the players in the world, maybe it doesn’t matter so much if one or two get dropped, and UDP is a good choice.
```

### 2.2 low level nonsense and network theory
![[Pasted image 20241208181811.png]]
```ad-note
Basically, it says this: a packet is born, the packet is wrapped (“encapsulated”) in a header (and rarely a footer) by the first protocol (say, the TFTP protocol), then the whole thing (TFTP header included) is encapsulated again by the next protocol (say, UDP), then again by the next (IP), then again by the final protocol on the hardware (physical) layer (say, Ethernet).

When another computer receives the packet, the hardware strips the Ethernet header, the kernel strips the IP and UDP headers, the TFTP program strips the TFTP header, and it finally has the data.

ISO/OSI network model
- Application
- Presentation
- Session
- Transport
- Network
- Data Link
- Physical

Unix network model
- Application Layer (_telnet, ftp, etc._)
- Host-to-Host Transport Layer (_TCP, UDP_)
- Internet Layer (_IP and routing_)
- Network Access Layer (_Ethernet, wi-fi, or whatever_)
```
- 整个网络传输就是多个封装-解封装的过程
- 传输过程就是网络边缘-网络核心-网络边缘, 在这个过程中, 一个包会途径多个处于不同协议栈层级的网络设备, 他们会将包解封装到他们的层级, 读出该层级的信息后, 将包重新封装, 根据该信息将包传递到下一个节点
- 在 `unix` 中, 用户可以直接接触并做修改的只有 `application layer`, 其余的都由操作系统和网络设备做好了
- `unix` 为此提供了上面提到的跟网络有关的系统调用


# ch 3 IP addresses, `struct`s, and Data Munging

### 3.1 IP addresses, versions 4 and 6
- `IPv4`, 第一个版本的 `IP address`, 使用 `4-bytes` 来表示一个地址, 总共 `32-bits, 2**32` 个地址, 通常记为 `192.0.2.111, 127.0.0.1`
- `IPv6`, 第二个版本, 使用 `128-bits, 2**128` 来表示一个地址, 通常使用两个字节一组的形式来记录, 总共 8 组
	- `2001:0db8:c9d2:0012:0000:0000:0000:0051, 2001:db8:c9d2:12::51`
	- `2001:0db8:ab00:0000:0000:0000:0000:0000, 2001:db8:ab00::`
	- `0000:0000:0000:0000:0000:0000:0000:0001, ::1`
- `IPv4` 不够用了所以才提出了 `IPv6`

###### 3.1.1 subnets
- 子网的概念通常用于将多个连续的 ip 地址划分给某些企业组织使用, 使用子网掩码来识别一个 ip 所处的子网
```ad-note
The network portion of the IP address is described by something called the _netmask_, which you bitwise-AND with the IP address to get the network number out of it. The netmask usually looks something like `255.255.255.0`. (E.g. with that netmask, if your IP is `192.0.2.12`, then your network is `192.0.2.12` AND `255.255.255.0` which gives `192.0.2.0`.)

But it’s a bit unwieldy to use a big string of numbers like `255.192.0.0` as a netmask. First of all, people don’t have an intuitive idea of how many bits that is, and secondly, it’s really not compact. So the New Style came along, and it’s much nicer. You just put a slash after the IP address, and then follow that by the number of network bits in decimal. Like this: `192.0.2.12/30`.

Or, for IPv6, something like this: `2001:db8::/32` or `2001:db8:5413:4028::9db9/64`.
```

###### 3.1.2 port numbers
- `ip address` 用于标识主机, `port number` 用于在主机内标识应用
- `TCP, UDP` 分别有自己的一套 `port number`
```ad-note
Turns out that besides an IP address (used by the IP layer), there is another address that is used by TCP (stream sockets) and, coincidentally, by UDP (datagram sockets). It is the _port number_. It’s a 16-bit number that’s like the local address for the connection.

Think of the IP address as the street address of a hotel, and the port number as the room number. That’s a decent analogy; maybe later I’ll come up with one involving the automobile industry.

Say you want to have a computer that handles incoming mail AND web services—how do you differentiate between the two on a computer with a single IP address?

Well, different services on the Internet have different well-known port numbers. You can see them all in [the Big IANA Port List](https://www.iana.org/assignments/port-numbers)[15](https://beej.us/guide/bgnet/html/split/more-references.html#fn15) or, if you’re on a Unix box, in your `/etc/services` file. HTTP (the web) is port 80, telnet is port 23, SMTP is port 25, the game [DOOM](https://en.wikipedia.org/wiki/Doom_(1993_video_game))[16](https://beej.us/guide/bgnet/html/split/more-references.html#fn16) used port 666, etc. and so on. Ports under 1024 are often considered special, and usually require special OS privileges to use.
```

# 3.2 byte order
[[little-endian 和 big-endian]]
- `big-endian, network byte order`
- `little-endian`
- 由于不同的 CPU 使用不同的架构, 因此有点使用大端字节序, 有的使用小端字节序, 网络传输为了统一, 使用网络字节序
- 为了实现统一跨平台, 一般操作系统都会提供以下函数将机器字节序信息向应用屏蔽
![[Pasted image 20241208194106.png]]

# 3.3 `struct`s
```c
struct addrinfo {
    int              ai_flags;     // AI_PASSIVE, AI_CANONNAME, etc.
    int              ai_family;    // AF_INET, AF_INET6, AF_UNSPEC
    int              ai_socktype;  // SOCK_STREAM, SOCK_DGRAM
    int              ai_protocol;  // use 0 for "any"
    size_t           ai_addrlen;   // size of ai_addr in bytes
    struct sockaddr *ai_addr;      // struct sockaddr_in or _in6
    char            *ai_canonname; // full canonical hostname

    struct addrinfo *ai_next;      // linked list, next node
};


struct sockaddr {
    unsigned short    sa_family;    // address family, AF_xxx
    char              sa_data[14];  // 14 bytes of protocol address
}; 

// (IPv4 only--see struct sockaddr_in6 for IPv6)
struct sockaddr_in {
    short int          sin_family;  // Address family, AF_INET
    unsigned short int sin_port;    // Port number
    struct in_addr     sin_addr;    // Internet address
    unsigned char      sin_zero[8]; // Same size as struct sockaddr
};

// (IPv4 only--see struct in6_addr for IPv6)
// Internet address (a structure for historical reasons)
struct in_addr {
    uint32_t s_addr; // that's a 32-bit int (4 bytes)
};


// (IPv6 only--see struct sockaddr_in and struct in_addr for IPv4)
struct sockaddr_in6 {
    u_int16_t       sin6_family;   // address family, AF_INET6
    u_int16_t       sin6_port;     // port number, Network Byte Order
    u_int32_t       sin6_flowinfo; // IPv6 flow information
    struct in6_addr sin6_addr;     // IPv6 address
    u_int32_t       sin6_scope_id; // Scope ID
};

struct in6_addr {
    unsigned char   s6_addr[16];   // IPv6 address
};


struct sockaddr_storage {
    sa_family_t  ss_family;     // address family

    // all this is padding, implementation specific, ignore it:
    char      __ss_pad1[_SS_PAD1SIZE];
    int64_t   __ss_align;
    char      __ss_pad2[_SS_PAD2SIZE];
};
```

```ad-note
`struct addrinfo` 是在网络编程中用于存储地址信息的结构体，特别是在使用套接字（sockets）进行网络通信时。这个结构体通常在 POSIX 系统中使用，尤其是在 `getaddrinfo` 函数的返回值中。下面逐字段解释这个结构体的各个成员：

### 字段解释

1. **`int ai_flags`**:
   - 存储地址信息的标志位。
   - 常见的标志包括：
     - `AI_PASSIVE`: 表示返回的地址可以用于被动套接字（即用于 `bind` 的地址）。
     - `AI_CANONNAME`: 表示返回的地址包含完整的规范名称。

2. **`int ai_family`**:
   - 表示地址族（Address Family）。
   - 常见的值有：
     - `AF_INET`: IPv4 地址。
     - `AF_INET6`: IPv6 地址。
     - `AF_UNSPEC`: 不限制地址类型，可以返回 IPv4 或 IPv6。

3. **`int ai_socktype`**:
   - 表示套接字类型。
   - 常见的值包括：
     - `SOCK_STREAM`: 面向连接的套接字（TCP）。
     - `SOCK_DGRAM`: 无连接的套接字（UDP）。
   - 可以用 `0` 表示不限制套接字类型。

4. **`int ai_protocol`**:
   - 表示协议类型。
   - 通常为 `0`，表示不限制协议。具体的协议可以用来指定 TCP 或 UDP。

5. **`size_t ai_addrlen`**:
   - 存储 `ai_addr` 指向的地址结构的大小（以字节为单位）。
   - 这有助于在使用地址时知道需要多少字节。

6. **`struct sockaddr *ai_addr`**:
   - 指向一个 `sockaddr` 结构体，该结构体包含具体的地址信息。
   - 根据 `ai_family` 的不同，这个指针可以指向不同类型的地址结构，例如 `sockaddr_in`（用于 IPv4）或 `sockaddr_in6`（用于 IPv6）。

7. **`char *ai_canonname`**:
   - 指向一个字符串，存储完全的规范名称（canonical name）。
   - 这通常用于 DNS 查询的结果，提供了主机的规范名称。

8. **`struct addrinfo *ai_next`**:
   - 指向下一个 `addrinfo` 结构体，形成一个链表。
   - 当 `getaddrinfo` 返回多个地址时，可以通过这个指针遍历所有地址。

### 用途

- `struct addrinfo` 主要用于网络编程，尤其是解析主机名和服务名以获取可用于创建套接字的地址信息。
- 使用 `getaddrinfo` 函数可以自动处理地址的解析和转换，返回一个 `addrinfo` 结构体的链表，程序可以遍历这个链表并创建所需的套接字。

### 示例

以下是一个使用 `getaddrinfo` 和 `addrinfo` 结构体的简单示例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

int main() {
    struct addrinfo hints, *res;
    int status;

    memset(&hints, 0, sizeof hints);
    hints.ai_family = AF_UNSPEC; // 不限制 IPv4 或 IPv6
    hints.ai_socktype = SOCK_STREAM; // TCP

    // 解析主机名
    if ((status = getaddrinfo("www.example.com", "80", &hints, &res)) != 0) {
        fprintf(stderr, "getaddrinfo error: %s\n", gai_strerror(status));
        return 1;
    }

    // 使用 res 进行 socket 创建等操作

    freeaddrinfo(res); // 释放返回的地址信息
    return 0;
}
```


### 3.4 IP addresses, part Deux
- 直接操作 `ip addresses, addrinfo, sockaddr` 是比较少见的, 有很多库函数提供了很多方便的功能
- `getaddrinfo()`, 获取某个主机某个服务的信息, 后续可以利用这些信息与该服务创建 `socket connection`
- `inet_pton, convert Internet address from presentation format to network format`, 从一个人类可读的 `presentation format` 转为网络编程中常用的 `sockaddr`
- `inet_ntop`, 与上面的函数相反
```c
#include <arpa/inet.h>
#include <netdb.h>
#include <netinet/in.h>
#include <stdio.h>

int main() {
  char ip4[INET_ADDRSTRLEN];
  struct sockaddr_in sa;

  inet_pton(AF_INET, "10.12.110.57", &(sa.sin_addr));
  inet_ntop(AF_INET, &(sa.sin_addr), ip4, INET_ADDRSTRLEN);
  printf("The IPv4 address is: %s\n", ip4);

  char ip6[INET6_ADDRSTRLEN];
  struct sockaddr_in6 sa6;
  inet_pton(AF_INET6, "2001:db8:63b3:1::3490", &(sa6.sin6_addr)); // IPv6
  inet_ntop(AF_INET6, &(sa6.sin6_addr), ip6, INET6_ADDRSTRLEN);
  printf("The IPv6 address is: %s\n", ip6);

  return 0;
}

```

```c
#include <netdb.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/types.h>

int main() {
  struct addrinfo hints, *res;
  int status;

  memset(&hints, 0, sizeof(hints));
  hints.ai_family = AF_UNSPEC;     // 不限制 IPv4 或 IPv6
  hints.ai_socktype = SOCK_STREAM; // TCP

  // 解析主机名
  if ((status = getaddrinfo("www.example.com", "80", &hints, &res)) != 0) {
    fprintf(stderr, "getaddrinfo error: %s\n", gai_strerror(status));
    return 1;
  }

  // 使用 res 进行 socket 创建等操作

  freeaddrinfo(res); // 释放返回的地址信息
  return 0;
}

```


###### 3.4.1 private networks
- 从公网到局域网
- `NAT, network address translation`
- `10.x.x.x`
- `172.y.x.x, 16 <= y <= 31`
- `192.168.x.x`
- 

# ch 4 jumping from IPv4 to IPv6
- 这里主要是讲了使用哪些兼容 `IPv6` 的库函数来让程序兼容 `IPv6`

```ad-note
1. First of all, try to use [`getaddrinfo()`](https://beej.us/guide/bgnet/html/split/ip-addresses-structs-and-data-munging.html#structs) to get all the `struct sockaddr` info, instead of packing the structures by hand. This will keep you IP version-agnostic, and will eliminate many of the subsequent steps.
    
2. Any place that you find you’re hard-coding anything related to the IP version, try to wrap up in a helper function.
    
3. Change `AF_INET` to `AF_INET6`.
    
4. Change `PF_INET` to `PF_INET6`.
    
5. Change `INADDR_ANY` assignments to `in6addr_any` assignments, which are slightly different:
    
    ```
    struct sockaddr_in sa;
    struct sockaddr_in6 sa6;
    
    sa.sin_addr.s_addr = INADDR_ANY;  // use my IPv4 address
    sa6.sin6_addr = in6addr_any; // use my IPv6 address
    ```
    
    Also, the value `IN6ADDR_ANY_INIT` can be used as an initializer when the `struct in6_addr` is declared, like so:
    
    ```
    struct in6_addr ia6 = IN6ADDR_ANY_INIT;
    ```
    
6. Instead of `struct sockaddr_in` use `struct sockaddr_in6`, being sure to add “6” to the fields as appropriate (see [`struct`s](https://beej.us/guide/bgnet/html/split/ip-addresses-structs-and-data-munging.html#structs), above). There is no `sin6_zero` field.
    
7. Instead of `struct in_addr` use `struct in6_addr`, being sure to add “6” to the fields as appropriate (see [`struct`s](https://beej.us/guide/bgnet/html/split/ip-addresses-structs-and-data-munging.html#structs), above).
    
8. Instead of `inet_aton()` or `inet_addr()`, use `inet_pton()`.
    
9. Instead of `inet_ntoa()`, use `inet_ntop()`.
    
10. Instead of `gethostbyname()`, use the superior `getaddrinfo()`.
    
11. Instead of `gethostbyaddr()`, use the superior `getnameinfo()` (although `gethostbyaddr()` can still work with IPv6).
    
12. `INADDR_BROADCAST` no longer works. Use IPv6 multicast instead.
```


# ch 5 system calls or bust
- 网络编程相关系统调用和库函数

### 5.1 `getaddrinfo()`, prepare to launch
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

int getaddrinfo(const char *node,     // e.g. "www.example.com" or IP
                const char *service,  // e.g. "http" or port number
                const struct addrinfo *hints,
                struct addrinfo **res);

// $ showip www.example.com
#include <arpa/inet.h>
#include <netdb.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[]) {
  struct addrinfo hints, *res, *p;
  int status;
  char ipstr[INET6_ADDRSTRLEN];

  if (argc != 2) {
    fprintf(stderr, "usage: showip hostname\n");
    return 1;
  }

  // setup hints
  memset(&hints, 0, sizeof(hints));
  // any IP version
  hints.ai_family = AF_UNSPEC;
  // TCP stream sockets
  hints.ai_socktype = SOCK_STREAM;

  // getaddrinfo
  if ((status = getaddrinfo(argv[1], NULL, &hints, &res))) {
    fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(status));
    return 2;
  }

  printf("IP addresses for %s:\n\n", argv[1]);

  for (p = res; p != NULL; p = p->ai_next) {
    void* addr;
    char* ipver;

    if (p->ai_family == AF_INET) {
      // IPv4
      struct sockaddr_in* ipv4 = (struct sockaddr_in*)p->ai_addr;
      addr = &(ipv4->sin_addr);
      ipver = "IPv4";
    } else {
      // IPv6
      struct sockaddr_in6* ipv6 = (struct sockaddr_in6*)p->ai_addr;
      addr = &(ipv6->sin6_addr);
      ipver = "IPv6";
    }

    // convert the IP to a string and print it
    inet_ntop(p->ai_family, addr, ipstr, sizeof(ipstr));
    printf("  %s: %s\n", ipver, ipstr);
  }

  freeaddrinfo(res);

  return 0;
}

```

### 5.2 `socket()`, get the file descriptor
- `AF, address family`, 地址族, 通常跟具体的网络地址有关
- `PF, protocol family`, 协议族, 通常跟某种网络的某种协议有关
```ad-note
It used to be people would hardcode these values, and you can absolutely still do that. (`domain` is `PF_INET` or `PF_INET6`, `type` is `SOCK_STREAM` or `SOCK_DGRAM`, and `protocol` can be set to `0` to choose the proper protocol for the given `type`. Or you can call `getprotobyname()` to look up the protocol you want, “tcp” or “udp”.)

(This `PF_INET` thing is a close relative of the `AF_INET` that you can use when initializing the `sin_family` field in your `struct sockaddr_in`. In fact, they’re so closely related that they actually have the same value, and many programmers will call `socket()` and pass `AF_INET` as the first argument instead of `PF_INET`. Now, get some milk and cookies, because it’s time for a story. Once upon a time, a long time ago, it was thought that maybe an address family (what the “AF” in “`AF_INET`” stands for) might support several protocols that were referred to by their protocol family (what the “PF” in “`PF_INET`” stands for). That didn’t happen. And they all lived happily ever after, The End. So the most correct thing to do is to use `AF_INET` in your `struct sockaddr_in` and `PF_INET` in your call to `socket()`.)
```

```c
#include <sys/types.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol); 

#include <arpa/inet.h>
#include <netdb.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>

int main(int argc, char* argv[]) {
  int fd;
  struct addrinfo hints, *res;
  getaddrinfo("www.example.com", "http", &hints, &res);

  fd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

  return 0;
}

```

### 5.3 `bind()`, what port am i on
- 需要接受外来外来请求的服务器使用
- 通过 `socket()` 获得了 `socket fd` 之后, 如果需要监听外来请求, 那么就需要使用 `bind()` 系统调用来告诉操作系统你将监听哪个端口, 后续操作系统会将来自该端口的请求转发给你
```c
#include <sys/types.h>
#include <sys/socket.h>

int bind(int sockfd, struct sockaddr *my_addr, int addrlen);
```

### 5.4 `connect()`, hey you!
- `connect()` 通常用于客户端程序, 也就是对程序的端口没有指定需求的程序, 会调用 `bind()` 选择一个可用的空闲端口
```c
#include <sys/types.h>
#include <sys/socket.h>

int connect(int sockfd, struct sockaddr *serv_addr, int addrlen);

#include <arpa/inet.h>
#include <netdb.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/types.h>

int main(int argc, char* argv[]) {
  int sockfd;
  struct addrinfo hints, *res;

  memset(&hints, 0, sizeof(hints));
  hints.ai_family = AF_UNSPEC;
  hints.ai_socktype = SOCK_STREAM;

  getaddrinfo("www.example.com", "3490", &hints, &res);

  sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

  connect(sockfd, res->ai_addr, res->ai_addrlen);

  return 0;
}

```

### 5.5 `listen()`, will somebody please call me
- 服务器使用, 用于监听到来的请求
- 参数中, `int backlog` 指的是请求队列的最大长度
```c
int listen(int sockfd, int backlog)

// a typical server-side code
getaddrinfo();
socket();
bind();
listen();
/* accept() goes here */ 
```

### 5.6 `accept()`, thank you for calling port 3490
- `accept()` 的成功调用会创建一个新的 `socket file`, 来表示这个请求单独的传递中介, 后续与该请求进行交互都是通过这个新 `socket file` 进行, 原来的 `socket` 还是用于监听端口接收请求
```ad-note
someone far far away will try to `connect()` to your machine on a port that you are `listen()`ing on. Their connection will be queued up waiting to be `accept()`ed. You call `accept()` and you tell it to get the pending connection. It’ll return to you a _brand new socket file descriptor_ to use for this single connection! That’s right, suddenly you have _two socket file descriptors_ for the price of one! The original one is still listening for more new connections, and the newly created one is finally ready to `send()` and `recv()`. We’re there!
```

```c
#include <sys/types.h>
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen); 

#include <netdb.h>
#include <string.h>
#include <sys/socket.h>

#define MYPORT "3490"
#define BACKLOG 10

int main(void) {
  struct sockaddr_storage their_addr;
  socklen_t addr_size;
  struct addrinfo hints, *res;
  int sockfd, new_fd;

  memset(&hints, 0, sizeof(hints));
  hints.ai_family = AF_UNSPEC;
  hints.ai_socktype = SOCK_STREAM;
  hints.ai_flags = AI_PASSIVE;

  getaddrinfo(NULL, MYPORT, &hints, &res);

  sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

  bind(sockfd, res->ai_addr, res->ai_addrlen);

  listen(sockfd, BACKLOG);

  addr_size = sizeof(their_addr);
  new_fd = accept(sockfd, (struct sockaddr*)&their_addr, &addr_size);

  return 0;
}

```

### 5.7 `send()` and `recv()`, talk to me
- `send(), recv()` 用于 `stream sockets, connected datagram sockets`
- 对于普通的 `unconnected datagram sockets`, 应该使用 `sendto(), recvfrom()`

