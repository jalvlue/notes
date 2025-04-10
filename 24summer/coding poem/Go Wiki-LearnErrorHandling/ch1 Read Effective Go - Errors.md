```go
type error interface {
    Error() string
}
```

- `golang` 函数的多值返回允许将详细的错误信息作为返回值返回给调用者
- 同时, go 的习惯也是将错误信息返回给调用者
- 如果调用者需要了解并处理详细的错误信息, 那么可以对类型进行判断

```go
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}

for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        deleteTempFiles()  // Recover some space.
        continue
    }
    return
}
```


### Panic
- 如果出现了致命错误, 程序因此无法运行下去, 此时就应该终止程序, `panic` 是一个内建函数, 会终止程序的运行
- 任意一个协程的 panic 都会导致整个程序的终止

```go
// A toy implementation of cube root using Newton's method.
func CubeRoot(x float64) float64 {
    z := x/3   // Arbitrary initial value
    for i := 0; i < 1e6; i++ {
        prevz := z
        z -= (z*z*z-x) / (3*z*z)
        if veryClose(z, prevz) {
            return z
        }
    }
    // A million iterations has not converged; something is wrong.
    panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```
- 这个例子就认为在 `1e6` 次迭代后还没有找出结果是一种异常, 直接使用 `panic` 终止程序的运行
- 实际上, 如果问题可以重试或者处理的话应该尽量避免使用 `panic`
- `panic` 在初始化时比较常用, 初始化的失败通常导致程序无法进行下去

```go
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```


# Recover
- `panic()` 调用之后, 协程立即终止当前的执行流, 开始执行 `defer` 调用栈
- 可以在 `defer func` 中调用内建的 `recover()` 来捕获 `panic`, 从而让该异常只终止当前协程, 而不影响到整个程序

```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```

- `err := recover()` 中, `err` 只是根据 `panic(...)` 信息构造的一个 `error`
- `recover()` 函数一般就将造成 `panic` 的信息打印成日志
