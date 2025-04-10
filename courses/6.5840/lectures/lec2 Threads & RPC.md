# Thread
Go 提供了 goroutine, 一种可以视为轻量级线程的机制

多线程带来的好处有很多: 高性能, 高并发等

多线程也带了很多编程问题:
资源竞争 --> 使用锁 `sync.Mutex`, 
线程同步 --> 使用 `channel`, `sync.Cond`, `sync.WaitGroup`
也就是说
state -- sharing and locks
communication -- channels, WaitGroup

# RPC
RPC 全称 remote procedure call, 远程程序调用
也就是使用网络通信, 在本地指定参数, 调用远程机器中的程序, 再通过网络返回结果
![[682b0545441642b6b3678b288e971ce1~tplv-k3u1fbpfcp-zoom-in-crop-mark_1512_0_0_0.webp]]

example: simple k/v remote storage

```go name=common and main
//
// Common RPC request/reply definitions
//
  
type PutArgs struct {
  Key   string
  Value string
}
  
type PutReply struct {
}
  
type GetArgs struct {
  Key string
}
  
type GetReply struct {
  Value string
}

//
// main
//
  
func main() {
  server()
  
  put("subject", "6.5840")
  fmt.Printf("Put(subject, 6.5840) done\n")
  fmt.Printf("get(subject) -> %s\n", get("subject"))
}
```
定义了在 RPC 过程中 server 和 client 之间数据交换所用载体的结构
以及在 main 函数(client)中远程调用 server 上的例子

```go
//
// Client
//
  
func connect() *rpc.Client {
  client, err := rpc.Dial("tcp", ":1234")
  if err != nil {
    log.Fatal("dialing:", err)
  }
  return client
}
  
func get(key string) string {
  client := connect()
  args := GetArgs{"subject"}
  reply := GetReply{}
  err := client.Call("KV.Get", &args, &reply)
  if err != nil {
    log.Fatal("error:", err)
  }
  client.Close()
  return reply.Value
}
  
func put(key string, val string) {
  client := connect()
  args := PutArgs{"subject", "6.824"}
  reply := PutReply{}
  err := client.Call("KV.Put", &args, &reply)
  if err != nil {
    log.Fatal("error:", err)
  }
  client.Close()
}
```
client 端的程序主要围绕着 get 和 put 两个程序, 流程都类似, 使用 rpc.Dial 连接 server, 然后再使用 client 的参数传递到 server 并调用 server,
RPC 标准库的优势就体现出来了, 只需一行 client.Call 就可以在调用 server 上的程序

```go
//
// Server
//
  
type KV struct {
  mu   sync.Mutex
  data map[string]string
}
  
func server() {
  kv := &KV{data: map[string]string{}}
  rpcs := rpc.NewServer()
  rpcs.Register(kv)
  l, e := net.Listen("tcp", ":1234")
  if e != nil {
    log.Fatal("listen error:", e)
  }
  go func() {
    for {
      conn, err := l.Accept()
      if err == nil {
        go rpcs.ServeConn(conn)
      } else {
        break
      }
    }
    l.Close()
  }()
}
  
func (kv *KV) Get(args *GetArgs, reply *GetReply) error {
  kv.mu.Lock()
  defer kv.mu.Unlock()
  
  reply.Value = kv.data[args.Key]
  
  return nil
}
  
func (kv *KV) Put(args *PutArgs, reply *PutReply) error {
  kv.mu.Lock()
  defer kv.mu.Unlock()
  
  kv.data[args.Key] = args.Value
  
  return nil
}
```
server 也类似, 写好的两个接口 Get 和 Put 都是等待被 client 远程调用的
为了 map 并发安全还加上了锁
server 函数最为关键, 开了一个新 rpc.NewServer, 把 kv 表注册上去

开了一个新 gouroutine 作为闭包, 当作 serve 主循环函数用
在 serve 里, 不断循环阻塞等待 client 的连接, 然后又新开 goroutine 服务这一个连接

RPC YES!
