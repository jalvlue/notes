Go 中标准库 `net/http` 已经提供了功能强大的 web 编程功能
掌握 web 基本知识和 `net/http` 包后, 上手其他 web 框架也非常迅速 (go web 框架比如 gin, beego 等基本上也是对 `net/http` 的封装, 提供易于使用的功能)

# Web 基础知识
Http 请求
- 请求行 (request-line)
- 零个或多个首部 (header)
- 一个空行
- 一个可选的请求主体 (body)

请求行最为重要, 格式一般为:
```
Method Path Version

例如:
GET localhost:8080/user/info.html
```

Http 响应
- 响应行 (response-line)
- 零个或多个首部 (header)
- 一个空行
- 一个可选的响应主体 (body)

响应行格式为:
```
StatusCode Description
```
http 将状态码分为了 5 大类，`1XX/2XX/3XX/4XX/5XX`。

- `1XX`：情报状态码，又叫做信息状态码。服务器通过这类状态码告知客户端，自己已经收到了客户端发送的请求。几个常见的状态码如下：
    - `100 Continue`：表示服务器到目前为止收到的内容都正常，客户端应该继续请求。如果已经完成，则忽略它。
    - `101 Switching Protocol`：这个状态码是响应客户端的 `Upgrade` 首部发送的，并且指示服务器也正在切换的协议。如客户端请求切换协议，服务器将协议切换至 Websocket，就会发送该状态码给客户端，并且在 `Upgrade` 首部中填上 Websocket。
- `2XX`：成功状态码。表示服务器已经收到了客户端的请求，并成功对请求进行了处理。几个常见的状态码如下：
    - `200 OK`：这最常见的状态码了，表示请求成功。
    - `201 Created`：请求已成功，并因此创建了一个新的资源。
- `3XX`：重定向状态码。服务器收到了请求，但是为了完整地处理该请求，客户端还需要执行指定的动作。一般用于 URL 重定向。几个常见的状态码如下：
    - `300 Multiple Choice`：被请求的资源有一系列可供选择的回馈信息，每个都有自己特定的地址。用户或浏览器可自行选择一个地址进行重定向。
    - `302 Moved Permanently`：被请求的资源已经永久移动到新的位置了。
- `4XX`：客户端错误状态码。表示客户端发送的请求中有错误，如格式不对。常见的状态码如下：
    - `404 Not Found`：最常见的状态码了，表示页面不存在。
    - `405 Method Not Allowed`：请求的方法不允许。
- `5XX`：服务器错误状态码。表示服务器由于某些原因无法正确处理请求。常见的状态码如下：
    - `500 Internal Server Error`：服务器遇到了不知道如何处理的情况。
    - `501 Not Implemented`：此请求方法不被服务器支持。



# 第一个 web 程序
![[code5.png]]
这时一个最简单的 go web 服务器
`http.HandleFunc` 的作用是将 `hello` 函数注册到 `/hello` 目录上
`hello` 函数是一个处理器 (handler), 接收两个参数: 
- `w http.ResponseWriter` 是发送给客户端的响应
- `r *http.Request` 是客户端发送的消息
`http.ListenAndServe` 将在本地 `8080` 端口上监听请求, 
对 `/hello` 路径的请求会交由 `hello` 函数处理
```ad-summary
handler -> register -> ListenAndServe
```

# 详细程序结构
![[Pasted image 20231108105052.png]]
一个典型的 Go web 程序结构

上文中提到的 `http.HandleFunc` 事实上是使用了默认的多路复用器
```go
// src/net/http/server.go

// DefaultServeMux is the default ServeMux used by Serve.
var DefaultServeMux = &defaultServeMux

var defaultServeMux ServeMux

func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}

```
将 `hello` 函数, 也就是 `handler` 注册到了默认多路复用器中

`http.ListenAndServe` 则创建一个默认服务器, 并由这个服务器处理服务
```go
// src/net/http/server.go
func ListenAndServe(addr string, handler Handler) {
    server := &Server{Addr: addr, Handler: handler}
    return server.ListenAndServe()
}

type serverHandler struct {
	srv *Server
}

func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
	handler := sh.srv.Handler
	if handler == nil {
		handler = DefaultServeMux
	}
	handler.ServeHTTP(rw, req)
}

```
同样的, `sh.srv.Handler` 为 `nil` 时, 也就是在 hello 程序中, 使用 `http.ListenAndServe(":8080", nil)`
Server 会使用默认多路复用器

```ad-summary
server收到的每个请求都会调用多路复用器, 并根据URL查找注册的处理器
然后将请求交给处理器处理

默认的多路复用器是一个全局变量, 标准库内创建的server也不太方便
实际生产中不建议使用, 应该自己创建多路复用器
```

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func hello(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello World")
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", hello)

	server := &http.Server{
		Addr:           ":8080",
		Handler:        mux,
		ReadTimeout:    1 * time.Second,
		WriteTimeout:   1 * time.Second,
	}

	if err := server.ListenAndServe(); err != nil {
		log.Fatal(err)
	}
}
```


除了直接用 `ServerMux.HandleFunc()` 直接将函数 (`func (w ResponseWrite, r *Request)` 签名的)注册到多路复用器中
还可以使用 `ServerMux.Hanlde()` 注册处理器, 这种做法需要自定义一个处理器, 并实现 `Handler` 接口
```go
// src/net/http/server.go
type Handler interface {
    func ServeHTTP(w Response.Writer, r *Request)
}

// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers. If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler that calls f.
type HandlerFunc func(ResponseWriter, *Request)
  
// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
  f(w, r)
}

// HandleFunc registers the handler function for the given pattern.
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
  if handler == nil {
    panic("http: nil handler")
  }
  mux.Handle(pattern, HandlerFunc(handler))
}

// Handle registers the handler for the given pattern.
// If a handler already exists for pattern, Handle panics.
func (mux *ServeMux) Handle(pattern string, handler Handler) {
  mux.mu.Lock()
  defer mux.mu.Unlock()
  
  if pattern == "" {
    panic("http: invalid pattern")
  }
  if handler == nil {
    panic("http: nil handler")
  }
  if _, exist := mux.m[pattern]; exist {
    panic("http: multiple registrations for " + pattern)
  }
  
  if mux.m == nil {
    mux.m = make(map[string]muxEntry)
  }
  e := muxEntry{h: handler, pattern: pattern}
  mux.m[pattern] = e
  if pattern[len(pattern)-1] == '/' {
    mux.es = appendSorted(mux.es, e)
  }
  
  if pattern[0] != '/' {
    mux.hosts = true
  }
}
```


例子:
```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

type GreetingHandler struct {
    Language string
}

func (h GreetingHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "%s", h.Language)
}

func main() {
    mux := http.NewServeMux()
    mux.Handle("/chinese", GreetingHandler{Language: "你好"})
    mux.Handle("/english", GreetingHandler{Language: "Hello"})
    
    server := &http.Server {
        Addr:   ":8080",
        Handler: mux,
    }
    
    if err := server.ListenAndServe(); err != nil {
        log.Fatal(err)
    }
}
```

```ad-summary
无论是使用HandleFunc还是使用Handle注册, 都是一样的, 最终都会到ServerMux.Handle(pattern string, handler Handler)进去注册

因为使用HandleFunc时, 这个函数(例如hello)就是一个底层为func (ResponseWriter, *Request)的函数, 也就是一个HandlerFunc

而HandlerFunc是实现Handler接口的(有ServeHTTP方法), 
因此可以作为一个Handler, 使用ServerMux.Hanlde()进行注册
```


# 应用例子
