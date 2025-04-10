在 Go 语言中，`encoding/json` 包提供了对 JSON 数据的编码和解码功能。它允许将 Go 语言中的数据结构与 JSON 之间进行转换，使得在处理 JSON 数据时变得更加方便和高效。以下是 `encoding/json` 包的一些常用功能和示例：

**1. 结构体与 JSON 之间的转换：**

可以使用 `json.Marshal` 和 `json.Unmarshal` 函数将 Go 语言中的结构体与 JSON 之间进行相互转换。

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Person struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func main() {
	// 结构体转换为JSON
	person := Person{Name: "John Doe", Age: 30}
	jsonData, err := json.Marshal(person)
	if err != nil {
		fmt.Println("JSON编码错误:", err)
		return
	}
	fmt.Println(string(jsonData)) // Output: {"name":"John Doe","age":30}

	// JSON转换为结构体
	var newPerson Person
	err = json.Unmarshal(jsonData, &newPerson)
	if err != nil {
		fmt.Println("JSON解码错误:", err)
		return
	}
	fmt.Println(newPerson) // Output: {John Doe 30}
}
```

在上面的示例中，我们定义了一个 `Person` 结构体，并使用 `json.Marshal` 将其转换为 JSON 字符串。然后，使用 `json.Unmarshal` 将 JSON 字符串转换为 `Person` 结构体。

**2. 自定义 JSON 字段标签：**

可以使用结构体字段的标签来自定义 JSON 的字段名。

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Person struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func main() {
	person := Person{Name: "John Doe", Age: 30}
	jsonData, err := json.Marshal(person)
	if err != nil {
		fmt.Println("JSON编码错误:", err)
		return
	}
	fmt.Println(string(jsonData)) // Output: {"name":"John Doe","age":30}
}
```

在上面的示例中，我们使用 `json:"name"` 和 `json:"age"` 标签来定义 `Person` 结构体字段在 JSON 中的字段名。

**3. 处理嵌套结构体和嵌套 JSON 数据：**

`encoding/json` 包可以处理嵌套的结构体和嵌套的 JSON 数据。

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Address struct {
	City  string `json:"city"`
	State string `json:"state"`
}

type Person struct {
	Name    string  `json:"name"`
	Age     int     `json:"age"`
	Address Address `json:"address"`
}

func main() {
	person := Person{
		Name: "John Doe",
		Age:  30,
		Address: Address{
			City:  "New York",
			State: "NY",
		},
	}

	jsonData, err := json.Marshal(person)
	if err != nil {
		fmt.Println("JSON编码错误:", err)
		return
	}
	fmt.Println(string(jsonData))
	// Output: {"name":"John Doe","age":30,"address":{"city":"New York","state":"NY"}}
}
```

在上面的示例中，我们定义了一个 `Person` 结构体，其中包含一个嵌套的 `Address` 结构体。通过使用嵌套结构体，我们可以处理复杂的 JSON 数据。

以上只是 `encoding/json` 包的一些基本功能和示例。该包还提供了其他功能，如处理 JSON 数组、空值、忽略字段、自定义类型的编码和解码方法等。通过使用 `encoding/json` 包，可以轻松地在 Go 语言中进行 JSON 数据的序列化和反序列化操作。