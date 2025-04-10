# 1. 前置知识
### sync.Locker
```go
// A Locker represents an object that can be locked and unlocked.
type Locker interface {
    Lock()    
    Unlock()
}
```
`Locker` 是标准的锁接口, 任何实现这个接口的类都可以作为一把锁, `sync.Mutex` 就是最常见的锁

[[锁]]
ants 中作者根据实际情况, 实现了一种更轻量级的自旋锁, 而不是使用 `sync.Mutex`
![[Pasted image 20240509110202.png]]

```go
type spinLock uint32
const maxBackoff = 16
func (sl *spinLock) Lock() {    
	backoff := 1    
	for !atomic.CompareAndSwapUint32((*uint32)(sl), 0, 1) {        
		for i := 0; i < backoff; i++ {            
			runtime.Gosched()        
		}        
		if backoff < maxBackoff {            
			backoff <<= 1        
		}    
	}
}
	
func (sl *spinLock) Unlock() {    
	atomic.StoreUint32((*uint32)(sl), 0)
}
```
- 通过一个整型状态值标识锁的状态：0-未加锁；1-加锁；
- 加锁成功时，即把 0 改为 1；解锁时则把 1 改为 0；改写过程均通过 atomic 包保证并发安全；
- 加锁通过 for 循环 + cas 操作实现自旋，无需操作系统介入执行 park 操作；
- 通过变量 backoff 反映抢锁激烈度，每次抢锁失败，执行 backoff 次让 cpu 时间片动作；backoff 随失败次数逐渐升级，封顶 16.

这里利用了 `atomic.CompareAndSwap` 进行加锁尝试, 并根据系统的繁忙情况调整自选的长度, 系统越繁忙就越自旋, 自旋时通过 `runtime.Gosched()` 主动将当前 M 让出, 给其他协程执行


### sync.Cond
[[GoGo/Up and Running Concurrency in Go/main#condition|main]]
条件变量, 同于指定条件下的协程阻塞唤醒操作
![[Pasted image 20240509111533.png]]

```go
type Cond struct {    
	noCopy noCopy    
	
	// L is held while observing or changing the condition    
	L Locker    
	
	notify  notifyList    
	checker copyChecker
}

type notifyList struct {
    wait   uint32
    notify uint32    
    lock   uintptr // key field of the mutex    
    head   unsafe.Pointer    
    tail   unsafe.Pointer
}

// NewCond returns a new Cond with Locker l.
func NewCond(l Locker) *Cond {
    return &Cond{L:
```

- 成员变量 noCopy + checker 是一套组合拳，保证 Cond 在第一次使用后不允许被复制；
- 核心变量 L，一把锁，用于实现阻塞操作；
- 核心变量 notify，阻塞链表，分别存储了调用 Cond.Wait () 方法的次数、goroutine 被唤醒的次数、一把系统运行时的互斥锁以及链表的头尾节点.

```go
func (c *Cond) Wait() {
    c.checker.check()    
    t := runtime_notifyListAdd(&c.notify)    
    c.L.Unlock()    
    runtime_notifyListWait(&c.notify, t)    
    c.L.Lock()
}

func (c *Cond) Signal() {
    c.checker.check()
    runtime_notifyListNotifyOne(&c.notify)
}

func (c *Cond) Broadcast() {
    c.checker.check()    
    runtime_notifyListNotifyAll(&c.notify)
}
```
这里可以看到, `sync.Cond` 提供的接口都比较简单, 根据进入等待的队列 `c. notify` 顺序 FIFO, 


### sync.Pool
sync.Pool 是 golang 标准库下并发安全的对象池，适合用于有大量对象资源会存在被反复构造和回收的场景，可缓存资源进行复用，以提高性能并减轻 GC 压力.

![[Pasted image 20240509162733.png]]
```go
type Pool struct {
    noCopy noCopy    
    local     unsafe.Pointer // local fixed-size per-P pool, actual type is [P]poolLocal    
    localSize uintptr        // size of the local array    
    victim     unsafe.Pointer // local from previous cycle    
    victimSize uintptr        // size of victims array    
    
    
    // New optionally specifies a function to generate    
    // a value when Get would otherwise return nil.    
    // It may not be changed concurrently with calls to Get.
    New func()
}

type poolLocal struct {    
poolLocalInternal
}

// Local per-P Pool appendix.
type poolLocalInternal struct {
    private any       // Can be used only by the respective P.
    shared  poolChain // Local P can pushHead/popHead; any P can popTail.
}
```
这里 pool 中最重要的成员就是 `local [P]pollLocal`, 表示与 GMP 中的每一个 P 对应, 而 P 作为 G 的代理, 每次有 G 与 P 结合之后, 需要申请一个池内元素时, 
从池子的角度出发, 实际上是只看到了 P, 因此对每一个 P 维护一个 `pollLocal`, 拥有 `private object` 和 `shared pollChain`

Poll.pin: 将当前的 P 和 G 暂时绑定

Poll.Get: 复用
![[Pasted image 20240509164806.png]]

Poll.Put:
![[Pasted image 20240509164833.png]]


回收:
![[Pasted image 20240509165050.png]]
poll 的回收是以 runtime GC 为指示的, 每一论 GC 开始时, 将每一个 `pollLocal` 的 `victim` 直接释放, 然后将当前的 `local` 转为 `victim`, `local` 则置为空


# 2. Ants
https://github.com/panjf2000/ants
是一个协程池, 将协程作为池子里的元素进行复用
场景:
- Http 请求比如 gin 会创建 gin. Context, 如果太多并发请求会导致性能出现尖刺, 使用协程池有利于平缓性能尖刺
- 提升性能：主要面向一类场景：大批量轻量级并发任务，任务执行成本与协程创建/销毁成本量级接近
- 有一个并发资源控制的概念：研发能够明确系统全局并发度以及各个模块的并发度上限
- 协程生命周期控制：实时查看当前全局并发的协程数量；有一个统一的紧急入口释放全局协程

![[Pasted image 20240509165823.png]]
ants 将协程封装成了 `worker`, 每一个都是长时运行的, 等待分配任务
```go
type goWorker struct {
    pool *Pool    
    task chan func()    
    recycleTime time.Time
}

type Pool struct {
    capacity int32    
    running int32    
    lock sync.Locker    
    workers workerArray    
    state int32    
    cond *sync.Cond    
    workerCache sync.Pool    
    waiting int32    
    heartbeatDone int32    
    stopHeartbeat context.CancelFunc    
    options *Options
}

type workerArray interface {
    len() int
    isEmpty() bool
    insert(worker *goWorker) error
    detach() *goWorker
    retrieveExpiry(duration time.Duration) []*goWorker
    reset()
}
```
- workers：goWorker 列表，即“真正意义上的协程池”
- workerCache：存放 goWorker 的对象池，用于缓存释放的 goworker 资源用于复用. 对象池需要区别于协程池，协程池中的 goWorker 仍存活，进入对象池的 goWorker 严格意义上已经销毁


TODO



