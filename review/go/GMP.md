https://stackoverflow.com/questions/68312082/when-will-go-scheduler-create-a-new-m-and-p


# 1. 基本概念
### 线程
- 操作系统内核级线程
- 是操作系统最小调度单元
- 创建、销毁、调度由内核完成
- 可充分利用多核实现并行

### 协程
![[Pasted image 20240508152253.png]]
协程，又称为用户级线程，核心点如下：
- 与内核线程存在映射关系，为 M: 1，通常多个协程依附于一个内核线程
- 创建、销毁、调度在用户态完成，对内核透明，更加轻量级
- 依附于同一个内核线程的协程无法并行

### gouroutine
![[Pasted image 20240508152630.png]]
Goroutine，经 Golang 优化后的特殊“协程”，核心点如下：
- 与线程存在映射关系，为 M：N；
- 创建、销毁、调度在用户态完成，对内核透明，足够轻便；
- 可利用多个线程实现并行
- 通过调度器的斡旋，实现 goroutines 和线程间的动态绑定和灵活调度；
- 栈空间大小可动态扩缩，因地制宜

# 2. gmp
gmp = goroutine + machine + processor

### g
- G 即 goroutine，是 golang 中对协程的抽象；
- G 有自己的运行栈、状态、以及执行的任务函数（用户通过 go func 指定）；
- G 需要绑定到 p 才能执行，在 g 的视角中，p 就是它的 cpu

### p
- P 即 processor，是 golang 中的调度器；
- P 是 gmp 的中枢，借由 p 承上启下，实现 g 和 m 之间的动态有机结合；
- 对 g 而言，p 是其 cpu，g 只有被 p 调度，才得以执行；
- 对 m 而言，p 是其执行代理，为其提供必要信息的同时（可执行的 g、内存分配情况等），并隐藏了繁杂的调度细节；
- P 的数量决定了 g 最大并行数量，可由用户通过 GOMAXPROCS 进行设定（超过 CPU 核数时无意义）

### m
- M 即 machine，是 golang 中对线程的抽象；
- M 不直接执行 g，而是先和 p 绑定，由其实现代理；
- 借由 p 的存在，m 无需和 g 绑死，也无需记录 g 的状态信息，因此 g 在全生命周期中可以实现跨 m 执行 (通过 p 的代理，在多个内核线程中执行)

### gmp
![[Pasted image 20240508153431.png]]

- M 是线程的抽象；G 是 goroutine；P 是承上启下的调度器；
- M 调度 G 前，需要和 P 绑定；
- 全局有多个 M 和多个 P，但同时并行的 G 的最大数量等于 P 的数量；
- G 的存放队列有三类：P 的本地队列；全局队列；和 wait 队列（图中未展示，为 io 阻塞就绪态 goroutine 队列）；
- M 调度 G 时，优先取 P 本地队列，其次取全局队列，最后取 wait 队列；这样的好处是，取本地队列时，可以接近于无锁化，减少全局锁竞争；==
- 为防止不同 P 的闲忙差异过大，设立 work-stealing 机制，本地队列为空的 P 可以尝试从其他 P 本地队列偷取一半的 G 补充到自身队列


# 3. 核心数据结构
### g
```go
type g struct {
	// ...
	m *m
	sched gobuf
	// ...
}

type gobuf struct {
	sp uintptr // stack pointer
	pc uintptr // program counter
	ret uintptr 
	bp uintptr // for framepointer-enabled architectures
}
```
runtime 中使用 `type g struct` 来抽象 g, 
这里有两个重要的域:
- m: 在 p 的代理，负责执行当前 g 的 m；
- sched: 保存当前 g 的一些执行信息(寄存器状态)

g 的状态机
![[Pasted image 20240508162250.png]]
```
const(  
	_Gidle = itoa // 0
	_Grunnable // 1
	_Grunning // 2
	_Gsyscall // 3
	_Gwaiting // 4
	_Gdead // 6
	_Gcopystack // 8
	_Gpreempted // 9
)
```

### m
```go
type m struct {
	g0      *g     // goroutine with scheduling stack
	// ...
	tls           [tlsSlots]uintptr // thread-local storage (for x86 extern register)
	// ...
}

const tlsSlots untyped int = 6
```
- g0：一类特殊的调度协程，不用于执行用户函数，负责执行 g 之间的切换调度. 与 m 的关系为 1:1；
- tls：thread-local storage，线程本地存储，存储内容只对当前线程可见. 线程本地存储的是 m.tls 的地址，m.tls[0] 存储的是当前运行的 g，因此线程可以通过 g 找到当前的 m、p、g 0 等信息


### p
```go
type p struct {
	// ...
	runqhead uint32
	runqtail uint32
	runq     [256]guintptr
	runnext guintptr
	// ...
}
```
- runq: 本地 g 队列, 最大长度为 256
- runqhead: 队列头
- runqtail: 队列尾
- runnext: 写一个可执行的g


### schedt
上图中, 每个 p 的本地调度队列之外的全局队列, 同样也是维护一个队列
```go
type schedt struct {
	// ...
	lock mutex
	// ...
	runq     gQueue
	runqsize int32
	// ...
}
```
- 使用一个互斥锁进行并发保护(多个 p 同时向 schedt 取元素)


# 4. 调度流程
### 两种 g
在上面的 m 结构中, 有一个帮助 m 选择执行用户协程的特殊协程 `g0`

因此调度过程基本上就是:
- m 执行 `g0`
- `g0` 运行时察觉到有可调度的用户协程, 使用 `func gogo()` 调度用户协程
- 用户协程执行完成或阻塞, 通过 `func m_call()` 切换回 `g0` (??? 谁通过 `func m_call()`)
- ...

![[Pasted image 20240508170246.png]]


gogo 和 mcall 可以理解为对偶关系，其定义位于 runtime/stubs.go 文件中
```go
func gogo(buf *gobuf)// ...
func mcall(fn func(*g))
```

### 调度类型
![[Pasted image 20240508170737.png]]
- 主动让出 `func gosched()`
- 被动调度 (阻塞) `func gopark()`, 从阻塞中恢复成就绪态使用 `func goready()` 
- 正常调度: g 执行完成
- 抢占调度(不理解)


### 宏观调度流程
![[Pasted image 20240508182525.png]]

- 以 g 0 -> g -> g 0 的一轮循环为例进行串联；
- g0 执行 schedule() 函数，寻找到用于执行的 g；
- g0 执行 execute() 方法，更新当前 g、p 的状态信息，并调用 gogo() 方法，将执行权交给 g；
- g 因主动让渡( gosche_m() )、被动调度( park_m() )、正常结束( goexit0() )等原因，调用 m_call 函数，执行权重新回到 g0 手中；
- g0 执行 schedule() 函数，开启新一轮循环

### func schedule()
```go
func schedule() {
	// ...
	gp, inheritTime, tryWakeP := findRunnable() // blocks until work is available
	
	// ...
	execute(gp, inheritTime)
}
```

### func fundRunnable()
![[Pasted image 20240508182739.png]]
- P 从本地队列中尝试获取
- P 从全局队列中尝试获取
- P 从就绪的 io 协程中尝试获取
- P 从其他 p 那里偷走一半的g

这里为了放置本地队列过于繁忙, 全局队列可能饥饿, 因此多加了一个机制: 每 61 次就优先操作全局队列
![[Pasted image 20240508182939.png]]

### func execute()
![[Pasted image 20240508184259.png]]

### func Gosched()
g 主动让渡的函数
![[Pasted image 20240508184517.png]]