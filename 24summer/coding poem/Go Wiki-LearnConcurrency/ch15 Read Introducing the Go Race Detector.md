- 用于数据竞争检测
- 现在已经是 Go 语言的一部分
- 在运行程序时使用 `-race` 标签即可进行竞争检测

```sh
$ go test -race mypkg    // test the package
$ go run -race mysrc.go  // compile and run the program
$ go build -race mycmd   // build the command
$ go install -race mypkg // install the package
```