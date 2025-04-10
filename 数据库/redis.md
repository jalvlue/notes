# 内置数据结构
- Strings
- Sets
- Lists
- SortedSet
- Hashes
- Streams

# 发布订阅模式
`publish channel message` 命令发布
`subscribe channel` 订阅, 是一个阻塞操作, 会持续关注这个频道, 其他 `redis-cli` 在这个频道中发布的内容会被接收

# 消息队列 Stream

# Redis protocol
-  `RESP, redis serialization protocol`

Redis 协议（通常称为 RESP，Redis Serialization Protocol）是一种简单而高效的协议，用于客户端与 Redis 服务器之间的通信。以下是一些关键特性和结构：

### 1. **数据格式**
RESP 协议支持多种数据类型：
- **简单字符串**：以 `+` 开头，结尾是 `\r\n`。
  ```
  +OK\r\n
  ```
- **错误**：以 `-` 开头，结尾是 `\r\n`。
  ```
  -Error message\r\n
  ```
- **整数**：以 `:` 开头，结尾是 `\r\n`。
  ```
  :12345\r\n
  ```
- **批量字符串**：以 `$` 开头，后跟字符串长度和 `\r\n`，内容后跟 `\r\n`。
  ```
  $6\r\nfoobar\r\n
  ```
- **数组**：以 `*` 开头，后跟元素数量，内容为每个元素的 RESP 格式。
  ```
  *2\r\n$3\r\nfoo\r\n$3\r\nbar\r\n
  ```
### 2. **命令格式**
客户端发送命令时，使用数组格式表示命令及其参数。例如，`SET key value` 命令的请求格式为：
```
*3\r\n$3\r\nSET\r\n$3\r\nkey\r\n$5\r\nvalue\r\n
```

### 3. **响应格式**
服务器返回响应时，使用上述数据格式之一。例如，对于 `GET key` 命令，返回值为：
- 如果键存在：
  ```
  $5\r\nvalue\r\n
  ```
- 如果键不存在：
  ```
  $-1\r\n
  ```

### 4. **优点**
- **简单性**：协议结构简单，易于实现。
- **高效性**：能够快速解析和序列化数据，适合高性能需求。

### 5. **使用场景**
RESP 协议广泛应用于 Redis 的各种客户端库，支持多种编程语言，使得开发者可以方便地与 Redis 进行交互。

### 总结
Redis 协议（RESP）是一个高效和灵活的协议，支持多种数据类型，适用于快速的客户端与服务器通信。