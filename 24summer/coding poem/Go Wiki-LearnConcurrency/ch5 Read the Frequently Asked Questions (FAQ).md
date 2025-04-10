
# Why build concurrency on the ideas of CSP?
- 之前的并发编程困难是因为 `pthreads` 这些太过于强调 `mutexes, condition variables, memory barriers` 这些底层概念
- `CSP-communicating sequential processes`, 是一种对并发的高级语义支持, 
- `go` 通过将 `channel` 作为一等公民实现了另一种 `CSP`


# Why goroutines instead of threads?
- `goroutine` 的本意是将多个函数执行流复用到 `os threads` 中
- 如果某个 `goroutine` 需要等待 `IO, sys call` 等而阻塞, 那么 `runtime` 就切换其他 `goroutine` 执行流在当前 `thread` 中运行
- `goroutine` 非常轻量, 只是在栈上多开辟几个 KB, 同时是可变的, 按需使用


# Why are map operations not defined to be atomic?
- 没必要
- 如果有并发写的需求就使用 `sync.Map`


# What operations are atomic? What about mutexes?
- `sync, sync/atomic` 提供了许多原子操作
- 对临界资源的简单修改, 例如增长减少等, 尽量使用原子操作
- 虽然是原子的, 但是还是尽量让一个携程一次处理一块数据
- 大型并发程序需要同时利用原子操作和锁和通道等机制


# Why doesn’t my program run faster with more CPUs?
- 取决于程序所解决的问题本质上是否可并发处理
- [Concurrency is not parallelism](https://youtu.be/oV9rvDllKEg)
- [Google I/O 2012 - Go Concurrency Patterns](https://www.youtube.com/watch?v=f6kdp27TYZs)


# How can I control the number of CPUs?
- 环境变量 `GOMAXPROCS` 掌控 Go 程序运行时可以使用的 CPU 核心数量
- 也可以使用 `runtime.GOMAXPROCS()` 函数进行设置


# What happens with closures running as goroutines?
- `go 1.22` 之前经典的错误, 使用
```go
func main() {
    done := make(chan bool)

    values := []string{"a", "b", "c"}
    for _, v := range values {
        go func() {
            fmt.Println(v)
            done <- true
        }()
    }

    // wait for all goroutines to complete before exiting
    for _ = range values {
        <-done
    }
}

for _, v := range values {
    go func(u string) {
        fmt.Println(u)
        done <- true
    }(v)
}

for _, v := range values {
    v := v // create a new 'v'.
    go func() {
        fmt.Println(v)
        done <- true
    }()
}
```
