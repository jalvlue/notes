[[GoGo/packages/context|context]]
context 的核心就是 `context.Context`

主要用途:
- 对 goroutine 的生命周期进行控制
- 在异步场景中进行并发协调

# 核心数据结构
```go
type Context interface {
	Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}    
    Err() error    
    Value(key any) any
}
```
![[Pasted image 20240428212225.png]]

### 标准 error
```go
var Canceled = errors.New("context canceled")

var DeadlineExceeded error = deadlineExceededError{}

type deadlineExceededError struct{}

func (deadlineExceededError) Error() string   { return "context deadline exceeded" }
func (deadlineExceededError) Timeout() bool   { return true }
func (deadlineExceededError) Temporary() bool { return true }
```
这里有两个标准错误, 分别是一个 Context 被取消或者超时时返回


# 最基本的 empytCtx
```go
type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {    return}

func (*emptyCtx) Done() <-chan struct{} {    return nil}

func (*emptyCtx) Err() error {    return nil}

func (*emptyCtx) Value(key any) any {    return }
```
这就是一个空的 Context, 没有超时限制、结束限制、没有错误
其他的所有 Context
- CancelCtx
- TimerCtx
- ValueCtx

都是基于这个 emptyCtx 新增功能衍生而来


### context.Background() context.TODO()
```go
var (    
	background = new(emptyCtx)    
	todo       = new(emptyCtx)
)

func Background() Context {    
	 return background
}

func TODO() Context {
	return todo
}
```


# 基石 cancelCtx
```go
type cancelCtx struct {
	Context       mu       sync.Mutex            // protects following fields
	done     atomic.Value          // of chan struct{}, created lazily, closed by first cancel call    
	children map[canceler]struct{} // set to nil by the first cancel call    
	err      error                 // set to non-nil by the first cancel call
}

type canceler interface {    
	cancel(removeFromParent bool, err error)    
	Done() <-chan struct{}
}
```

![[Pasted image 20240428213228.png]]

这里 cancelCtx, 顾名思义就是可以取消的上下文, 
取消后, 只读通道成员 `Done` 就会被关闭, 
这样, 使用这个上下文的子协程就可以通过 `Done` 识别到这个上下文被关闭了, 从而进行协程回收工作等


TODO
