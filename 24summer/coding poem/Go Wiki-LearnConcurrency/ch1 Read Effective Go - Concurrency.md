
# Share by communicating
- 并发编程的难题来自于: 用多个携程对同一个共享变量进行正确访问的需求
- Go 采取了另外的方法来尝试解决这个问题: 不直接在携程之间共享变量, 而是通过 `channel` 传递共享变量
	- `Do not communicate by sharing memory; instead, share memory by communicating.`
- `CSP-Communicating Sequential Processes`


# Goroutines
- 区别于其他并发实体的概念, `goroutine` 只是一个简单的轻量级的函数执行流
- 同时, 通过 `GMP` 模型, `goroutine` 能在多个 `m` 中复用


# Channels
- 无缓冲的 `channel` 也称为同步 `channel`, 可以在数据流通方向强制两个携程同步
```go
c := make(chan int)  // Allocate a channel.
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.
```
- 带有缓冲的 `channel` 则支持异步操作, 可以当作信号量使用
- 这个例子利用缓冲通道做信号量来控制并发携程数, 所有并发执行流必须获取到一个信号量之后才能执行逻辑, 执行完成后释放一个信号量, 这样保证了并发执行逻辑的携程数只有 `MaxOutstanding` 个
```go
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
    sem <- 1    // Wait for active queue to drain.
    process(r)  // May take a long time.
    <-sem       // Done; enable next request to run.
}

func Serve(queue chan *Request) {
    for {
        req := <-queue
        go handle(req)  // Don't wait for handle to finish.
    }
}
```
- 上面的例子虽然控制了通知执行逻辑的携程数, 但是没有控制被创建出来的携程数, 如果请求来的太快, 就可能会导致创建太多携程资源消费殆尽
```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func(req *Request) {
            process(req)
            <-sem
        }(req)
    }
}
```
- 还可以创建长时运行的工作携程来处理任务, 这样也可以避免太多并发携程同时处理逻辑
```go
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // Start handlers
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // Wait to be told to exit.
}
```

# Channels of channels
- 在 Go 中, `channel` 是一等公民, 一个 `channel` 本身也可以作为传递的内容在另一个 `channel` 中传递
- 这个例子是另外一种常见的模式, 将 `resultChan` 包括在请求中, 消费者处理完了就通过 `resultChan` 将请求返回给生产者
```go
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}

func sum(a []int) (s int) {
    for _, v := range a {
        s += v
    }
    return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// Send request
clientRequests <- request
// Wait for response.
fmt.Printf("answer: %d\n", <-request.resultChan)

func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```

# Parallelization
- 并行, 将任务分块并分散到多个 CPU 核心上执行, 执行完成后再合并, 此时可以通过 `channel` 来提示任务的完成
- 这个例子中, 将任务分为 `numCPU` 片子任务, 然后分散到不同 `goroutine` 中并行执行, 
```go
type Vector []float64

// Apply the operation to v[i], v[i+1] ... up to v[n-1].
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
    for ; i < n; i++ {
        v[i] += u.Op(v[i])
    }
    c <- 1    // signal that this piece is done
}

const numCPU = 4 // number of CPU cores

func (v Vector) DoAll(u Vector) {
    c := make(chan int, numCPU)  // Buffering optional but sensible.
    for i := 0; i < numCPU; i++ {
        go v.DoSome(i*len(v)/numCPU, (i+1)*len(v)/numCPU, u, c)
    }
    // Drain the channel.
    for i := 0; i < numCPU; i++ {
        <-c    // wait for one task to complete
    }
    // All done.
}
```

# A leaky buffer
- `channel` 除了在并发场景使用之外, 有缓冲的还可以作为缓冲池, 来缓存一些可以复用的资源
- 这个例子中, 使用 `freeList` 是一个 `Buffer` 缓存池, 如果池子里有东西就复用, 没有就新建
```go
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
    for {
        var b *Buffer
        // Grab a buffer if available; allocate if not.
        select {
        case b = <-freeList:
            // Got one; nothing more to do.
        default:
            // None free, so allocate a new one.
            b = new(Buffer)
        }
        load(b)              // Read next message from the net.
        serverChan <- b      // Send to server.
    }
}

func server() {
    for {
        b := <-serverChan    // Wait for work.
        process(b)
        // Reuse buffer if there's room.
        select {
        case freeList <- b:
            // Buffer on free list; nothing more to do.
        default:
            // Free list full, just carry on.
        }
    }
}
```
