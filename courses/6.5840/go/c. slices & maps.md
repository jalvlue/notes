# slices
数组:
Go 中的数组是固定长度的, 因此数组的类型与长度有关
```go
var arr [3]string
primes := [3]int{2, 3, 5, 7, 11, 13}
```

切片:
类似 python, 就是在底层数组上引用数组连续的一段, 作为切片使用
```go
primesSlice := primes[1:4]  // {3, 5, 7}
```
arrayname[lowIndex:highIndex]
arrayname[lowIndex:]
arrayname[:highIndex]
arrayname[:]

除此之外, 还可以在不先创建底层数组的情况下, 使用 `make` 关键字直接创建初始化为零值的切片
```go
// func make([]T, len, cap)
mySlice := make([]int, 5, 10)
// cap 是可选的, 不指定则使用默认大小
mySlice := make([]int, 5)
```

还可以在声明时初始化切片
```go
mySlice := []string{"I", "love", "go"}
fmt.Println(cap(mySlice)) // 3
```

备注: `make()` 是内建函数, 类似地, `len()`, `cap()` 这两个内建函数对切片/数组使用

`...` 运算符:
```go
func concat(strs ...string) string {
    final := ""
    // strs is just a slice of strings
    for _, str := range strs {
        final += str
    }
    return final
}

func main() {
    final := concat("Hello ", "there ", "friend!")
    fmt.Println(final)
    // Output: Hello there friend!
}

```
使用 `...` 运算符传递函数参数时, 
函数调用者传递的是离散的, 用 `,` 分隔开的一个个的参数, (例如上面的 concat("Hello ", "there ", "friend!"))
但是在函数里, `...` 运算符会将这些一个个的参数收集起来, 放在一个切片中, (例如上面的 func concate(strs ...))
因此在函数中, 这些参数就只是切片 `strs` 中的一个个元素

```go
func printStrings(strings ...string) {
	for i := 0; i < len(strings); i++ {
		fmt.Println(strings[i])
	}
}

func main() {
    names := []string{"bob", "sue", "alice"}
    printStrings(names...)
}

```
此外, 对切片使用 `...` 运算符, 会将这个切片分解成一个个元素
对于一些接收 `...` 运算符传递参数的函数, 这样的转化是必须的


append:
append 做的是给传递进去的切片添加传递进去的元素, 并且返回添加后的切片
就是一个典型的 `...` 运算符传递参数函数
```go
func append(slice []Type, elems ...Type) []Type

slice = append(slice, oneThing)
slice = append(slice, firstThing, secondThing)
slice = append(slice, anotherSlice...)
```
可以直接在 append 中传递多个需要添加进去的单个元素,
也可以直接传一个切片使用 `...` 分解

注意: append 函数在切片的底层数组容量不足以完成车次 append 时, 会自动换一个更大的底层数组, 因此对于 append 的使用
必须要 sameslice = append(sameslice, ...)
对于同一个切片使用

多维切片:
就类似其他语言的多维数组
```go
rows := [][]int{}

func createMatrix(rows, cols int) [][]int {
	matrix := make([][]int, 0)
	for i:=0; i<rows; i++{
		row := make([]int, 0)
		for j:=0; j<cols; j++{
			row = append(row, i*j)
		}
		matrix = append(matrix, row)
	}
	return matrix
}
```

range:
Go 提供了一个语法糖, 用于更简单地遍历一个切片
```go
// 完整语法
for INDEX, ELEMENT := range SLICE {
	...
}

// 不需要element
for INDEX := range SLICE {
	...
}

// 不需要index
for _, ELEMENT := range SLICE {
	...
}
```


# maps
在 Go 中有类似哈希表, 字典的内建类型: map
初始化方式:
```go
m1 := make(map[int]string)

m2 := map[int]string {
	0 : "zero",
	1 : "one",
}
```

需要注意的是:
使用 var 关键字声明, 但没有使用 `make` 或没有在声明时使用列表初始化的映射成为 nil 映射, 没有键, 也不能插入键(可以使用 `make` 给这个映射插入的能力)

映射操作:  
插入或修改:  
m[key] = value

获得元素:  
elem = m[key]

删除某个键:  
delete(m, key)  
delete 也是内建函数

检测键是否存在:  
elem, 🆗 = m[key]  
key 存在时, 🆗 为 true, elem 为 value
不存在时, 🆗 为 false, elem 为该类型的零值


maps key type:
在一个映射中, key 必须是可比较的(comparable, 可比较的 key 才能用于做哈希),
因此, 一个 key 不能是函数, 切片和映射

此外, 对于嵌套映射(有多个 key), 更好的方法是将多个 key 放在一个结构体中组织(嵌套映射很容易造成访问 nil 映射造成 panic )


# 总结
切片和映射都是引用类型的, 也就是说在函数参数中传递时, 是传引用