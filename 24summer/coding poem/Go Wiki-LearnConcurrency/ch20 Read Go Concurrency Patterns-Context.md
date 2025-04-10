
[[review/go/Context|Context]]

# Introduction
- Go 的服务器一般在接受到请求之后另起协程来处理该请求
- 这个处理协程可能又另起好几个协程来访问数据库或者调用 RPC 服务
- 整个处理链路中, 可能都需要一些一致的数据, 例如用户信息, authorization token, request deadline...
- 此外, 当一个请求被取消时, 就需要一个机制来通知这些协程停止执行, 以便回收资源
- 以上两个需求就引入了 `context`, 来在 API 边界之间做上面的事情


# Context
```go
// A Context carries a deadline, cancellation signal, and request-scoped values
// across API boundaries. Its methods are safe for simultaneous use by multiple
// goroutines.
type Context interface {
    // Done returns a channel that is closed when this Context is canceled
    // or times out.
    Done() <-chan struct{}

    // Err indicates why this context was canceled, after the Done channel
    // is closed.
    Err() error

    // Deadline returns the time when this Context will be canceled, if any.
    Deadline() (deadline time.Time, ok bool)

    // Value returns the value associated with key or nil if none.
    Value(key interface{}) interface{}
}
```
- `Done()`: 取消通知
- `Err()`: Done 的原因
- `Deadline()`: 
- `Value()`: 可以在当前 `Context` 中存储一些跨 API 使用的值
- 多个协程共用一个 `Context` 是并发安全的

# Derived contexts
- 上下文可以共享, 也可以树的形式派生
- 如果跟 `Context` 被取消了, 那么整棵树的所有 `Context` 都会被取消
- `Background` 一般用作根 `Context`, 永远不会被取消
```go
// Background returns an empty Context. It is never canceled, has no deadline,
// and has no values. Background is typically used in main, init, and tests,
// and as the top-level Context for incoming requests.
func Background() Context
```
- 派生的 `Context`
```go
// WithCancel returns a copy of parent whose Done channel is closed as soon as
// parent.Done is closed or cancel is called.
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)

// A CancelFunc cancels a Context.
type CancelFunc func()

// WithTimeout returns a copy of parent whose Done channel is closed as soon as
// parent.Done is closed, cancel is called, or timeout elapses. The new
// Context's Deadline is the sooner of now+timeout and the parent's deadline, if
// any. If the timer is still running, the cancel function releases its
// resources.
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)

// WithValue returns a copy of parent whose Value method returns val for key.
func WithValue(parent Context, key interface{}, val interface{}) Context
```

# Example: Google Web Search
- 就是简单讲了如何派生 `Context`, 同时如何传递


# Adapting code for Contexts
- 除了标准库的一些其他 `Context` 实现


# Conclusion
- Google 的规范是如果又上述的超时和取消控制需求 `ctx context.Context` 一般都要放在函数的第一个参数中
