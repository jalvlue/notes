![[code.png]]

1. router: 字面意思就是路由器, 在整个 `http server` 中的作用是作为一个中转站, 为每一个 `http request` 根据他们的特点, 链接到对应的 `handler` 进行处理
2. http.Server: 就是项目的核心 `http server`, 这里用了 `chi-router` 作为 `Handler`, 并定义了地址为设置好的 `port number`

有了这个基础之后就可以具体地为特定的 `http request` 调用特定的 `handler` 处理 request 了

![[code1.png]]
1. CORS: cross-origin resource sharing, 是一种机制, 允许 `client` 进行 cross-origin requests 访问不同域名下的 `server`
2. router.Use(cors.Handler()): cors 是一个 go CORS 中间件, 定义了一些访问规则(图中的例子使用了最宽松的规则, 几乎允许任何形式的访问), Use 方法将这个中间件用来配置 router 
3. 然后创建了一个新的 `v1Router`, 使用 Mount 方法挂载在主 router 之下
4. 在 `v1Router` 之下, 定义了对 `v1/healthz`, `v1/err` 路径的 GET request 的路由, 分配到对应的 `handler` 中进行处理

至此, 基本框架已经搭建完毕, server 正常运行时, 可以正常处理 request

附:
![[code2.png]]
![[code3.png]]
![[code4.png]]

