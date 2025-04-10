# Basic types (build-in types)

```go
bool
string
int int8 int16 int32 int64
uint uint8 uint16 uint32 uint64
byte // equals to unit8
rune // equals to int32, a unicode character
float32 float64
complex64 complex128
```

使用 `fmt.Printf()` 控制格式打印时, `%T` 是类型, `%v` 是值
```go
package main

import (
	"fmt"
	"math/cmplx"
)

var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}

```
`fmt.Sprintf()` 与 `fmt.Printf()` 类似, 都可以用来格式化字符串, 
但返回格式化后的字符串, 而不是将字符串打印

类型转换:
使用不同类型的变量给别的变量赋值时, 必须使用显式类型转换 `T(v)`

常量:
类似变量, 但使用 `const` 关键字定义, 因此不能用简洁定义
```go
package main

import "fmt"

const (
	Pi    = 3.14
	World = "World"
)

func main() {
	const Truth = true

	fmt.Println(Pi, World, Truth)
}

```
常量也可以通过计算得到(在编译时期就能得出结果的计算)
```go
package main

import "fmt"

func main() {
	const secondsInMinute = 60
	const minutesInHour = 60
	const secondsInHour = secondsInMinute * minutesInHour// ?

	fmt.Println("number of seconds in an hour:", secondsInHour)
}

```


for:
只有这一种循环, 没有小括号, 但大括号是必须的
初始化和后置语句是可选的, 但判断是必选的(也可以删除, 删除后是死循环)  
如果没有初始化和后置语句, 可以把分号删掉, 只剩判断语句, 此时类似 while 语句
```go
package main

import "fmt"

func main() {
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)

	sum = 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}

```

if:
与 C 类似, 但可以在 if 最开始添加一个简洁赋值语句, 该变量的作用范围在 if-else 语句内
```go
package main

import (
	"fmt"
	"math"
)

func s(x float64) string {
	if x < 0 {
		return s(-x) + "i"
	}
	return fmt.Sprint(math.Sqrt(x))
}

func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	} else if{
	...
	} else {
		fmt.Printf("%g>=%g\n", v, lim)
	}
	return lim
}

func main() {
	fmt.Println(s(2), s(-4), pow(11, 2, 100))
}

```

switch:
自动在每个 case 内包含了 break, 因此如果需要不断执行则需要用 fallthrough 语句接触 break 
```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Print("Runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		fmt.Printf("%s.\n", os)
	}
}

```

没有条件的 switch:  
就是 switch true, 它只会检查 case 中的 T/F, 就像是一长串的 if-then-else
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning.")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
}

```

defer:
作用于函数调用, 会将函数推迟到外层函数(也就是这个函数的调用者即将返回时执行, 但是它的参数会立即求值)  
所有被 defer 的函数都会被压入一个栈, 执行时按出栈顺序执行
```go
package main

import (
	"fmt"
)

func main() {
	for i := 0; i < 10; i++ {
		defer fmt.Println("defer func", i)
	}
	fmt.Println("main func")
}

```

指针:
Go 拥有指针, 但与 C 不同, 不支持指针运算  
指针 0 值为 nil
```go
package main

import (
	"fmt"
)

func main() {
	i, j := 22, 2701

	p := &i
	fmt.Println(*p)
	*p = 21
	fmt.Println(i)

	p = &j
	*p = *p / 37
	fmt.Println(j)
}

```
