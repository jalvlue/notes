https://pkg.go.dev/encoding/base64
[[散记/base64|base64]]
- util package for performing `base64` encoding/decoding
- the core package functions are `NewEncoding(), NewEncoder()`, the core data structure is `Encoding`, which performs most functionalities of package `base64`
```go
package main

import (
	"encoding/base64"
	"fmt"
)

func main() {
	msg := "Hello, 世界"
	encoded := base64.StdEncoding.EncodeToString([]byte(msg))
	fmt.Println(encoded)
	decoded, err := base64.StdEncoding.DecodeString(encoded)
	if err != nil {
		fmt.Println("decode error:", err)
		return
	}
	fmt.Println(string(decoded))
}

```

在 Go 语言中，`encoding/base64` 包提供了 Base64 编码和解码的功能。Base64 是一种将二进制数据转为 ASCII 字符串的编码方式，常用于在网络中安全地传输数据。

### 主要功能

1. **编码**：
   - 将字节切片编码为 Base64 字符串。

2. **解码**：
   - 将 Base64 字符串解码回原始的字节切片。

### 主要类型和函数

1. **常量**：
   - `StdEncoding`：标准的 Base64 编码。
   - `URLEncoding`：用于 URL 的 Base64 编码，不使用 `+` 和 `/` 字符，而是使用 `-` 和 `_`。

2. **编码函数**：
   - `EncodeToString(b []byte) string`：将字节切片 `b` 编码为 Base64 字符串。
   - `Encode(w io.Writer, src []byte)`：将字节切片 `src` 编码并写入 `w`（实现了 `io.Writer` 接口的对象）。

3. **解码函数**：
   - `DecodeString(s string) ([]byte, error)`：将 Base64 字符串 `s` 解码为字节切片。
   - `Decode(r io.Reader) ([]byte, error)`：从实现了 `io.Reader` 接口的对象 `r` 中读取 Base64 编码的数据并解码。

### 示例代码

以下是一个简单的示例，展示如何使用 `encoding/base64` 包进行编码和解码：

```go
package main

import (
    "encoding/base64"
    "fmt"
)

func main() {
    // 原始数据
    data := "Hello, World!"

    // 编码
    encoded := base64.StdEncoding.EncodeToString([]byte(data))
    fmt.Println("Encoded:", encoded)

    // 解码
    decoded, err := base64.StdEncoding.DecodeString(encoded)
    if err != nil {
        fmt.Println("Error decoding:", err)
    } else {
        fmt.Println("Decoded:", string(decoded))
    }
}
```

### 注意事项

- Base64 编码后的数据比原始数据大约增加了 33% 的大小。
- 使用 `URLEncoding` 进行 URL 安全的编码时，注意字符的变化。

### 总结

- `encoding/base64` 包在 Go 中提供了简单而高效的方式来处理 Base64 编码和解码，适用于需要传输二进制数据的场景，如电子邮件、数据存储等。
