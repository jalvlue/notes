# context
`context` 结构将所有的 `http request` 以及对应的 `http responseWriter` 进行了封装, 
每一个 `http request` 都会生成一个 `context`,
将动态传递的参数解析出来, 并定义好中间件 `handler` 以及真正执行处理逻辑的 `handler`, 
因此有了 `context` 后, `handler` 的定义也就变成了
```go
type HandlerFunc func(*Context)
```

除了处理 `request` 外, `context` 还有成员函数
```go
// response with plain text
func (c *Context) String(code int, format string, values ...interface{})

// response with json format
func (c *Context) JSON(code int, obj interface{})

// response with HTML format
func (c *Context) HTML(code int, html string)
```
对 `http responseWriter` 的操作也进行了封装,
可以快速方便地创建一个回复

# trie
为了支持动态路由, 使用 `trie` 前缀树进行路由匹配
![[Pasted image 20231210131319.png]]
每一个节点都有自己的 `part`, 只有 `endpoint` 节点才有 `pattern`
`trie` 提供的功能其实非常简单, 只是给路由的每一段都作为一个节点, 形成树形结构(因此支持路由一层层匹配, 从而支持动态路由), 并提供了简单地 `insert`, `search` 接口

# router
![[Pasted image 20231210131722.png]]
1. 包含所有注册的 `endpoint` 的 `handler`, 让一个访问某个 `endpoint` 的 `request` 可以获取到对应 `handler`
2. 包含所有类型的 `request` 的对应前缀树根节点, 以便路由匹配

提供 `addRoute` 接口, 新增 `endpoint` 和对应的 `handler`
提供 `getRoute` 接口, 根据给定的 `method+pattern`, 返回这个 `pattern` 对应的 `endpoint trie node` 以及动态解析出来的参数(如果有动态解析)
```go
// add a route
func (r *router) addRoute(method string, pattern string, handler HandlerFunc)

// Get node and dynamic resolve result of a request URL
func (r *router) getRoute(method string, path string) (*node, map[string]string)
```

最后提供 `handle` 接口, 作为处理一个 `request` 的入口
```go
// handle incomming requests
// handler to this request would be set as the last of c.handlers
// right after middlewares and call c.Next() to begin handle
// Params would be set into Context
// so that handler functions would be able to
// get dynamic resolve result through Context.
func (r *router) handle(c *Context) {
  
  // get endpoint node and dynamic params
  n, params := r.getRoute(c.Method, c.Path)
  if n != nil {
    // set params into Context for handlers' accesses
    c.Params = params
  
    // append the actual handler as the last c.handlers
    // right after middlewares
    key := c.Method + "-" + n.pattern
    c.handlers = append(c.handlers, r.handlers[key])
  } else {
    c.handlers = append(c.handlers, func(ctx *Context) {
      c.String(http.StatusNotFound, "404 NOT FOUND: %s\n", c.Path)
    })
  }
  
  // activate the first middleware and begin hanlde
  c.Next()
}
```

# logger
一个简单的自定义中间件, 记录一些简单的处理过程信息
```go
// logger middleware
func Logger() HandlerFunc {
  return func(ctx *Context) {
    // start a timer
    t := time.Now()
  
    // begin handle
    ctx.Next()
  
    // print the handle log(status, request URI and time usage here)
    log.Printf("[%d] %s in %v", ctx.StatusCode, ctx.Req.RequestURI, time.Since(t))
  }
}
```

# RouterGroup
![[Pasted image 20231210133833.png]]
`RouterGroup` 就是将一组拥有公共前缀的 `endpoint` 集合起来(因为一般同一组中的 `endpoint` 会有类似的行为, 中间件的使用需求也应该相同, 因此以组为单位添加中间件, 如果一个一个 `endpoint` 添加中间件就太麻烦了)

`RouterGroup` 提供了很多接口, 因此基本上在路由这方面一个 `RouterGroup` 就是一个小生态
```go
// create a new group under a existing group
func (group *RouterGroup) Group(prefix string) *RouterGroup {
  engine := group.engine
  newGroup := &RouterGroup{
    prefix: group.prefix + prefix,
    engine: engine,
  }
  engine.groups = append(engine.groups, newGroup)
  
  return newGroup
}
  
// add middlewares to the RouterGroup
// each RouterGroup may use different middlewares
func (group *RouterGroup) Use(middlewares ...HandlerFunc) {
  group.middlewares = append(group.middlewares, middlewares...)
}
  
// add a route under the group
func (group *RouterGroup) addRoute(method string, comp string, handler HandlerFunc) {
  pattern := group.prefix + comp
  group.engine.router.addRoute(method, pattern, handler)
}
  
func (group *RouterGroup) GET(pattern string, handler HandlerFunc) {
  group.addRoute(GET, pattern, handler)
}
  
func (group *RouterGroup) POST(pattern string, handler HandlerFunc) {
  group.addRoute(POST, pattern, handler)
}
```

# Engine
`Engine` 是最重要的结构, 框架的入口,
![[Pasted image 20231210134517.png]]
`Engine` 拥有一个匿名的, 作用于根节点的 `RouterGroup`
以及指向 `router` 的指针, 以及其他的所有 `sub-RouterGroup`

`Engine` 提供的 `ServeHTTP` 方法让他实现了 `http.Handler` 接口,
可以用于 `http.ListenAndServe`, 将所有的 `request` 都拦进 `ServeHTTP` 中慢慢处理
![[Pasted image 20231210134852.png]]

# main
框架的简单使用
```go
func main() {
  e := gee.New()
  
  // global logger middleware
  e.Use(gee.Logger())
  e.GET("/index", func(ctx *gee.Context) {
    ctx.HTML(http.StatusOK, "<h1>Index Page</h1>\n")
  })
  
  v1 := e.Group("/v1")
  {
    v1.GET("/", func(ctx *gee.Context) {
      ctx.HTML(http.StatusOK, "<h1>hello v1 group\n</h1>")
    })
    v1.GET("/hello", func(ctx *gee.Context) {
      // e.g. /hello?name=gee
      ctx.String(http.StatusOK, "hello %s, you're at %s\n", ctx.Query("name"), ctx.Path)
    })
  }
  
  v2 := e.Group("/v2")
  // middleware only fo v2
  v2.Use(onlyForV2())
  {
    v2.GET("/", func(ctx *gee.Context) {
      ctx.String(http.StatusOK, "hello v2 group")
    })
    v2.GET("/hello/:name", func(ctx *gee.Context) {
      // e.g. /hello/gee
      ctx.String(http.StatusOK, "hello %s, you're at %s\n", ctx.Param("name"), ctx.Path)
    })
  }
  
  e.Run(":9999")
}
```

```ad-summary
router, context, routerGroup, engine都很重要
```

# recovery
从错误中恢复过来是很重要的功能, 比如处理 `request` 过程中可能会被动触发 `panic`, 此时应该从 `panic` 中跳出并返回错误, 而不是让系统宕机

`panic` 机制: 中断程序, 但在退出前会先将当前 `routine` 中已经 `defer` 的任务先执行, 因此可以利用这个特性在 `defer` 函数中添加一些恢复机制

`recover()`: 内置函数, 可以避免因为 `panic` 而终止整个程序的运行, 只在 `defer` 中生效
```go
// hello.go  
func test_recover() {  
	defer func() {  
		fmt.Println("defer func")  
		if err := recover(); err != nil {  
			fmt.Println("recover success")  
		}  
	}()  
  
	arr := []int{1, 2, 3}  
	fmt.Println(arr[4])  
	fmt.Println("after panic")  
}  
  
func main() {  
	test_recover()  
	fmt.Println("after recover")  
}

$ go run hello.go   
defer func  
recover success  
after recover
```
捕获到 `panic` 后, 程序执行权交到了 `defer func()` 中, 执行 `defer func` 就返回, 不会继续在原来触发 `panic` 的位置继续执行下去

新编写一个中间件 `Recovery`, 在全局使用, 这样在真正处理的 `handler` 抛出 `panic` 时, 就能捕获返回错误, 而不是程序中断, 服务器宕机


