在 Go 编程语言中，`encoding/csv` 包提供了用于读取和写入 CSV（逗号分隔值）文件的功能。CSV 文件是一种常见的数据交换格式，通常用于存储表格数据。

```go
type Reader struct {
	// seperate character, default to ','
	Comma rune

	// comment character
	Comment rune
	...
}
func NewReader(r io.Reader) *Reader
func (r *Reader) Read() (record []string, err error)
func (r *Reader) ReadALL() (records [][]string, err error)


type Writer struct {
	Comma rune
	UserCRLF bool
}
func NewWriter(w io.Writer) *Writer
func (w *Writer) Flush()
func (w *Writer) Write(record []string) error
func (w *Writer) WriteAll(records [][]string) error
```


下面是一个简单的示例，演示了如何使用 `encoding/csv` 包来读取和写入 CSV 文件：

**读取 CSV 文件：**

```go
package main

import (
	"encoding/csv"
	"fmt"
	"os"
)

func main() {
	// 打开CSV文件
	file, err := os.Open("data.csv")
	if err != nil {
		fmt.Println("无法打开CSV文件:", err)
		return
	}
	defer file.Close()

	// 创建CSV读取器
	reader := csv.NewReader(file)

	// 读取所有记录
	records, err := reader.ReadAll()
	if err != nil {
		fmt.Println("无法读取CSV文件:", err)
		return
	}

	// 遍历记录并打印
	for _, record := range records {
		for _, value := range record {
			fmt.Print(value, " ")
		}
		fmt.Println()
	}
}
```

在上述示例中，我们首先使用 `os.Open` 函数打开要读取的 CSV 文件（这里假设文件名为"data. Csv"），然后使用 `csv.NewReader` 创建一个 CSV 读取器。接下来，我们使用 `reader.ReadAll` 读取所有的记录，并通过嵌套的循环遍历记录并打印出来。

**写入 CSV 文件：**

```go
package main

import (
	"encoding/csv"
	"os"
)

func main() {
	// 创建CSV文件
	file, err := os.Create("output.csv")
	if err != nil {
		panic(err)
	}
	defer file.Close()

	// 创建CSV写入器
	writer := csv.NewWriter(file)

	// 写入记录
	record1 := []string{"John", "Doe", "john@example.com"}
	record2 := []string{"Jane", "Smith", "jane@example.com"}

	writer.Write(record1)
	writer.Write(record2)

	// 刷新缓冲区并检查错误
	writer.Flush()
	if err := writer.Error(); err != nil {
		panic(err)
	}
}
```

在上述示例中，我们首先使用 `os.Create` 函数创建一个新的 CSV 文件（这里假设文件名为"output. Csv"），然后使用 `csv.NewWriter` 创建一个 CSV 写入器。接下来，我们使用 `writer.Write` 方法写入记录。每个记录都以字符串切片的形式表示，其中每个元素代表记录中的一个字段。最后，我们调用 `writer.Flush` 刷新缓冲区，并通过 `writer.Error` 检查是否有任何写入错误。

这是 `encoding/csv` 包的基本用法示例。您可以根据需要使用更多的功能和选项来处理 CSV 文件，例如设置分隔符、处理空字段或自定义读取和写入的逻辑。有关更详细的信息，请参阅 Go 官方文档中 `encoding/csv` 包的文档。