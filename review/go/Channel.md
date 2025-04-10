# 1. 核心数据结构
![[Pasted image 20240508200510.png]]
![[Pasted image 20240508200547.png]]

### waitq
waitq：阻塞的协程队列(链式)
```go
type waitq struct {
	first *sudog
	last  *sudog
}
```

封装的协程节点
```go
type sudog struct {
	g *g    
	next *sudog    
	prev *sudog    
	elem unsafe.Pointer // data element (may point to stack)
	isSelect bool    
	c        *hchan 
}
```

# 2. 构造 channel
![[Pasted image 20240508202522.png]]
主要分为三类进行构造
- 存储缓冲元素需要的空间为 0 (无缓冲或类型为 `struct{}`)
- 有缓冲 struct 型 channel
- 有缓冲 pointer 型 channel

![[Pasted image 20240508202847.png]]


# 3. 写流程
### 两类异常
- 对于未初始化的 chan，写入操作会引发死锁；
- 对于已关闭的 chan，写入操作会引发 panic


### 写时存在阻塞读协程
取出一个阻塞协程的 sudog, 然后将元素拷贝给它
![[Pasted image 20240508205738.png]]

### 无阻塞协程而且缓冲区仍有空间
将元素添加到缓冲区中

### 无阻塞协程而且无缓冲空间
![[Pasted image 20240508205948.png]]
封装 sudog 然后加入阻塞队列中, 

同时对当前 goroutine, 使用 `func gopart()` 被动调度进入阻塞状态


### 总体流程
![[Pasted image 20240508210325.png]]


# 4. 读流程
### 两种异常
- 读空 channel, 导致调用 `func gopark()` 造成死锁 (实际上 runtime 会抛出 panic )
- channel 已关闭而且内部无元素, 直接返回默认值在

### 读时有阻塞的写协程
![[Pasted image 20240508210852.png]]

- 加锁；
- 从阻塞写协程队列中获取到一个写协程；
- 倘若 channel 无缓冲区，则直接读取写协程元素，并唤醒写协程；
- 倘若 channel 有缓冲区，则读取缓冲区头部元素，并将写协程元素写入缓冲区尾部后唤醒写写成；
- 解锁，返回

### 读时无阻塞写协程, 并且缓冲区有元素
直接从缓冲区的头读

### 读时无阻塞写协程而且缓冲区无元素
- 加锁；
- 构造封装当前 goroutine 的 sudog 对象；
- 完成指针指向，建立 sudog、goroutine、channel 之间的指向关系；
- 把 sudog 添加到当前 channel 的阻塞读协程队列中；
- park 当前协程；
- 倘若协程从 park 中被唤醒，则回收 sudog（sudog能被唤醒，其对应的元素必然已经被写入）；
- 解锁，返回


### 总体流程
![[Pasted image 20240508212903.png]]


# 5. 阻塞与非阻塞模式
在上述描述中, 都是默认是阻塞模式, 也就是直接对 channel 进行读写

然而在 select 中, 对 channel 的读写是不阻塞的, 也就是用到了 channel 的非阻塞模式


# 6. 两种读 channel 的方式
![[Pasted image 20240508225022.png]]

# 7. 关闭 channel
![[Pasted image 20240508225035.png]]



- 关闭未初始化过的 channel 会 panic；
- 加锁；
- 重复关闭 channel 会 panic；
- 将阻塞读协程队列中的协程节点统一添加到 glist；
- 将阻塞写协程队列中的协程节点统一添加到 glist；
- 唤醒 glist 当中的所有协程.


一点补充:
这里将阻塞的读写协程都唤醒了
实际上, 对于读协程, 由 close 唤醒会导致协程抛出读已关闭的 channel 的 panic

对于写协程则没有问题
