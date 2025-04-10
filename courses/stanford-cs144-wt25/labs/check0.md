# Webget
- 使用 `os socket interface` 编写 `client code` 向 `http web server` 发送 `GET` 请求
- 简单 `client socket connect + write request + print response`
```cpp
#include "address.hh"
#include "socket.hh"

#include <cstdlib>
#include <format>
#include <iostream>
#include <span>
#include <string>
#include <utility>

using namespace std;

void get_URL( const string& host, const string& path )
{
  // create a tcp stream socket and connect to server as a client
  auto sock = TCPSocket {};
  sock.connect( Address { host, "http" } );

  // issue and sent the request
  auto req = format( "GET {} HTTP/1.1\r\nHost: {}\r\nConnection: close\r\n\r\n", path, host );
  sock.write( std::move( req ) );

  // read response back
  string resp {};
  while ( !sock.eof() ) { // set `Connection: close` would close the connection and set sock.eof
    sock.read( resp );
    cout << resp;
  }
}

int main( int argc, char* argv[] )
{
  try {
    if ( argc <= 0 ) {
      abort(); // For sticklers: don't try to access argv[0] if argc <= 0.
    }

    auto args = span( argv, argc );

    // The program takes two command-line arguments: the hostname and "path" part of the URL.
    // Print the usage message unless there are these two arguments (plus the program name
    // itself, so arg count = 3 in total).
    if ( argc != 3 ) {
      cerr << "Usage: " << args.front() << " HOST PATH\n";
      cerr << "\tExample: " << args.front() << " stanford.edu /class/cs144\n";
      return EXIT_FAILURE;
    }

    // Get the command-line arguments.
    const string host { args[1] };
    const string path { args[2] };

    // Call the student-written function.
    get_URL( host, path );
  } catch ( const exception& e ) {
    cerr << e.what() << "\n";
    return EXIT_FAILURE;
  }

  return EXIT_SUCCESS;
}

```

# ByteStream
- 共享状态的 `byte oriented stream` 单线程生产者消费者
- 生产者写入 `std::string`, 消费者读取
- 使用 `std::queue<std::string>` 来获取 `push` 参数 `std::string` 的所有权, 避免拷贝
- 返回给消费者 `std::string_view` 避免拷贝

- `byte_stream.hh`
```cpp
#pragma once

#include <cstdint>
#include <queue>
#include <string>
#include <string_view>

class Reader;
class Writer;

class ByteStream
{
public:
  explicit ByteStream( uint64_t capacity );

  // Helper functions (provided) to access the ByteStream's Reader and Writer interfaces
  Reader& reader();
  const Reader& reader() const;
  Writer& writer();
  const Writer& writer() const;

  void set_error() { error_ = true; };       // Signal that the stream suffered an error.
  bool has_error() const { return error_; }; // Has the stream had an error?

protected:
  // Please add any additional state to the ByteStream here, and not to the Writer and Reader interfaces.
  std::queue<std::string> buf; // use queue of string to prevent data copy while push
  uint64_t capacity_;
  uint64_t size_;
  uint64_t bytes_pushed_;
  uint64_t bytes_popped_;
  bool error_ {};
  bool close_;
};

class Writer : public ByteStream
{
public:
  void push( std::string data ); // Push data to stream, but only as much as available capacity allows.
  void close();                  // Signal that the stream has reached its ending. Nothing more will be written.

  bool is_closed() const;              // Has the stream been closed?
  uint64_t available_capacity() const; // How many bytes can be pushed to the stream right now?
  uint64_t bytes_pushed() const;       // Total number of bytes cumulatively pushed to the stream
};

class Reader : public ByteStream
{
public:
  std::string_view peek() const; // Peek at the next bytes in the buffer
  void pop( uint64_t len );      // Remove `len` bytes from the buffer

  bool is_finished() const;        // Is the stream finished (closed and fully popped)?
  uint64_t bytes_buffered() const; // Number of bytes currently buffered (pushed and not popped)
  uint64_t bytes_popped() const;   // Total number of bytes cumulatively popped from stream
};

/*
 * read: A (provided) helper function thats peeks and pops up to `max_len` bytes
 * from a ByteStream Reader into a string;
 */
void read( Reader& reader, uint64_t max_len, std::string& out );

```

- `byte_stream.cc`
```cpp
#include "byte_stream.hh"
#include <string_view>

using namespace std;

ByteStream::ByteStream( uint64_t capacity )
  : buf()
  , capacity_( capacity )
  , size_( 0 )
  , bytes_pushed_( 0 )
  , bytes_popped_( 0 )
  , error_( false )
  , close_( false ) {};

void Writer::push( string data )
{
  // return directly
  if ( this->close_ or data.empty() or !this->available_capacity() ) {
    return;
  }

  auto num_pushed = min( this->available_capacity(), data.size() );
  data.resize( num_pushed );
  this->buf.push( move( data ) );
  this->size_ += num_pushed;
  this->bytes_pushed_ += num_pushed;
}

void Writer::close()
{
  // if ( !this->close_ ) {
  //   this->close_ = true;
  // }
  this->close_ = true;
}

bool Writer::is_closed() const
{
  return this->close_;
}

uint64_t Writer::available_capacity() const
{
  return this->capacity_ - this->size_;
}

uint64_t Writer::bytes_pushed() const
{
  return this->bytes_pushed_;
}

string_view Reader::peek() const
{
  if ( this->size_ == 0 ) {
    return {};
  }
  return string_view { this->buf.front() };
}

void Reader::pop( uint64_t len )
{
  auto num_popped = min( len, this->size_ );
  while ( num_popped > 0 ) {
    auto front_size = this->buf.front().size();
    if ( front_size <= num_popped ) {
      this->size_ -= front_size;
      this->bytes_popped_ += front_size;
      num_popped -= front_size;
      this->buf.pop();
    } else {
      this->buf.front().erase( 0, num_popped );
      this->size_ -= num_popped;
      this->bytes_popped_ += num_popped;
      // num_popped = 0;
      return;
    }
  }
}

bool Reader::is_finished() const
{
  return this->close_ and this->size_ == 0;
}

uint64_t Reader::bytes_buffered() const
{
  return this->size_;
}

uint64_t Reader::bytes_popped() const
{
  return this->bytes_popped_;
}

```