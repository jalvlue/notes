`Gin` 是一个 web 开发框架

Demo:
```go
package main
  
import "github.com/gin-gonic/gin"
  
func main() {
  r := gin.Default()
  r.GET("/ping", func(c *gin.Context) {
    c.JSON(200, gin.H{"message": "pong"})
  })
  r.Run()
}
```
在这个例子中, 首先使用 `gin.Default()` 创建了一个类似 `net/http` 中 server 的 `gin.Engine`,
并在这个 `*Engine r` 中注册了 `/ping` 路径下的 `GET` 方法对应的 `handler` (返回一个字典 `{"message": "pong"}` )
最后使用 `r.Run()` 开始运行 `Engine` 并在 `8080` 端口中监听

Gin 源码:
```go
type Engine struct {
	RouterGroup
	...
}

func New() *Engine {
  debugPrintWARNINGNew()
  engine := &Engine{...} // 省略n串默认参数
  engine.RouterGroup.engine = engine
  engine.pool.New = func() any {
    return engine.allocateContext(engine.maxParams)
  }
  return engine
}

func Default() *Engine {
  debugPrintWARNINGDefault()
  engine := New()
  engine.Use(Logger(), Recovery())
  return engine
}
```
```ad-summary
gin 中的 Engine 是最重要的一个数据结构
Engine 中的匿名 RouterGroup 是 Engine 结构中最重要的组成

New()和Default()函数则是创建Engine的函数, gin的入口所在

值得一提的是, 用 Use() 方法注册的中间件 Logger, Recovery 也属于是 handler
```


```go
func (group *RouterGroup) combineHandlers(handlers HandlersChain) HandlersChain {
  finalSize := len(group.Handlers) + len(handlers)
  assert1(finalSize < int(abortIndex), "too many handlers")
  mergedHandlers := make(HandlersChain, finalSize)
  copy(mergedHandlers, group.Handlers)
  copy(mergedHandlers[len(group.Handlers):], handlers)
  return mergedHandlers
}

func (group *RouterGroup) handle(httpMethod, relativePath string, handlers HandlersChain) IRoutes {
  absolutePath := group.calculateAbsolutePath(relativePath)
  handlers = group.combineHandlers(handlers)
  group.engine.addRoute(httpMethod, absolutePath, handlers)
  return group.returnObj()
}

func (group *RouterGroup) GET(relativePath string, handlers ...HandlerFunc) IRoutes {
  return group.handle(http.MethodGet, relativePath, handlers)
}
func POST
func DELETE
...
```
```ad-summary
在这里就可以看出 Engine 中匿名成员 RouterGroup 的重要之处,
RouterGroup 用来保存所有注册在 Engine 中的 handler

每次使用 GET, POST 这些方法, 就是在注册新的 handler

同时, 由于 RouterGroup 是 Engine 的匿名成员,
因此可以直接使用 Engine 调用 RouterGroup 的方法
例如 demo 中的 r.GET
r 是一个 *Engine
```

```go
func (engine *Engine) Run(addr ...string) (err error) {
  defer func() { debugPrintError(err) }()
  
  if engine.isUnsafeTrustedProxies() {
    debugPrint("[WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.\n" +
      "Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.")
  }
  
  address := resolveAddress(addr)
  debugPrint("Listening and serving HTTP on %s\n", address)
  err = http.ListenAndServe(address, engine.Handler())
  return
}
```
```ad-summary
最后调用的 r.Run() 就调用了 net/http 标准库中的 ListenAndServe,
开始监听请求, 并由 Engine.Handler() 处理
```

可以看出 `Gin` 就是对 `net/http` 的一个很好的封装
省去了很多繁杂的东西

# API Examples
###### GET, POST, PUT, PATCH...
```go
func main() {
  // Creates a gin router with default middleware:
  // logger and recovery (crash-free) middleware
  router := gin.Default()

  router.GET("/someGet", getting)
  router.POST("/somePost", posting)
  router.PUT("/somePut", putting)
  router.DELETE("/someDelete", deleting)
  router.PATCH("/somePatch", patching)
  router.HEAD("/someHead", head)
  router.OPTIONS("/someOptions", options)

  // By default it serves on :8080 unless a
  // PORT environment variable was defined.
  router.Run()
  // router.Run(":3000") for a hard coded port
}
```

###### 路径参数
定义路由时使用 `:paramName` 代表参数, 使用 `*paramName` 截断成一个参数
在 handler 中, 如果要获取对应参数, 就使用 `ctx.Param("paramName")` 从路径中将参数读取出来
```go
func main() {
  router := gin.Default()

  // This handler will match /user/john but will not match /user/ or /user
  router.GET("/user/:name", func(c *gin.Context) {
    name := c.Param("name")
    c.String(http.StatusOK, "Hello %s", name)
  })

  // However, this one will match /user/john/ and also /user/john/send
  // If no other routers match /user/john, it will redirect to /user/john/
  router.GET("/user/:name/*action", func(c *gin.Context) {
    name := c.Param("name")
    action := c.Param("action")
    message := name + " is " + action
    c.String(http.StatusOK, message)
  })

  // For each matched request Context will hold the route definition
  router.POST("/user/:name/*action", func(c *gin.Context) {
    b := c.FullPath() == "/user/:name/*action" // true
    c.String(http.StatusOK, "%t", b)
  })

  // This handler will add a new router for /user/groups.
  // Exact routes are resolved before param routes, regardless of the order they were defined.
  // Routes starting with /user/groups are never interpreted as /user/:name/... routes
  router.GET("/user/groups", func(c *gin.Context) {
    c.String(http.StatusOK, "The available groups are [...]")
  })

  router.Run(":8080")
}
```

###### 查询参数
路径上没什么不同, 只是使用 `ctx.Query("queryKey")` 从 URL 中读出对应的值
```go
func main() {
  router := gin.Default()

  // Query string parameters are parsed using the existing underlying request object.
  // The request responds to an url matching:  /welcome?firstname=Jane&lastname=Doe
  router.GET("/welcome", func(c *gin.Context) {
    firstname := c.DefaultQuery("firstname", "Guest")
    lastname := c.Query("lastname") // shortcut for c.Request.URL.Query().Get("lastname")

    c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
  })
  router.Run(":8080")
}
```

使用 `POST` 的查询参数
```go
func main() {
  router := gin.Default()

  router.POST("/form_post", func(c *gin.Context) {
    message := c.PostForm("message")
    nick := c.DefaultPostForm("nick", "anonymous")

    c.JSON(http.StatusOK, gin.H{
      "status":  "posted",
      "message": message,
      "nick":    nick,
    })
  })
  router.Run(":8080")
}
```
`curl -X POST -d "message=Hello&nick=John" http://localhost:8080/form_post`

混合的 `POST`:
```go
func main() {
  router := gin.Default()

  router.POST("/post", func(c *gin.Context) {

    id := c.Query("id")
    page := c.DefaultQuery("page", "0")
    name := c.PostForm("name")
    message := c.PostForm("message")

    fmt.Printf("id: %s; page: %s; name: %s; message: %s", id, page, name, message)
  })
  router.Run(":8080")
}
```
```
POST /post?id=1234&page=1 HTTP/1.1
Content-Type: application/x-www-form-urlencoded

name=manu&message=this_is_great

>>>

id: 1234; page: 1; name: manu; message: this_is_great
```

###### grouping routes
```go
func main() {
  router := gin.Default()

  // Simple group: v1
  v1 := router.Group("/v1")
  {
    v1.POST("/login", loginEndpoint)
    v1.POST("/submit", submitEndpoint)
    v1.POST("/read", readEndpoint)
  }

  // Simple group: v2
  v2 := router.Group("/v2")
  {
    v2.POST("/login", loginEndpoint)
    v2.POST("/submit", submitEndpoint)
    v2.POST("/read", readEndpoint)
  }

  router.Run(":8080")
}
```

###### 中间件
中间件实际上就是在最终的 `handler` 之前的 `HandlerFunc`, 他们的函数签名都是一样的, 只是做一些中间处理, 比如将请求中的一些信息提前拿出来保存在 Context 中(auth token)等等

使用 `gin.New()` 可以新建一个没有中间件的路由
使用 `gin.Use(handler)` 将某个 handler 注册为中间件
```go
func main() {
  // Creates a router without any middleware by default
  r := gin.New()

  // Global middleware
  // Logger middleware will write the logs to gin.DefaultWriter even if you set with GIN_MODE=release.
  // By default gin.DefaultWriter = os.Stdout
  r.Use(gin.Logger())

  // Recovery middleware recovers from any panics and writes a 500 if there was one.
  r.Use(gin.Recovery())

  // Per route middleware, you can add as many as you desire.
  r.GET("/benchmark", MyBenchLogger(), benchEndpoint)

  // Authorization group
  // authorized := r.Group("/", AuthRequired())
  // exactly the same as:
  authorized := r.Group("/")
  // per group middleware! in this case we use the custom created
  // AuthRequired() middleware just in the "authorized" group.
  authorized.Use(AuthRequired())
  {
    authorized.POST("/login", loginEndpoint)
    authorized.POST("/submit", submitEndpoint)
    authorized.POST("/read", readEndpoint)

    // nested group
    testing := authorized.Group("testing")
    // visit 0.0.0.0:8080/testing/analytics
    testing.GET("/analytics", analyticsEndpoint)
  }

  // Listen and serve on 0.0.0.0:8080
  r.Run(":8080")
}
```

###### Model binding and validation
对于 model binding 处理 `json, xml, yaml` 等数据格式的转换, 使用标签和 golang 提供的 `should bind, must bind` 进行处理

`Context.ShouldBind` 方法族是 Gin 框架提供的一组方法，用于将 HTTP 请求中的数据绑定到结构体实例中。通过在结构体字段上使用标签（如 `form`、`json`、`xml` 等），Gin 可以自动解析不同格式的数据（如表单数据、JSON、XML 等）并将其转换为对应的结构体类型。
这种方式相比于手动解析和提取参数更加方便和简洁。您可以在定义结构体时使用合适的标签，然后通过 `Context.ShouldBind` 方法将请求数据直接绑定到结构体实例中，而无需逐个提取和转换参数。

这样做的好处是：
- 简化了参数提取和转换的过程，减少了重复的代码。
- 使代码更加可读和易于维护，通过结构体的字段可以清晰地看出需要提取的参数和它们的类型。
- 提供了对多种数据格式的支持，无论是表单数据、JSON、XML 还是其他格式，您只需定义对应的结构体和标签即可。
总而言之，`Context.ShouldBind` 方法族是 Gin 框架中用于简化参数提取和转换的强大工具，使得处理 HTTP 请求的代码更加简洁和可读。

对于验证, 例如某个字段是必须的, 可以在标签中加上 `binding:"required"`
[[validator]]

完整例子
```go
// Binding from JSON
type Login struct {
  User     string `form:"user" json:"user" xml:"user"  binding:"required"`
  Password string `form:"password" json:"password" xml:"password" binding:"required"`
}

func main() {
  router := gin.Default()

  // Example for binding JSON ({"user": "manu", "password": "123"})
  router.POST("/loginJSON", func(c *gin.Context) {
    var json Login
    if err := c.ShouldBindJSON(&json); err != nil {
      c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
      return
    }

    if json.User != "manu" || json.Password != "123" {
      c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
      return
    }

    c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
  })

  // Example for binding XML (
  //  <?xml version="1.0" encoding="UTF-8"?>
  //  <root>
  //    <user>manu</user>
  //    <password>123</password>
  //  </root>)
  router.POST("/loginXML", func(c *gin.Context) {
    var xml Login
    if err := c.ShouldBindXML(&xml); err != nil {
      c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
      return
    }

    if xml.User != "manu" || xml.Password != "123" {
      c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
      return
    }

    c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
  })

  // Example for binding a HTML form (user=manu&password=123)
  router.POST("/loginForm", func(c *gin.Context) {
    var form Login
    // This will infer what binder to use depending on the content-type header.
    if err := c.ShouldBind(&form); err != nil {
      c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
      return
    }

    if form.User != "manu" || form.Password != "123" {
      c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
      return
    }

    c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
  })

  // Listen and serve on 0.0.0.0:8080
  router.Run(":8080")
}
```


# Context
当然！以下是一些使用 Gin 的 `Context` 对象的高级用法示例：

1. 参数解析：
   ```go
   func handler(c *gin.Context) {
       queryParam := c.Query("param")
       postFormParam := c.PostForm("param")
       routeParam := c.Param("param")
       // ...
   }
   ```

2. JSON 绑定：
   ```go
   type User struct {
       Name  string `json:"name"`
       Email string `json:"email"`
   }

   func handler(c *gin.Context) {
       var user User
       if err := c.ShouldBindJSON(&user); err == nil {
           // 处理绑定后的user对象
           // ...
       } else {
           // 处理绑定错误
           // ...
       }
   }
   ```

3. 文件上传：
   ```go
   func handler(c *gin.Context) {
       file, err := c.FormFile("file")
       if err != nil {
           // 处理文件上传错误
           // ...
       } else {
           // 保存上传的文件到指定位置
           err = c.SaveUploadedFile(file, "uploads/"+file.Filename)
           if err != nil {
               // 处理文件保存错误
               // ...
           } else {
               // 文件保存成功
               // ...
           }
       }
   }
   ```

4. 中间件：
   ```go
   func AuthMiddleware() gin.HandlerFunc {
       return func(c *gin.Context) {
           // 在此处进行身份验证逻辑
           // ...
           if authenticated {
               c.Next() // 继续处理请求
           } else {
               c.AbortWithStatus(http.StatusUnauthorized) // 终止请求并返回401状态码
           }
       }
   }

   func handler(c *gin.Context) {
       // 处理受身份验证保护的请求
       // ...
   }

   // 使用中间件
   router := gin.Default()
   router.Use(AuthMiddleware())
   router.GET("/protected", handler)
   ```

5. 错误处理：
   ```go
   func handler(c *gin.Context) {
       if someError {
           c.Error(errors.New("something went wrong"))
           c.AbortWithStatus(http.StatusInternalServerError)
           return
       }
       // 处理正常情况
       // ...
   }
   ```

6. Cookie 和 Session：
   ```go
   func handler(c *gin.Context) {
       // 设置Cookie
       c.SetCookie("user", "john", 3600, "/", "example.com", false, true)

       // 获取Cookie
       cookie, err := c.Cookie("user")
       if err == nil {
           // 处理Cookie值
           // ...
       }

       // 设置Session
       session := sessions.Default(c)
       session.Set("key", "value")
       session.Save()

       // 获取Session
       session := sessions.Default(c)
       value := session.Get("key")
       // ...
   }
   ```

这些示例涵盖了 Gin 的 `Context` 对象的一些高级用法，您可以根据具体需求进行修改和扩展。使用这些功能，您可以更灵活和高效地处理请求和响应。

### 在上下文中传递值
在 Gin 框架中，`Context.Set` 和 `Context.MustGet` 是用于在请求处理过程中存储和检索数据的方法。

1. `Context.Set`：
   `Context.Set` 方法用于在当前请求的上下文中存储键值对数据。这些数据可以是任何类型的，例如字符串、整数、结构体等。您可以使用该方法将数据存储在 `Context` 对象中，并在当前请求的处理过程中访问这些数据。存储的数据对于当前请求是唯一的，不会影响其他请求的上下文。

   以下是一个示例，展示如何使用 `Context.Set` 方法存储和检索数据：
   ```go
   func handler(c *gin.Context) {
       c.Set("userID", 123)
       // ...
       userID := c.GetInt("userID")
       // 使用存储的userID进行其他操作
       // ...
   }
   ```

2. `Context.MustGet`：
   `Context.MustGet` 方法用于从当前请求的上下文中检索存储的数据。与 `Context.Get` 方法不同的是，`Context.MustGet` 方法会在找不到指定键的值时触发恐慌（panic）。这意味着您必须确保在使用 `Context.MustGet` 方法之前已经使用 `Context.Set` 方法存储了相应的数据。这种方法适用于那些必须存在的数据，如果数据不存在，应该引发错误。

   以下是一个示例，展示如何使用 `Context.MustGet` 方法检索存储的数据：
   ```go
   func handler(c *gin.Context) {
       userID := c.MustGet("userID").(int)
       // 使用userID进行其他操作
       // ...
   }
   ```

请注意，使用 `Context.Set` 和 `Context.MustGet` 方法存储和检索数据时，需要在同一个请求的处理过程中进行操作。这些方法仅在当前请求的上下文范围内有效，并且不适用于跨多个请求之间的数据传递。