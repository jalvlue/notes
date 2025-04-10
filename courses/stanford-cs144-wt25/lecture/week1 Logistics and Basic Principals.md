- 关于计算机网络的一门课
- ![[Pasted image 20250325153123.png]]

# Three key concepts
- `datagrams`
	- `self-contained`
	- 包含 `header(destination...), payload(data...)`
	- `service abstraction`: 
		- 网络 (互联网/因特网) 对所有的 `datagrams` 会做 `best efforts delivery`, 尽力去传输, 意味着所有的 `datagrams` 都是相互独立的, 如果发送了一连串, 则可能丢失一部分, 可能在传输过程中被修改, 可能不按顺序到达...
		- 网络中的任何一个组成都知道如何将这个 `datagrams` 传递到 `destination(next hop)`
		- 有自己的格式 ![[Pasted image 20250326013915.png]]
- `encapsulation`
	- 任何的对象都可以作为 `datagrams` 的 `payload`, 即使本身已经封装过 `headers`, 同时封装保证不会损坏其内部的内容, 只让 `associative endpoint` 检查其中的内容
- `multiplexing`
	- `sharing a resource by dispatching according to a runtime identifier`
	- `ip address` 共享同一个数据链 ip (内网, 子网)
	- `protocol field` 让一个端口可以被多种网络协议使用
	- `port number` 让同一种协议可以在计算机上的多个程序上同时运行

- 发送 `datagrams` 的例子
```cpp
#include "socket.hh"
#include <cstdlib>
#include <iostream>
using namespace std;
// NOLINTBEGIN(*-casting)
class RawSocket : public DatagramSocket
{
public:
  RawSocket() : DatagramSocket( AF_INET, SOCK_RAW, IPPROTO_RAW ) {}
};

auto zeroes( auto n )
{
  return string( n, 0 );
}

void send_internet_datagram( const string& payload )
{
  RawSocket sock;
  string datagram;
  datagram += char( 0b0100'0101 ); // v4, and headerlength 5words
  datagram += zeroes( 7 );
  datagram += char( 64 );  // TTL
  datagram += char( 1 );   // protocol
  datagram += zeroes( 6 ); // checksum + src address
  datagram += char( 34 );
  datagram += char( 93 );
  datagram += char( 94 );
  datagram += char( 131 );
  datagram += payload;
  sock.sendto( Address { "1" }, datagram );
}

void send_icmp_message( const string& payload )
{
  send_internet_datagram( "\x08" + payload );
}

void program_body()
{
  string payload;
  while ( cin.good() ) {
    getline( cin, payload );
    send_icmp_message( payload + "\n" );
  }
}

int main()
{
  try {
    program_body();
  } catch ( const exception& e ) {
    cerr << e.what() << "\n";
    return EXIT_FAILURE;
  }
  return EXIT_SUCCESS;
}

// NOLINTEND(*-casting)

```