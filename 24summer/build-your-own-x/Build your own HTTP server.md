[[cURL]]

搭建一个 `HTTP server`

# Bind to a port
```go
package main

import (
	"fmt"
	"net"
	"os"
)

func main() {

	l, err := net.Listen("tcp", "0.0.0.0:4221")
	if err != nil {
		fmt.Println("Failed to bind to port 4221")
		os.Exit(1)
	}

	_, err = l.Accept()
	if err != nil {
		fmt.Println("Error accepting connection: ", err.Error())
		os.Exit(1)
	}
}

```
- 设置服务器向外提供服务所用的端口
- `net.Listen("tcp", "0.0.0.0:4221")`
	- `tcp, 4221` 表示监听 `tcp` 连接
	- `0.0.0.0` 表示监听所有网络接口, 也就是监听来自所有 IP 的 `tcp` 连接
- `conn, err := l.Accept()` 表示接收一个 `tcp` 连接, 这个方法是阻塞的