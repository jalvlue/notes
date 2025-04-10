Golang 中内置了许多数据结构, 这里看几种比较常见的数据结构的实现
- 数组
- 切片
- 哈希表
- 字符串


# 3.1 数组
```go
// Array contains Type fields specific to array types.
type Array struct {
	Elem  *Type // element type
	Bound int64 // number of elements; <0 if unknown yet
}
```

- Golang 中数组需要从两种维度进行描述
	- 存储的元素类型
	- 最大能存储的元素个数 (数组长度)
- 数组长度确定之后无法改变, 两个数组只有元素类型相同且长度相同才是同一类型
- 数组访问越界是很严重的问题
	- 常量作为下标访问如果出现越界会在编译期间发现
	- 使用变量访问数组越界时, go runtime 会阻止


# 3.1 切片
```go
// Slice contains Type fields specific to slice types.
type Slice struct {
	Elem *Type // element type
}

// NewSlice returns the slice Type with element type elem.
func NewSlice(elem *Type) *Type {
	if t := elem.cache.slice; t != nil {
		if t.Elem() != elem {
			base.Fatalf("elem mismatch")
		}
		if elem.HasShape() != t.HasShape() {
			base.Fatalf("Incorrect HasShape flag for cached slice type")
		}
		return t
	}

	t := newType(TSLICE)
	t.extra = Slice{Elem: elem}
	elem.cache.slice = t
	if elem.HasShape() {
		t.SetHasShape(true)
	}
	return t
}

// runtime struct of a slice
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```

![[Pasted image 20240709165957.png]]
- 切片实际上就是在数组上进行了封装, 同时引入了动态扩容机制

