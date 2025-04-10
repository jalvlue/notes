- 搭一个 redis, 包括客户端和服务端


# Bind to a port
```go
l, err := net.Listen("tcp", "0.0.0.0:6379")
conn, err := l.Accept()
// ..handle conn
```
- 监听一个端口
- 等待客户端连接
- 处理连接
