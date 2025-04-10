结构体:
Go 中没有传统意义上的类(class), 而是使用结构体(struct)和方法(method)相结合的方式实现面向对象

用结构体封装数据
再用方法操作数据

结构体对象和指针都同一使用 `.` 运算符访问成员字段

```go
package main

import (
	"fmt"
)

type Vertex struct {
	X int
	Y int
}

func main() {
	v := Vertex{1, 2}
	fmt.Println(v)
	v.X = 5
	fmt.Println(v)

	p := &v
	(*p).X = 10
	p.Y = 9
	fmt.Println(v)
}

```
`type` 关键字就像 C 中的 `typedef` 一样

初始化:
创建时在结构体名后的花括号中传入字段值
```go
package main

import (
	"fmt"
)

type Vertex struct {
	X, Y int
}

var (
	v1   = Vertex{1, 2}
	v2   = Vertex{X: 1} // Y被默认置为零值
	v3   = Vertex{}
	ptr1 = &Vertex{1, 2}
	ptr2 = new(Vertex) // 使用new关键字不能传参, 只会创造默认0值的对象
)

func main() {
	fmt.Println(v1, v2, v3, ptr1, *ptr2)
	// {1 2} {1 0} {0 0} &{1 2} {0 0}
}

```

方法:
方法就是带有特殊接收者(对象)的函数, 这个接收者在方法名的前面
语法: func (接收者) 方法名 (方法参数) 返回值 {...}
```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
}

```
对象调用方法也直接使用 `.` 运算符

指针接收者:
传指针, 可以修改对象的数据成员的值
```go
package main

import (
	"fmt"
)

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) {
	v.X *= f
	v.Y *= f
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(v)
	v.Scale(10)
	fmt.Println(v)
}

```


接口:
接口是对结构体的一种抽象, 接口中通常包含了许多函数,
实现了这些函数的结构就可被认为实现了这个接口,
并可以通过接口调用结构的方法

语法:
type 接口名 interface {
	method1()
	...
}
```go
type shape interface {
  area() float64
  perimeter() float64
}

type rect struct {
    width, height float64
}
func (r rect) area() float64 {
    return r.width * r.height
}
func (r rect) perimeter() float64 {
    return 2*r.width + 2*r.height
}

type circle struct {
    radius float64
}
func (c circle) area() float64 {
    return math.Pi * c.radius * c.radius
}
func (c circle) perimeter() float64 {
    return 2 * math.Pi * c.radius
}

```
在这个例子中, rect 和 circle 这两个结构体就实现了 shape 这个接口
