在这样一个多用户多 rss feeds 的系统中, 用户比较关心的一般都是自己关注(或者说自己往 feeds 中添加的 feed ), 因此新建一个 feed_follows 表, 用来表示用户与其关注的 feeds 之间的联系
```sql
CREATE TABLE feed_follows (
    id UUID PRIMARY KEY,
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP NOT NULL,
    user_id UUID NOT NULL REFERENCES users (id) ON DELETE CASCADE,
    feed_id UUID NOT NULL REFERENCES feeds (id) ON DELETE CASCADE,
    UNIQUE (user_id, feed_id)
);
```
有了这个表后, 就能通过 user_id 查询对应的这个用户关注的 rss feeds
(通过 goose up/down 创表或删表)

常规操作:
插入记录和查询
```sql
-- name: CreateFeedFollow :one
INSERT INTO feed_follows (id, created_at, updated_at, feed_id, user_id)
VALUES ($1, $2, $3, $4, $5)
RETURNING *;
  
-- name: GetFeedFollows :many
SELECT * FROM feed_follows WHERE user_id = $1;
```
(通过 sqlc 生成对应的 go 接口)
有了 sqlc 生成的接口之后, 就可以方便地写对应 `request` 的 `handler` 了
(从 http request 的 header 或 body 中得到必要的数据, 之后操作就跟之前几个 handler 类似)


删除记录:
删除记录中需要用到两个数据:
feed_follow id 和 user_id
user_id 可以在 authentication 时通过 request header 获得
但是如果按照之前的做法, feed_follow id 就只能通过 request body 用 JSON 格式传递, 这样做不是很方便:
问了下 gpt:
url 传递:
优点: 可以减轻 http request 的负载, 符合 RESTful api 的设计理念(资源必须放在 url 中), 可以更加方便地获取信息(从 url 提取要更简单)
缺点: id 完全暴露在 url 中, 如果是敏感信息那么将会有风险, url 长度有限

http request body 传递:
优点: 更安全, 可以同时传递更多信息
缺点: 不 RESTful, 让 request 负载加大, 从 body 中提取 id 会更复杂一些

因此, 综上考虑, 在 delete 中使用了 url 传递 feed_follow id 的方式
```go
func (apiCfg *apiConfig) handlerDeleteFeedFollow(w http.ResponseWriter, r *http.Request, user database.User) {
  feedFollowIDStr := chi.URLParam(r, "feedFollowID")
  feedFollowID, err := uuid.Parse(feedFollowIDStr)
  if err != nil {
    respondWithError(w, 400, fmt.Sprintf("error parsing feed follow ID: %v", err))
    return
  }
  
  err = apiCfg.DB.DeleteFeedFollow(r.Context(), database.DeleteFeedFollowParams{
    ID:     feedFollowID,
    UserID: user.ID,
  })
  if err != nil {
    respondWithError(w, 400, fmt.Sprintf("error deleting feed_follow: %v", err))
    return
  }
  
  respondWithJSON(w, 200, struct{}{})
}
```
使用 chi.URLParam 就可提取出 url 中的 id
```go
v1Router.Delete("/feed_follows/{feedFollowID}", apiCfg.middlewareAuth(apiCfg.handlerDeleteFeedFollow))
```
