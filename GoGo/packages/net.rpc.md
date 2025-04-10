RPC (Remote Procedure Call)
![[682b0545441642b6b3678b288e971ce1~tplv-k3u1fbpfcp-zoom-in-crop-mark_1512_0_0_0.webp]]

就像它的名字一样, 起到的作用就是使用网络远程调用其他机器上某个程序的某个函数

demo:
server.go
![[code6.png]]
`rpc.Register` 注册了 `*Arith` 的两个方法
`rpc` 库对注册的方法的限制是: 函数签名必须满足 `func (t *T) MethodName(args ArgsType, reply *ReplyType) error`
- 方法必须是导出的 (大写开头)
- 方法必须接收两个参数 (导出的或内置类型, 第二个参数为返回给客户端的响应, 必须为指针, 需要修改里面的内容)
- 必须返回 `error`, 非 `nil` 表示调用出错

`rpc.HandleHTTP` 注册路由
`http.ListenAndServe` 开始监听


client.go
![[code7.png]]
`rpc.DialHTTP("tcp", ":1234")` 连接到 server 的监听地址, 返回的是 `*rpc.Client`, 可以使用这个返回值的 `Call()` 方法调用 server 中的方法


rpc 源码:
![[code8.png]]
server 中, `rpc.HandleHTTP` 原来调用了 `http.Handle` 把注册路由,
然后再调用 `http.ListenAndServe` 开始监听

![[code9.png]]
`rpc.Register` 则是将这个类(rcvr, demo 中的 arith 的所有符合标准的方法注册到 rpc 默认 server 中)


在上面的 demo 中, client 使用了同步的调用方式, 一直在等待服务端的响应或出错, 等待过程中无法处理其他任务, 效率较低
因此可以使用异步的调用方式

async_client.go
![[code10.png]]
异步调用使用 `client.Go()` 方法, 参数与同步调用基本一致, 只是返回的是 `*rpc.Call`
```go
// Call represents an active RPC.
type Call struct {
  ServiceMethod string     // The name of the service and method to call.
  Args          any        // The argument to the function (*struct).
  Reply         any        // The reply from the function (*struct).
  Error         error      // After completion, the error status.
  Done          chan *Call // Receives *Call when Go is complete.
}
```
通过监听信道 `Done` 是否传入值来判断调用是否完成


自定义编码格式:
在默认情况下, client-server 之间的数据使用 `gob` 编码, 但是可以实现 `rpc.ServerCodec` 接口, 从而使用其他格式编码
```go
// A ServerCodec implements reading of RPC requests and writing of
// RPC responses for the server side of an RPC session.
// The server calls ReadRequestHeader and ReadRequestBody in pairs
// to read requests from the connection, and it calls WriteResponse to
// write a response back. The server calls Close when finished with the
// connection. ReadRequestBody may be called with a nil
// argument to force the body of the request to be read and discarded.
// See NewClient's comment for information about concurrent access.
type ServerCodec interface {
  ReadRequestHeader(*Request) error
  ReadRequestBody(any) error
  WriteResponse(*Response, any) error
  
  // Close can be called multiple times and must be idempotent.
  Close() error
}

type gobServerCodec struct {
  rwc    io.ReadWriteCloser
  dec    *gob.Decoder
  enc    *gob.Encoder
  encBuf *bufio.Writer
  closed bool
}
  
func (c *gobServerCodec) ReadRequestHeader(r *Request) error {
  return c.dec.Decode(r)
}
  
func (c *gobServerCodec) ReadRequestBody(body any) error {
  return c.dec.Decode(body)
}
  
func (c *gobServerCodec) WriteResponse(r *Response, body any) (err error) {
	...
}
  
func (c *gobServerCodec) Close() error {
	...
}
```


自定义服务器:
同样类似 `net/http`, 直接在程序中调用 `rpc.Register`, `rpc.HandleHttp` 都是使用默认的 DefaultServer
因此是不太推荐的, 一般情况是使用自己创建的 server
