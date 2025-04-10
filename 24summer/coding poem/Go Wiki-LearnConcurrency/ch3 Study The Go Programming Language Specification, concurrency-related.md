# Go statements
- `go` 关键字用来启动一个新的 `goroutine` 来执行一个函数逻辑流
- 表达式必须是函数或方法调用
- 函数如果有返回值会默认被丢弃
```go
go func_name()
```

# Channel types
- 给并发执行的函数提供了一种相互交流机制
- 有三种类型的 `channel`, 一般声明时会声明读写 `channel`, 然后在参数传递时, 根据函数的需求, 传递只读或只写的 `channel`
- 使用内置 `make()` 函数进行初始化, 如果没有初始化而只是声明, 此时是 `nil`, 无法读写
- 使用内置 `close()` 函数可以关闭一个通道, 此后无法写, 但是可以读
- 使用内置 `len(), cap()` 函数
	- 无缓冲: `len()` 返回等待接受通道的 `goroutine` 数量, `cap()` 返回 0
	- 有缓冲: `len()` 返回缓冲区内元素个数, `cap()` 返回缓冲区大小
```go
chan T          // can be used to send and receive values of type T
chan<- float64  // can only be used to send float64s
<-chan int      // can only be used to receive ints

ch := make(chan int, 0)
```

# Send statements
- 将数据发送到 `channel` 中, 可能阻塞
```go
ch := make(chan int)
ch <- 3  // send value 3 to channel ch
```

# Receive operator
- 从 `channel` 中接受数据, 可能阻塞
```go
v1 := <-ch
v2 = <-ch
f(<-ch)
<-strobe  // wait until clock pulse and discard received value

// yields an additional untyped boolean result reporting whether the communication succeeded
x, ok = <-ch
x, ok := <-ch
var x, ok = <-ch
var x, ok T = <-ch
```

# Select statements
- 从一组通道接受/发送操作中, 选择出第一个可执行的进行执行
- 如果没有可执行的就执行 `default` 分支
- 如果没有 `default` 分支就等待第一个分支可执行
```go
var a []int
var c, c1, c2, c3, c4 chan int
var i1, i2 int
select {
case i1 = <-c1:
	print("received ", i1, " from c1\n")
case c2 <- i2:
	print("sent ", i2, " to c2\n")
case i3, ok := (<-c3):  // same as: i3, ok := <-c3
	if ok {
		print("received ", i3, " from c3\n")
	} else {
		print("c3 is closed\n")
	}
case a[f()] = <-c4:
	// same as:
	// case t := <-c4
	//	a[f()] = t
default:
	print("no communication\n")
}

for {  // send random sequence of bits to c
	select {
	case c <- 0:  // note: no statement, no fallthrough, no folding of cases
	case c <- 1:
	}
}

select {}  // block forever
```
