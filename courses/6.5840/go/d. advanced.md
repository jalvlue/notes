# higher order function
Go 支持函数作为参数或者返回值(类似 python 高阶函数)
```go
package main

import (
	"fmt"
	"math"
)

func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}

```
compute 就是一个高阶函数, 接收一个函数作为参数

当返回值是一个函数时, 可以说返回的是一个闭包  
对于这个闭包, 闭包之外的函数体都是闭包的引用, 闭包可以访问修改这些引用, 每个闭包都绑定自己的这些引用

```go
package main

import (
	"fmt"
)

func adder() func(int) int {
	sum := 0
	return func(i int) int {
		sum += i
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}

0 0
1 -2
3 -6
6 -12
10 -20
15 -30
21 -42
28 -56
36 -72
45 -90
```
在这个例子中, pos, neg 就是函数 adder 返回的两个闭包, 他们各自绑定自己的 sum

实现 fibonacci 闭包:
```go
package main

import (
	"fmt"
)

func fib() func() int {
	prev := -1
	curr := 1
	return func() int {
		tmp := prev + curr
		prev = curr
		curr = tmp
		return curr
	}
}

func main() {
	f := fib()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}

0
1
1
2
3
5
8
13
21
34
```


# defer
defer 关键字用于推迟一个函数的执行(将这个函数的执行推迟到调用者返回前执行)
```go
func fileCopy(destName, srcName string) (...) {
	src, err := os.open(srcName)
	if err != nil {
		return
	}

	defer src.close()

	...
	if {
		...
		return ...
	}

	if {
		...
		return ...
	}
}
```
在这个例子中, 有多个 `return` 语句, 如果在每个 return 语句之前都加上关闭文件的操作就很繁杂了,
使用 `defer` 关键字可以方便地解决这个问题


# local development
Go 的程序都是由包(package)构成的, 每一个 `.go` 源文件都在文件最开始使用 `package` 关键字声明了这个源文件所在的包
`main` 是一个特殊的包, 包含了程序的 main 函数, 是程序的入口
其他的包则属于 `Library`, 提供一些可导出的函数或变量, 供其他文件导入使用
```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	fmt.Println("My favorite number is", rand.Intn(10))
}

```
例如 `fmt`, `math/rand` 就是两个 Library, 导出了 fmt.Println 和 rand.Intn

所有的 packages 又构成了一个 module, 特指同时 release 的所有 packages(也就是一个版本)
可以简单理解为一个 module 就是 Go 项目的一个版本
go.mod 文件会说明 module 的一些信息:
```go
module github.com/bootdotdev/exampleproject

go 1.20

require github.com/google/examplepackage v1.3.0
```
同时, module 可以方便地进行依赖管理


# concurrency & channels
关键字 `go` 可以让新建一个 `goroutine` (可以理解为是一个轻量级线程), 并行地运行一个函数
语法:
```go
go doSomething()
```

`goroutine` 之间通过 `channel` 进行数据交换
`channel` 是一种有类型的, 线程安全的队列, 使用 `make` 关键字进行创建
```go
ch := make(chan int)
ch <- 66
v := <-ch
```
跟操作系统中的读者写者问题有些类似
如果 `channel` 为空, 则读出操作会被 `block`, 直到其他 `goroutine` 向 `channel` 内传入东西
类似的, `channel` 满了, 则写入操作会被 `block`
需要注意的是, 直接使用 `make(chan T)` 创建的 `channel` 容量为 1, 因此需要一读一写
因此 `channel` 操作可能会导致死锁,

除了用来交换信息外, channel 也可以用来作为一元信号量(就是简单地作为某种触发的方式)
这种 channel 通常使用一个空结构体作为类型(因为不用传任何值)
```go
ch := make(chan struct{})
<-ch // 等待某件事情的发生,
```


buffered channels
```go
ch := make(chan int, 3)
```
使用 buffered channels 可以在缓存容量满之前一直放入数据

range 和 close :
发送者向 channel 发送完信息后可以使用 close 关闭, 表示没有要再往 channel 中发送的值了(只有发送者可以关闭)
range 会逐个读出 channel 中的值
```go
package main

import "fmt"

func fib(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fib(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}

```
需要注意的是: 使用 range 遍历 channel 只能有一个遍历参数(获得 channel 中的内容副本)
如果没有被 close 就使用 range 遍历, 那么在读完最后一个值后, range 语句将被 `block`, 直到有新值(继续 range )或 channel 被关闭(退出 range 语句, 继续往下执行)


select:
select 会阻塞到某个分支可以继续执行为止, 可以执行后就会执行该分支, 当多个分支都准备好执行时, 会随机选择一个分支执行
```go
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x: // 当通道 c 可写时，将当前斐波那契数 x 发送到通道 c 中
			x, y = y, x+y // 计算下一个斐波那契数
		case <-quit: // 当通道 quit 可读时，输出 "quit" 并返回
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int) // 创建一个整型通道 c
	quit := make(chan int) // 创建一个整型通道 quit

	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c) // 从通道 c 中接收并打印斐波那契数
		}
		quit <- 0 // 发送一个退出信号到通道 quit
	}()

	fibonacci(c, quit) // 启动斐波那契数列生成器
}

```
在这个例子中, main 函数创造了一个匿名的 goroutine, 用于从 c 中读出 10 个 fib 数, 并在读完了后向 quit 信道发出信号,  
在 main 函数中, fib 函数内使用 select 进行选择  
可加在 case 最后加上 default, 默认执行, 防止死锁


注意:
channel 与 map 和一样, 没有 `make`, 而是简单声明, 则会创造一个 `nil channel`, 不能接收也不能发送(会造成永远 `block`)

给一个已关闭的 channel 再发送内容会造成 `panic`



sync.Mutex:
Go sync 标准库包含了 Mutex 类型, 用于同步操作
Mutex 提供了两个接口
1. Lock()
2. Unlock()

对于线程不安全的类型(例如 map), 多个 `goroutine` 同时访问这个变量会造成 `panic`
因此对这些类型变量在并发时加锁是必须的
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	c.v[key]++
	c.mux.Unlock()
}
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	defer c.mux.Unlock()
	return c.v[key]
}
func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}

1000
```
使用 defer 语句解锁可以保证一定进行解锁操作(preferred)


sync.RWMutex
与严格的 Mutex 不同, RWMutex(读写锁)允许多个读者(操作系统读者写者问题), 因此在读密集的场景中, RWMutex 会有更好的表现
RWMutex 在 Mutex 之上, 还提供了
1. RLock()
2. RUnlock()

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	m := map[int]int{}

	mu := &sync.RWMutex{}

	go writeLoop(m, mu)
	go readLoop(m, mu)
	go readLoop(m, mu)
	go readLoop(m, mu)
	go readLoop(m, mu)

	// stop program from exiting, must be killed
	block := make(chan struct{})
	<-block
}

func writeLoop(m map[int]int, mu *sync.RWMutex) {
	for {
		for i := 0; i < 100; i++ {
			mu.Lock()
			m[i] = i
			mu.Unlock()
		}
	}
}

func readLoop(m map[int]int, mu *sync.RWMutex) {
	for {
		mu.RLock()
		for k, v := range m {
			fmt.Println(k, "-", v)
		}
		mu.RUnlock()
	}
}

```


Generics
泛型(就类似 C/C++的模板)
```go
func functionName[T any](s T) T {
	...
}
```
需要在参数前加上一个中括号表示泛型支持的类型

大部分情况下, 支持的类型通常是实现了某些功能的接口
```go
type stringer interface {
    String() string
}

func concat[T stringer](vals []T) string {
    result := ""
    for _, val := range vals {
        // this is where the .String() method
        // is used. That's why we need a more specific
        // constraint instead of the any constraint
        result += val.String()
    }
    return result
}

```
同时, 这也带来了一种新的定义 `interface` 的方法: 直接声明某种类型实现了这个接口
```go
// Ordered is a type constraint that matches any ordered type.
// An ordered type is one that supports the <, <=, >, and >= operators.
type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
        ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr |
        ~float32 | ~float64 |
        ~string
}

```

```go
// The store interface represents a store that sells products.
// It takes a type parameter P that represents the type of products the store sells.
type store[P product] interface {
	Sell(P)
}

type product interface {
	Price() float64
	Name() string
}

type book struct {
	title  string
	author string
	price  float64
}

func (b book) Price() float64 {
	return b.price
}

func (b book) Name() string {
	return fmt.Sprintf("%s by %s", b.title, b.author)
}

type toy struct {
	name  string
	price float64
}

func (t toy) Price() float64 {
	return t.price
}

func (t toy) Name() string {
	return t.name
}

// The bookStore struct represents a store that sells books.
type bookStore struct {
	booksSold []book
}

// Sell adds a book to the bookStore's inventory.
func (bs *bookStore) Sell(b book) {
	bs.booksSold = append(bs.booksSold, b)
}

// The toyStore struct represents a store that sells toys.
type toyStore struct {
	toysSold []toy
}

// Sell adds a toy to the toyStore's inventory.
func (ts *toyStore) Sell(t toy) {
	ts.toysSold = append(ts.toysSold, t)
}

// sellProducts takes a store and a slice of products and sells
// each product one by one.
func sellProducts[P product](s store[P], products []P) {
	for _, p := range products {
		s.Sell(p)
	}
}

func main() {
	bs := bookStore{
		booksSold: []book{},
	}
    
    // By passing in "book" as a type parameter, we can use the sellProducts function to sell books in a bookStore
	sellProducts[book](&bs, []book{
		{
			title:  "The Hobbit",
			author: "J.R.R. Tolkien",
			price:  10.0,
		},
		{
			title:  "The Lord of the Rings",
			author: "J.R.R. Tolkien",
			price:  20.0,
		},
	})
	fmt.Println(bs.booksSold)

    // We can then do the same for toys
	ts := toyStore{
		toysSold: []toy{},
	}
	sellProducts[toy](&ts, []toy{
		{
			name:  "Lego",
			price: 10.0,
		},
		{
			name:  "Barbie",
			price: 20.0,
		},
	})
	fmt.Println(ts.toysSold)
}

```