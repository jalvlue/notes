# WaitGroup
`sync.WaitGroup` 本质上就是一个计数器, 里面保存的是需要等待的 `goroutine` 的数量
```go
var wg = sync.WaitGroup{}

// add waiting(in main routine)
wg.Add(<int>)

// decrease waiting(in sub routine)
wg.Done()

// begin waiting(in main routine)
wg.Wait
```
```go
package main
  
import (
  "fmt"
  "sync"
  "time"
)
  
var wg = sync.WaitGroup{}
  
func main() {

  wg.Add(2)
  start := time.Now()
  go doSomething()
  go doSomethingElse()
  wg.Wait()
  
  fmt.Println("\n\nI guess I'm done")
  elapsed := time.Since(start)
  fmt.Printf("Processes took %s\n", elapsed)
}
  
func doSomething() {
  time.Sleep(time.Second * 2)
  fmt.Println("\nI've done something")
  wg.Done()
}
  
func doSomethingElse() {
  time.Sleep(time.Second * 2)
  fmt.Println("I've done something else")
  wg.Done()
}
```

如果不使用全局变量时, 需要使用参数传递方式, 此时需要传递指针
```go
wg := sync.WaitGroup{}
wg.Add(1)
go someFunc(&wg)
...
```

```ad-summary
WaitGroup 就是一个计数器, 主要作用就是实现同步, 把 sub-routine 都加进 wg中, 然后 main-routine 等待这些子线程完成后才继续运行
```

# Channel
> Don't communicating by shared data, sharing data by communication.

`channel` 就是用来进行进程间通信的工具

##### simple channel
最简单的 `channel` 只有一个功能就是装入一个数据, 再将数据发出
```go
ch := make(chan string)

ch <- myData // blocking
myVar <- ch // blocking

close(ch)
```
`simple channel` 中, 装入和发出都会被阻塞, 只有装入和发出都准备好时
数据的流通才会在两个进程间进行

##### buffered channel
与 `simple channel` 类似, 但是带有一个容量, 
在容量未满之前, 装入数据不会被阻塞, 发出数据按照装入队列顺序
```go
ch := make(string, 3)
ch <- "hello"
ch <- "from"
ch <- "someone"

msg := <-c
fmt.Print(msg)

msg = <-c
fmt.Print(msg)

msg = <-c
fmt.Print(msg)
```

在使用 `range` 遍历一个 `channel` 时, 必须先使用 `ch.Close()` 关闭这个 `channel`

##### select
在多个 `channel` 同时存在时, 如果需要监听多个 `channel` 则可以使用 `select` 关键字, 
`select` 会选择未阻塞的 `channel` 分支并执行对应操作(多个则随机选择一个)
```go
  c1 := make(chan string)
  c2 := make(chan string)
  c3 := make(chan string)
  
  go func() {
    for {
      time.Sleep(time.Second)
      c1 <- "Sending every 1 second"
    }
  }()
  go func() {
    for {
      time.Sleep(time.Second * 4)
      c2 <- "Sending every 4 sec"
    }
  }()
  go func() {
    for {
      time.Sleep(time.Second * 10)
      c3 <- "We're done"
    }
  }()
  
  for { // infinite for loop  This is the operator - listening for activity on all channels.
    // This is a clever way to get around the blocking nature when you try to read a channel
    select {
    case msg := <-c1:
      fmt.Println(msg)
    case msg := <-c2:
      fmt.Println(msg + " Something cool happened")
    case msg := <-c3:
      fmt.Println(msg)
      os.Exit(0)
  
    }
  }
```


# mutex
`mutex` 即互斥锁, 保护上锁的变量在同一时间内只有一个 `goroutine` 可以读写
```go
mu := sync.Mutex{}
mu.Lock()
mu.Unlock()
```
```go
package main
  
import (
  "fmt"
  "sync"
)
  
var (
  wg              sync.WaitGroup
  mutex                 = sync.Mutex{}
  widgetInventory int32 = 1000 //Package-level variable will avoid all the pointers

)
  
func main() {
  fmt.Println("Starting inventory count = ", widgetInventory)
  wg.Add(2)
  go makeSales()
  go newPurchases()
  wg.Wait()
  fmt.Println("Ending inventory count = ", widgetInventory)
}
  
func makeSales() { // 1000000 widgets sold
  for i := 0; i < 3000; i++ {
    mutex.Lock()
    widgetInventory -= 100
    fmt.Println(widgetInventory)
    mutex.Unlock()
  }
  
  wg.Done()
}
  
func newPurchases() { // 1000000 widgets purchased
  for i := 0; i < 3000; i++ {
    mutex.Lock()
    widgetInventory += 100
    fmt.Println(widgetInventory)
    mutex.Unlock()
  }
  wg.Done()
}
```
在这个例子中, 变量互斥访问是实现了, 但是竞争是随机的, 因此可能卖出不存在的商品(库存为负数)


# condition
`condition` 也就是条件变量, 就跟操作系统中的概念类似, 
在锁的互斥选择中加入一个条件, 满足条件的获得锁, 不满足条件的等待锁
```go
var mu = sync.Mutex{}
var condition = sync.NewCond(&mu)

condition.Signal()
condition.Broadcast()
condition.Wait()
```
```go
package main
  
import (
  "fmt"
  "sync"
)
  
var (
  wg              sync.WaitGroup
  mutex                 = sync.Mutex{}
  widgetInventory int32 = 1000 //Package-level variable will avoid all the pointers
  newPurchase           = sync.NewCond(&mutex)
)
  
func main() {
  fmt.Println("Starting inventory count = ", widgetInventory)
  wg.Add(2)
  go makeSales()
  go newPurchases()
  wg.Wait()
  fmt.Println("Ending inventory count = ", widgetInventory)
}
  
func makeSales() { // 1000000 widgets sold
  for i := 0; i < 3000; i++ {
    mutex.Lock()
    if widgetInventory-100 < 0 {
      newPurchase.Wait()
    }
    widgetInventory -= 100
    fmt.Println(widgetInventory)
    mutex.Unlock()
  }
  
  wg.Done()
}
  
func newPurchases() { // 1000000 widgets purchased
  for i := 0; i < 3000; i++ {
    mutex.Lock()
    widgetInventory += 100
    fmt.Println(widgetInventory)
    newPurchase.Signal()
    mutex.Unlock()
  }
  wg.Done()
}
```
在 `makeSales` 中加入检测, 库存为负数时就调用 `newPurchase.Wait()` 阻塞当前线程
在进货函数中加入 `newPurchase.Signal()`, 唤醒阻塞的线程(如果有)

# atomic
`atomic` 是一个包, 里面提供了很多原子操作(实际上就是对 mutex 封装了)
```go
var inventory int32 = 1000
atomic.AddInt32(&inventory, -100)
```


```ad-summary
Sharing data by communication.

所有这些waitgroup, channel, mutex, condition, atomic都是为了进程间通信存在的, 想好通信模型, 使用合适的同步方式, 就可以写出好的多线程代码
```
