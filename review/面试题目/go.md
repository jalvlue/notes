### 如何判断 channel 是否关闭
```go
func isChanClose(ch chan int) bool { 
	select { 
	case _, received := <- ch: 
		return !received 
	default:
	}
	return false 
}
```

### 常见字符串拼接
- `+`, 直接将字符串相加, 开辟新空间存储
- `fmt.Sprintf`, 使用接口参数, 必须使用反射获取值, 性能损耗
- `strings.Builder`, 使用 `func WriteString()` 逐步构建, 底层是 `[]byte`
- `bytes.Buffer`, `[]byte`, 可转为 string
- `strings.Join`, 基于 `strings.Builder`

### 什么是 rune 类型
- `int32` 的别名
- 标识 `unicode` 的一个码点
- 使用 for range 遍历 string, 每一个字符的类型为 rune

string 底层是一个只读数组, 
`[]byte` 和 `string` 的相互转换会导致复制

### defer
defer 有一个栈, 所有被 defer 的函数被压入栈中, FIFO

```go
func test() int {
	i := 0
	defer func() {
		fmt.Println("defer1")
	}()
	defer func() {
		i += 1
		fmt.Println("defer2")
	}()
	return i
}

func main() {
	fmt.Println("return", test())
}
// defer2
// defer1
// return 0
```
defer 函数可以修改返回值, 只是对于没有命名的返回值, 无法修改, 
因为没有命名的返回值, runtime 将会创建一个临时变量保存, 这个临时变量在进入 defer 栈之前就已经确定了(由你 return 的值赋值过去), 因此后续在的 defer 函数中对 return 值的修改不会影响这个临时变量

```go
func test() (i int) {
	i = 0
	defer func() {
		i += 1
		fmt.Println("defer2")
	}()
	return i
}

func main() {
	fmt.Println("return", test())
}
// defer2
// return 1
```
这样的有命名返回值则在 defer 函数中的修改会实际影响到返回值, 因为这里不会再创建临时变量了

### Go 语言 tag 的用处
在 Go 语言中，结构体字段可以使用 tag（标签）来附加==元数据==。Tag 是一个以反引号包围的字符串，位于结构体字段的后方，例如：

```go
type Person struct {
	Name string `json:"name" xml:"fullName"`
	Age  int    `json:"age" xml:"age"`
}
```

在上述示例中，`json:"name" xml:"fullName"` 和 `json:"age" xml:"age"` 就是结构体字段的 tag。

Tag 的主要用途包括以下几个方面：

1. ==序列化与反序列化==：Tag 常用于指定结构体字段在序列化（将结构体转换为其他格式，如 JSON、XML）和反序列化（将其他格式转换为结构体）时的字段名、格式等信息。在上述示例中，`json:"name"` 和 `xml:"fullName"` 指定了该字段在 JSON 和 XML 序列化中的名称。

2. ==数据校验==：Tag 还可以用于数据校验，例如指定字段的最大长度、正则表达式等。一些 Go 语言的表单验证库可以通过解析结构体字段的 tag 来进行数据有效性验证。

3. ==ORM==（对象关系映射）：在使用 ORM 框架时，Tag 可以用于指定结构体字段与数据库表字段之间的映射关系，方便实现对象与数据库的转换。

需要注意的是，Tag 本身并不会被 Go 语言原生的代码使用或解析，而是由相关库或工具根据需要进行解析和处理。

通过使用 Tag，开发者可以在结构体字段上附加元数据，为字段提供更多的语义信息，使得代码在序列化、反序列化、数据校验等方面更加灵活和方便。


### reflect
TODO: 看看结构和 api

### 结构体打印时，`%v` 和 `%+v` 的区别

`%v`输出结构体各成员的值；

`%+v`输出结构体各成员的**名称**和**值**；

`%#v` 输出结构体名称和结构体各成员的名称和值

### 空 struct{} 的用途
- 使用 map 模拟 set, 不占空间
- 只传递信号的 channel 
- 只有方法, 不需要数据的结构体

### init() 函数是什么时候执行的？
执行顺序：import 解析依赖关系, 从依赖最轻的开始 –> const –> var –> `init()` –> `main()`

一个文件可以有多个`init()`函数！

### 如何知道一个对象是分配在栈上还是堆上
Golang 的编译器和运行时系统会根据对象的大小、生命周期等因素来决定对象是分配在栈上还是堆上。对于较小且生命周期短暂的对象，编译器通常会将其分配在栈上以提高性能。而对于较大或生命周期较长的对象，通常会分配在堆上。

总之，Golang 的内存管理是由垃圾回收器负责的，我们无法直接知道对象的具体分配位置。我们只需关注对象的引用和生命周期，而不必显式地管理内存的分配和释放。

如果**变量离开作用域后没有被引用**，则**优先**分配到栈上，否则分配到堆上。

### 2 个 nil 可能不相等吗？
可能不等。interface 在运行时绑定值，只有值为 nil 接口值才为 nil，但是与指针的 nil 不相等。举个例子：

```go
var p *int = nil
var i interface{} = nil
if(p == i){
	fmt.Println("Equal")
}
```
两者并不相同。总结：**两个 nil 只有在类型相同时才相等**。


### ==简述 Go 语言 GC(垃圾回收)的工作原理==
[[GC]]

### ==slice 工作机制==
[[slice]]

### 无缓冲的 channel 和有缓冲的 channel 的区别
- 无缓冲: 用于缓冲的环形数组容量为 0, 
	- 如果对方阻塞队列中有正在等待的协程, 则将队列首的协程节点取出, 将数据写给对方, 然后唤醒对方, 
	- 如果无, 则自己打包成一个 `sudog`, 加入我方的阻塞队列中, 等待一个对方来唤醒我
- 有缓冲: 缓冲数组没满则无序阻塞直接放缓冲中, 满了则跟无缓冲一样

[[Channel]]

### 为什么有协程泄露(Goroutine Leak)？
1. 缺少接收器，导致发送阻塞
2. 缺少发送器，导致接收阻塞
3. 死锁。多个协程由于竞争资源导致死锁。
4. 创建协程的没有回收。

### Go 可以限制运行时操作系统线程的数量吗？ 常见的 goroutine 操作函数有哪些？
- 可以，使用 runtime.GOMAXPROCS(num int)可以设置线程数目。该值默认为 CPU 逻辑核数，如果设的太大，会引起频繁的线程切换，降低性能。

- runtime.Gosched()，用于让出 CPU 时间片，让出当前 goroutine 的执行权限，调度器安排其它等待的任务运行，并在下次某个时候从该位置恢复执行。  

- runtime.Goexit()，调用此函数会立即使当前的 goroutine 的运行终止（终止协程），而其它的 goroutine 并不会受此影响。runtime.Goexit 在终止当前 goroutine 前会先执行此 goroutine 的还未执行的 defer 语句。请注意千万别在主函数调用 runtime.Goexit，因为会引发 panic。


### 如何控制协程数目
- `sync.WaitGroup`
- `channel`
- 协程池

### new 和 make 的区别
1. `new` 函数：`new` 用于分配内存并返回指向该内存的指针。它接受一个类型作为参数，并返回一个指向该类型的零值的指针。它不会初始化内存，而只会为其分配零值。一般用于分配值类型（如基本类型、结构体等）的内存。
    
2. `make`函数：`make`用于创建并初始化引用类型的数据结构（如切片、映射和通道）。它接受一个类型、一些长度参数和容量参数（在适用的情况下），并返回一个已经初始化的该类型的值。`make`函数会分配内存，并根据参数进行初始化，返回的是一个非零值。

### init
1. 无法被调用：`init` 函数不能被显式地调用，只能在程序启动时由运行时系统自动执行。
2. 无法被引用：`init`函数不能被其他函数或代码引用，因为它们没有名称。
3. 无法传递参数：`init`函数不能接受任何参数，也不能返回任何值。
4. 用途：`init` 函数通常用于执行包的初始化操作，例如初始化全局变量、注册驱动、打开数据库连接、执行配置解析等。`init` 函数的执行顺序对于包的初始化过程非常重要。
5. 避免依赖

### 下面这句代码是什么作用，为什么要定义一个空值？

```go
type GobCodec struct{
	conn io.ReadWriteCloser
	buf *bufio.Writer
	dec *gob.Decoder
	enc *gob.Encoder
}

type Codec interface {
	io.Closer
	ReadHeader(*Header) error
	ReadBody(interface{})  error
	Write(*Header, interface{}) error
}

var _ Codec = (*GobCodec)(nil)
```
- 进行类型检查和接口实现的静态验证。
- 如果 `GobCodec` 类型没有实现 `Codec` 接口的所有方法，编译器会在编译时产生错误。
- 不关心具体的值。通过将其赋值为空值，可以避免创建一个实际的对象实例。


### ==简述 go 内存管理机制==
[[GC]]

### mutex有几种模式
[[Mutex]]
- 正常模式: 先自旋后阻塞, 不太公平
- 饥饿模式: 无自旋, 一律进入阻塞队列

### Go 什么时候发生阻塞？阻塞时，调度器会怎么做。

- 用于**原子、互斥量或通道**操作导致goroutine阻塞，调度器将把当前阻塞的goroutine从本地运行队列**LRQ换出**，并重新调度其它goroutine；
- 由于**网络请求**和**IO**导致的阻塞，Go 提供了网络轮询器（Netpoller）来处理，后台用 epoll 等技术实现 IO 多路复用。

### goroutine 什么情况会发生内存泄漏？如何避免。

在Go中内存泄露分为暂时性内存泄露和永久性内存泄露。

**暂时性内存泄露**
- 获取长slice中的一段导致长slice未释放
- 在长slice新建slice导致泄漏


**永久性内存泄露**
- goroutine永久阻塞而导致泄漏
- time.Ticker未关闭导致泄漏

### Go GC 有几个阶段

目前的go GC采用**三色标记法**和**混合写屏障**技术。

Go GC有**四**个阶段:

- STW，开启混合写屏障，扫描栈对象；
- 将所有对象加入白色集合，从根对象开始，将其放入灰色集合。每次从灰色集合取出一个对象标记为黑色，然后遍历其子对象，标记为灰色，放入灰色集合；
- 如此循环直到灰色集合为空。剩余的白色对象就是需要清理的对象。
- STW，关闭混合写屏障；
- 在后台进行 GC（并发）。


### 如果若干个 goroutine，有一个 panic 会怎么做？

有一个 panic，那么剩余 goroutine 也会退出，程序退出。如果不想程序退出，那么必须通过调用 recover() 方法来捕获 panic 并恢复将要崩掉的程序。

### gRPC 是什么
基于 go 的**远程过程调用**。RPC 框架的目标就是让远程服务调用更加简单、透明，RPC 框架负责屏蔽底层的传输方式（TCP 或者 UDP）、序列化方式（XML/Json/ 二进制）和通信细节。服务调用者可以像调用本地接口一样调用远程的服务提供者，而不需要关心底层通信细节和调用过程。

### GIN 怎么做参数校验
[[simplebank-BackendMasterClass#10. 自定义 validator]]

go 采用 validator 作参数校验。

它具有以下独特功能：

- 使用验证tag或自定义validator进行跨字段Field和跨结构体验证。
- 允许切片、数组和哈希表，多维字段的任何或所有级别进行校验。
- 能够对哈希表key和value进行验证
- 通过在验证之前确定它的基础类型来处理类型接口。
- 别名验证标签，允许将多个验证映射到单个标签，以便更轻松地定义结构体上的验证
- Gin web 框架的默认验证器；

### 中间件用过吗？
[[simplebank-BackendMasterClass#17. 新增 auth 中间件]]
Middleware 是 Web 的重要组成部分，中间件（通常）是一小段代码，它们接受一个请求，对其进行处理，每个中间件只处理一件事情，完成后将其传递给另一个中间件或最终处理程序，这样就做到了程序的解耦。


### Go 解析 Tag 是怎么实现的？

Go解析tag采用的是**反射**。

具体来说使用reflect.ValueOf方法获取其反射值，然后获取其Type属性，之后再通过Field(i)获取第i+1个field，再.Tag获得Tag。

反射实现的原理在: `src/reflect/type.go` 中

### 持久化怎么做的
- 简单的方式就是采用 io ioutil 的 WriteFile ()方法将字符串写到磁盘上
- 数据库

### 说说 atomic 底层怎么实现的.

atomic 源码位于 `sync\atomic`。通过阅读源码可知，atomic 采用**CAS**（CompareAndSwap）的方式实现的。所谓 CAS 就是使用了 CPU 中的原子性操作。在操作共享变量的时候，CAS 不需要对其进行加锁，而是通过类似于乐观锁的方式进行检测，总是假设被操作的值未曾改变（即与旧值相等），并一旦确认这个假设的真实性就立即进行值替换。本质上是**不断占用 CPU 资源来避免加锁的开销**。

### channel 底层实现？是否线程安全。
[[Channel]]

### map 的底层实现
[[map]]

### select 底层实现
TODO

### go 的调试/分析工具用过哪些。

go的自带工具链相当丰富，

- go cover : 测试代码覆盖率；
- godoc: 用于生成go文档；
- pprof：用于性能调优，针对cpu，内存和并发；
- race：用于竞争检测；

### 说说 context 包的作用？你用过哪些，原理知道吗？
[[review/go/Context|Context]]
`context` 可以用来在 `goroutine` 之间传递上下文信息，相同的 `context` 可以传递给运行在不同 `goroutine` 中的函数，上下文对于多个 `goroutine` 同时使用是安全的，`context` 包定义了上下文类型，可以使用 `background`、`TODO` 创建一个上下文，在函数调用链之间传播 `context`，也可以使用 `WithDeadline`、`WithTimeout`、`WithCancel` 或 `WithValue` 创建的修改副本替换它，听起来有点绕，其实总结起就是一句话：**`context` 的作用就是在不同的 `goroutine` 之间同步请求特定的数据、取消信号以及处理请求的截止日期**。


### golang 并发模型
golang 的并发模型是 CSP-communicating sequential process, 通信顺序进程模型

CSP 模型的核心思想是“不要通过共享内存来通信，而要通过通信来共享内存”。这一思想有效地避免了传统并发编程中常见的竞态条件和死锁问题。

- 要点是使用 channel





### 实现使用字符串函数名，调用函数。
```go
package main
import (
	"fmt"
    "reflect"
)

type Cat struct{
}

func (c *Cat) Speak(){
    fmt.Println("meow")
}

func main(){
	c := Cat{}
	v := reflect.ValueOf(&c)
	m := v.MethodByName("Speak")
	m.Call(nil)
}
```

### （Goroutine）有三个函数，分别打印"cat", "fish","dog"要求每一个函数都用一个goroutine，按照顺序打印100次。

```go
package main
  
import (
  "fmt"
  "sync"
)
  
func cat() {
  defer wg.Done()
  for i := 0; i < 100; i++ {
    fmt.Println("cat")
    cd <- struct{}{}
    <-fc
  }
}
  
func dog() {
  defer wg.Done()
  for i := 0; i < 100; i++ {
    <-cd
    fmt.Println("dog")
    df <- struct{}{}
  }
}
  
func fish() {
  defer wg.Done()
  for i := 0; i < 100; i++ {
    <-df
    fmt.Println("fish")
    fc <- struct{}{}
  }
}
  
var (
  wg = sync.WaitGroup{}
  
  cd = make(chan struct{})
  df = make(chan struct{})
  fc = make(chan struct{})
)
  
func main() {
  wg.Add(3)
  go cat()
  go dog()
  go fish()
  
  wg.Wait()
}
```

### 两个协程交替打印10个字母和数字

```go
package main
  
import (
  "fmt"
  "sync"
)
  
var (
  wg = sync.WaitGroup{}
  
  num   = make(chan struct{})
  digit = make(chan struct{})
)
  
func printNum() {
  defer wg.Done()
  
  for i := 0; i < 10; i++ {
    <-digit
    fmt.Println(1)
    num <- struct{}{}
  }
}
  
func printDigit() {
  defer wg.Done()
  
  for i := 0; i < 10; i++ {
    fmt.Println("a")
    digit <- struct{}{}
    <-num
  }
}
  
func main() {
  wg.Add(2)
  go printNum()
  go printDigit()
  
  wg.Wait()
}
```

### 启动 2个 groutine 2秒后取消，第一个协程1秒执行完，第二个协程3秒执行完。
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func f1(in chan struct{}) {

	time.Sleep(1 * time.Second)
	in <- struct{}{}

}

func f2(in chan struct{}) {
	time.Sleep(3 * time.Second)
	in <- struct{}{}
}

func main() {
	ch1 := make(chan struct{})
	ch2 := make(chan struct{})
	ctx, _ := context.WithTimeout(context.Background(), 2*time.Second)

	go func() {
		go f1(ch1)
		select {
		case <-ctx.Done():
			fmt.Println("f1 timeout")
			break
		case <-ch1:
			fmt.Println("f1 done")
		}
	}()

	go func() {
		go f2(ch2)
		select {
		case <-ctx.Done():
			fmt.Println("f2 timeout")
			break
		case <-ch2:
			fmt.Println("f2 done")
		}
	}()
	time.Sleep(time.Second * 5)
}
```


### 当 select 监控多个 chan 同时到达就绪态时，如何先执行某个任务？
```go
func priority_select(ch1, ch2 <-chan string) {
  select {
  case val := <-ch1:
    fmt.Println(val)
  case val2 := <-ch2:
  priority:
    for {
      select {
      case val1 := <-ch1:
        fmt.Println(val1)
  
      default:
        break priority
      }
    }
    fmt.Println(val2)
  }
}
```

