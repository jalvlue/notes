[[散记/uuid|uuid]]

在 golang 中, google 提供了一个 uuid 的实现

当使用 `github.com/google/uuid` 库时，以下是一些示例用法：

1. 生成随机 UUID：
   ```go
   id := uuid.New()
   fmt.Println(id.String())
   ```

2. 解析 UUID 字符串：
   ```go
   str := "9e6824a3-8dd8-47f7-8a1e-9959c2991faa"
   id, err := uuid.Parse(str)
   if err != nil {
       fmt.Println("Invalid UUID")
   } else {
       fmt.Println(id.String())
   }
   ```

3. 验证 UUID 的有效性：
   ```go
   str := "9e6824a3-8dd8-47f7-8a1e-9959c2991faa"
   isValid := uuid.IsValid(str)
   fmt.Println(isValid) // true
   ```

4. 生成基于时间戳的 UUID（版本 1）：
   ```go
   id := uuid.NewUUID()
   fmt.Println(id.String())
   ```

5. 生成空的 UUID：
   ```go
   id := uuid.Nil
   fmt.Println(id.String()) // 00000000-0000-0000-0000-000000000000
   ```

这些示例演示了一些常见的用法，包括生成随机 UUID、解析 UUID 字符串、验证 UUID 的有效性以及生成基于时间戳的 UUID。您可以根据自己的需求和具体情况使用 `github.com/google/uuid` 库中的其他功能。