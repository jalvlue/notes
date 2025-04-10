# Overview
> Package io provides basic interfaces to I/O primitives. Its primary job is to wrap existing implementations of such primitives, such as those in package os, into shared public interfaces that abstract the functionality, plus some other related primitives.
   Because these interfaces and primitives wrap lower-level operations with various implementations, unless otherwise informed clients should not assume they are safe for parallel execution.

# Constants
```go
const (
	SeekStart   = 0 // seek relative to the origin of the file
	SeekCurrent = 1 // seek relative to the current offset
	SeekEnd     = 2 // seek relative to the end
)
```


# Functions
### `func Copy(dst Writer, src Reader) (written int64, err error)`
- `dst` 是第一个参数, `src` 是第二个参数~!
- 一般情况下底层基本上封装的, 调用 `src.WriteTo(dst), dst.ReadFrom(src)`
```go
package main

import (
	"io"
	"log"
	"os"
	"strings"
)

func main() {
	r := strings.NewReader("some io.Reader stream to be read\n")

	if _, err := io.Copy(os.Stdout, r); err != nil {
		log.Fatal(err)
	}
}

```
- 类似的同族函数
	- `io.CopyBuffer(dst, src, buf)`
	- `io.CopyN(dst, src, n)`

### `func Pipe() (*PipeReader, *PipeWriter)`
- 类似管道系统调用, 可以在管道两端零缓存同步传递数据
```go
package main

import (
	"fmt"
	"io"
	"log"
	"os"
	"time"
)

func main() {
	r, w := io.Pipe()

	go func() {
		fmt.Fprint(w, "some io.Reader stream to be read\n")
		time.Sleep(2 * time.Second)
		fmt.Fprint(w, "********************************\n")
		time.Sleep(2 * time.Second)
		fmt.Fprint(w, "some io.Reader stream to be read\n")
		w.Close()
	}()

	if _, err := io.Copy(os.Stdout, r); err != nil {
		log.Fatal(err)
	}
}

```


### `func Read All(r Reader) ([]byte, error)`
- 从 `io.Reader` 中将所有数据 (`EOF` 之前的所有数据) 读取出来放到一个 `[]byte` 中
```go
package main

import (
	"fmt"
	"io"
	"log"
	"strings"
)

func main() {
	r := strings.NewReader("Go is a general-purpose language designed with systems programming in mind.")

	b, err := io.ReadAll(r)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%s\n", b)
}

```

### `func ReadAtLeast(r, buf, min)`
- 至少要读 `min bytes`, 如果不足就会抛 `io.UnexpectedEOF`
- `buf < min` 就抛 `io.ShortBuffer`
```go
package main

import (
	"fmt"
	"io"
	"log"
	"strings"
)

func main() {
	r := strings.NewReader("some io.Reader stream to be read\n")

	// ok
	buf := make([]byte, 14)
	if _, err := io.ReadAtLeast(r, buf, 4); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s\n", buf)

	// buf < min
	shortBuf := make([]byte, 3)
	if _, err := io.ReadAtLeast(r, shortBuf, 4); err != nil {
		fmt.Println("error:", err)
	}

	// reader stream < min
	longBuf := make([]byte, 64)
	if _, err := io.ReadAtLeast(r, longBuf, 64); err != nil {
		fmt.Println("error:", err)
	}
}

```

### `func WriteString(w Writer, s string) (n int, err error)`
- 向一个 `io.Writer` 中写入字符串
- 封装了 `io.StringWriter.WriteString(), io.Writer.Write()`
```go
package main

import (
	"io"
	"log"
	"os"
)

func main() {
	if _, err := io.WriteString(os.Stdout, "Hello, Go!\n"); err != nil {
		log.Fatal(err)
	}
}

```

# Types

