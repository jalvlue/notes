# Timeout
- 使用 `time.After()`
```go
timeout := make(chan bool, 1)
go func() {
    time.Sleep(1 * time.Second)
    timeout <- true
}()

select {
case <-ch:
    // a read from ch has occurred
case <-timeout:
    // the read from ch has timed out
}
```


# Moving on
- 从多个数据库访问中取一个的结果, 遇到阻塞的就跳过直接尝试下一个
```go
func Query(conns []Conn, query string) Result {
    ch := make(chan Result)
    for _, conn := range conns {
        go func(c Conn) {
            select {
            case ch <- c.DoQuery(query):
            default:
            }
        }(conn)
    }
    return <-ch
}
```