
# Channel synchronization
- `channel` 也可以用来做同步
```go
package main

import (
    "fmt"
    "time"
)

func worker(done chan bool) {
    fmt.Print("working...")
    time.Sleep(time.Second)
    fmt.Println("done")

    done <- true
}

func main() {

    done := make(chan bool, 1)
    go worker(done)

    <-done
}
```

# Channel directions
- 可以为 `channel` 限定数据流通方向, 让一个 `channel` 只读或这只写
- 通常用于将 `channel` 作为函数参数传递时使用
```go
package main

import "fmt"

// write-only channel
func ping(pings chan<- string, msg string) {
    pings <- msg
}

// read-only channel
func pong(pings <-chan string, pongs chan<- string) {
    msg := <-pings
    pongs <- msg
}

func main() {
    pings := make(chan string, 1)
    pongs := make(chan string, 1)
    ping(pings, "passed message")
    pong(pings, pongs)
    fmt.Println(<-pongs)
}
```


# Timeouts
- 利用 `select + time.After()`, 可以非常简单地编写超时处理
- `time.After()` 返回一个 `<-chan time.Time`, 只读的通道, 如果这个通道中读出数据了, 说明设置的 `timeout` 到时了, 可以进行超时处理了
```go
package main

import (
    "fmt"
    "time"
)

func main() {

    c1 := make(chan string, 1)
    go func() {
        time.Sleep(2 * time.Second)
        c1 <- "result 1"
    }()

    select {
    case res := <-c1:
        fmt.Println(res)
    case <-time.After(1 * time.Second):
        fmt.Println("timeout 1")
    }

    c2 := make(chan string, 1)
    go func() {
        time.Sleep(2 * time.Second)
        c2 <- "result 2"
    }()
    select {
    case res := <-c2:
        fmt.Println(res)
    case <-time.After(3 * time.Second):
        fmt.Println("timeout 2")
    }
}
```


# Non-Blocking Channel Operations
- 非阻塞的 `channel` 操作
- 在不希望将当前携程阻塞的场景中, 可以利用 `select + default` 避免阻塞
```go
package main

import "fmt"

func main() {
    messages := make(chan string)
    signals := make(chan bool)

    select {
    case msg := <-messages:
        fmt.Println("received message", msg)
    default:
        fmt.Println("no message received")
    }

    msg := "hi"
    select {
    case messages <- msg:
        fmt.Println("sent message", msg)
    default:
        fmt.Println("no message sent")
    }

    select {
    case msg := <-messages:
        fmt.Println("received message", msg)
    case sig := <-signals:
        fmt.Println("received signal", sig)
    default:
        fmt.Println("no activity")
    }
}
```


# Closing Channels
- 使用内建 `close()` 函数将一个 `channel` 关闭代表着不再往 `channel` 中发送数据
- 但是还可以接收数据 (如果有)
- 通常跟双返回值的读取 `channel` 表达式结合 `j, ok := <-jobs`
- 如果 `ok == false`, 说明这个 `channel` 已经关闭, 此时 `j` 是零值
- 向一个关闭的 `channel` 再写数据会造成 `panic`
```go
package main

import "fmt"

func main() {
    jobs := make(chan int, 5)
    done := make(chan bool)

    go func() {
        for {
            j, more := <-jobs
            if more {
                fmt.Println("received job", j)
            } else {
                fmt.Println("received all jobs")
                done <- true
                return
            }
        }
    }()

    for j := 1; j <= 3; j++ {
        jobs <- j
        fmt.Println("sent job", j)
    }
    close(jobs)
    fmt.Println("sent all jobs")

    <-done

    _, ok := <-jobs
    fmt.Println("received more jobs:", ok)
}
```

# Range over Channels
- `channel` 关闭后, `for ... range ch` 自动退出
- 没数据时行为类似无限读 `channel` 将会阻塞等待数据
```go
package main

import "fmt"

func main() {

    queue := make(chan string, 2)
    queue <- "one"
    queue <- "two"
    close(queue)

    for elem := range queue {
        fmt.Println(elem)
    }
}
```


# Timers
- `time.Timer` 是内置的一个定时提醒工具, 通常用于做定时任务
- 到时之后, `runtime` 通过向 `Timer.C` 中写数据提醒
- 还可以通过 `Timer.Stop()` 手动停止定时任务
```go
// The Timer type represents a single event.
// When the Timer expires, the current time will be sent on C,
// unless the Timer was created by AfterFunc.
// A Timer must be created with NewTimer or AfterFunc.
type Timer struct {
	C <-chan Time
	r runtimeTimer
}

package main

import (
    "fmt"
    "time"
)

func main() {

    timer1 := time.NewTimer(2 * time.Second)

    <-timer1.C
    fmt.Println("Timer 1 fired")

    timer2 := time.NewTimer(time.Second)
    go func() {
        <-timer2.C
        fmt.Println("Timer 2 fired")
    }()
    stop2 := timer2.Stop()
    if stop2 {
        fmt.Println("Timer 2 stopped")
    }

    time.Sleep(2 * time.Second)
}
```


# Tickers
- 跟 `time.Timer` 类似, `time.Ticker` 可以做重复的定时任务, 每次时间间隔一到就往 `Ticker.C` 中写数据
- 同样可以通过 `Ticker.Stop()` 停止
```go
// A Ticker holds a channel that delivers “ticks” of a clock
// at intervals.
type Ticker struct {
	C <-chan Time // The channel on which the ticks are delivered.
	r runtimeTimer
}

package main

import (
    "fmt"
    "time"
)

func main() {

    ticker := time.NewTicker(500 * time.Millisecond)
    done := make(chan bool)

    go func() {
        for {
            select {
            case <-done:
                return
            case t := <-ticker.C:
                fmt.Println("Tick at", t)
            }
        }
    }()

    time.Sleep(1600 * time.Millisecond)
    ticker.Stop()
    done <- true
    fmt.Println("Ticker stopped")
}
```


# Worker Pools
- 复用 `worker goroutine`
- 通过 `channel` 来传递 `job + result`
```go
package main

import (
    "fmt"
    "time"
)

func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Println("worker", id, "started  job", j)
        time.Sleep(time.Second)
        fmt.Println("worker", id, "finished job", j)
        results <- j * 2
    }
}

func main() {

    const numJobs = 5
    jobs := make(chan int, numJobs)
    results := make(chan int, numJobs)

    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }

    for j := 1; j <= numJobs; j++ {
        jobs <- j
    }
    close(jobs)

    for a := 1; a <= numJobs; a++ {
        <-results
    }
}
```

# WaitGroups
- `wg`, 没啥好说
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int) {
    fmt.Printf("Worker %d starting\n", id)

    time.Sleep(time.Second)
    fmt.Printf("Worker %d done\n", id)
}

func main() {

    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1)

        go func() {
            defer wg.Done()
            worker(i)
        }()
    }

    wg.Wait()

}
```


# Atomic Counters
- 原子操作
```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {

    var ops atomic.Uint64

    var wg sync.WaitGroup

    for i := 0; i < 50; i++ {
        wg.Add(1)

        go func() {
            for c := 0; c < 1000; c++ {

                ops.Add(1)
            }

            wg.Done()
        }()
    }

    wg.Wait()

    fmt.Println("ops:", ops.Load())
}
```

# Mutexes
- 锁, 保护并发临界区
- 相对于原子操作只能安全操作简单的数据类型, 使用锁可以保护更复杂的临界区
```go
package main

import (
    "fmt"
    "sync"
)

type Container struct {
    mu       sync.Mutex
    counters map[string]int
}

func (c *Container) inc(name string) {

    c.mu.Lock()
    defer c.mu.Unlock()
    c.counters[name]++
}

func main() {
    c := Container{

        counters: map[string]int{"a": 0, "b": 0},
    }

    var wg sync.WaitGroup

    doIncrement := func(name string, n int) {
        for i := 0; i < n; i++ {
            c.inc(name)
        }
        wg.Done()
    }

    wg.Add(3)
    go doIncrement("a", 10000)
    go doIncrement("a", 10000)
    go doIncrement("b", 10000)

    wg.Wait()
    fmt.Println(c.counters)
}
```


# Stateful Goroutines
- 上面的例子中, 使用 `mutex` 显示地锁住了临界区, 让多个 `goroutines` 实际上共享了内存, 实际上是不太符合 go 的思想的
- 通过 `channel`, 传递数据的所有权, 让每一块数据在同一时刻恰好只有一个 `goroutine` 所有
- 这个例子就巧妙地实现了这个模式, 实现了一个并发安全的 `map`
- 对外提供 `read, write` 接口
	- 一旦有请求, 就将请求写进 `reads, writes` 这两个通道中
	- 然后在持有 `map` 的携程中, 通过 `select`, 每次只处理一个请求, 也就通过传递数据, 每次只让一个 `goroutine` 可以访问一块数据 (`state, read, write`)

```go
package main

import (
    "fmt"
    "math/rand"
    "sync/atomic"
    "time"
)

type readOp struct {
    key  int
    resp chan int
}
type writeOp struct {
    key  int
    val  int
    resp chan bool
}

func main() {

    var readOps uint64
    var writeOps uint64

    reads := make(chan readOp)
    writes := make(chan writeOp)

    go func() {
        var state = make(map[int]int)
        for {
            select {
            case read := <-reads:
                read.resp <- state[read.key]
            case write := <-writes:
                state[write.key] = write.val
                write.resp <- true
            }
        }
    }()

    for r := 0; r < 100; r++ {
        go func() {
            for {
                read := readOp{
                    key:  rand.Intn(5),
                    resp: make(chan int)}
                reads <- read
                <-read.resp
                atomic.AddUint64(&readOps, 1)
                time.Sleep(time.Millisecond)
            }
        }()
    }

    for w := 0; w < 10; w++ {
        go func() {
            for {
                write := writeOp{
                    key:  rand.Intn(5),
                    val:  rand.Intn(100),
                    resp: make(chan bool)}
                writes <- write
                <-write.resp
                atomic.AddUint64(&writeOps, 1)
                time.Sleep(time.Millisecond)
            }
        }()
    }

    time.Sleep(time.Second)

    readOpsFinal := atomic.LoadUint64(&readOps)
    fmt.Println("readOps:", readOpsFinal)
    writeOpsFinal := atomic.LoadUint64(&writeOps)
    fmt.Println("writeOps:", writeOpsFinal)
}
```
