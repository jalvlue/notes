项目开始前都需要使用
```shell
go mod init <remote-repo>
```
指令进行初始化, 
初始化后会在项目根目录下生成 `go.mod` 文件, 是 `go` 进行依赖管理的工具


项目根目录下的 `.env` 文件通常用来描述项目的一些配置
```.env
PORT=8080
```
例如 `PORT` 表示这个 `web server` 运行在电脑的哪个端口

有了 `.env` 文件后, 在程序中可以使用 `os` 包内的 `Getenv` 函数使用 `.env` 文件中的键获得配置信息
```go
import "os"
import "fmt"

func main() {
	portString := os.Getenv("PORT")
	fmt.Println(portString)
}
```
需要注意的是, `os.Getenv(key)` 函数在当前 shell 中寻找名称为 `key` 的 shell 变量, 并返回它的值,

因此需要使用一个包将 `.env` 中的配置在程序运行时自动导入到 shell 中,
包函数是 `godotenv.Load`
```go
import "os"
import "fmt"
import "github.com/joho/godotenv"

func main() {
	godotenv.Load(".env")

	portString := os.Getenv("PORT")
	fmt.Println(portString)
}
```

计网常识:
1. Http request: `client` 通过 `http` 协议, 向 `server` 做出一些访问, 并让 `server` 做出回应
2. REST apis: