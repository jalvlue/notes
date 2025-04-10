完成了以上之后, 就正式进入真正的业务逻辑了
rss 订阅需要有来源(feeds), 因此需要建表存 feeds
```sql

-- +goose Up
CREATE TABLE feeds(
  id UUID PRIMARY KEY,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL,
  name TEXT NOT NULL,
  url TEXT UNIQUE NOT NULL,
  
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
);

-- +goose Down
DROP TABLE feeds;

```
值得注意的是:
`user_id` 是一个外键, 引用了 `users(id)`, 而且 `on delete cascade`, 级联, 也就是对应的 user 被删除时, `feeds` 表中对应属于这个 user 的记录也会一起删除

在这里, 新增 feeds 的 handler 中也需要 authentication, 因此可以把这个逻辑抽象一下作为一个中间件

新增 feeds 中, 需要使用 apikey 跟踪是哪个用户新增的 feed, 因此将通过 apikey 获取用户抽象成中间件,
将具体进行业务操作的函数作为参数传进中间件,
在中间件获取到 user 后, 再调用具体业务函数
```go
type authedHandler func(http.ResponseWriter, *http.Request, database.User)
  
func (apiCfg *apiConfig) middlewareAuth(handler authedHandler) http.HandlerFunc {
  return func(w http.ResponseWriter, r *http.Request) {
    apiKey, err := auth.GetApiKey(r.Header)
    if err != nil {
      respondWithError(w, 403, fmt.Sprintf("auth error: %v", err))
      return
    }
  
    user, err := apiCfg.DB.GetUserByAPIKey(r.Context(), apiKey)
    if err != nil {
      respondWithError(w, 400, fmt.Sprintf("couldn't get user: %v", err))
      return
    }

    handler(w, r, user)
  }
}
```

新增 feed 只需要在请求的请求体中包含新增的 feed 的名字以及对应的 url 即可(请求头还需包含 apikey 来做 auth)

其他的也都类似, 具体的, 往数据库插入数据的函数已经使用 sqlc 写好了, 直接调用即可
```go
func (apiCfg *apiConfig) handlerCreateFeed(w http.ResponseWriter, r *http.Request, user database.User) {
  type parameters struct {
    Name string `json:"name"`
    Url  string `json:"url"`
  }
  decoder := json.NewDecoder(r.Body)
  
  params := parameters{}
  err := decoder.Decode(&params)
  if err != nil {
    respondWithError(w, 400, fmt.Sprintf("error parsing JSON: %v", err))
  }  

  feed, err := apiCfg.DB.CreateFeed(r.Context(), database.CreateFeedParams{
    ID:        uuid.New(),
    CreatedAt: time.Now().UTC(),
    UpdatedAt: time.Now().UTC(),
    Name:      params.Name,
    Url:       params.Url,
    UserID:    user.ID,
  })
  if err != nil {
    respondWithError(w, 400, fmt.Sprintf("error creating feed: %v", err))
    return
  }
  
  respondWithJSON(w, 201, databaseFeedToFeed(feed))
}
```

最后再将这个 handler 挂载到对应的路由上
```go
v1Router.Post("/feeds", apiCfg.middlewareAuth(apiCfg.handlerCreateFeed))
```

查询 feeds:
类似地, 查询操作不需要从 request 中获取什么, 
只需要调用 sqlc 生成的函数查询然后返回就好了