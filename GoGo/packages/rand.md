在 Go 语言的标准库中，`rand` 包提供了生成随机数的功能。它可以用于生成伪随机数、随机洗牌、随机选择等操作。下面是一些 `rand` 包中常用的函数和用法：

1. `Intn(n int) int`：生成一个介于 0 和 n-1 之间的伪随机整数。例如，`rand.Intn(100)` 会生成一个 0 到 99 之间的随机整数。

2. `Float64() float64`：生成一个介于 0.0 和 1.0 之间的伪随机浮点数。例如，`rand.Float64()` 会生成一个 0.0 到 1.0 之间的随机浮点数。

3. `Perm(n int) []int`：生成一个长度为 n 的随机排列的整数切片。这在洗牌和随机选择中特别有用。例如，`rand.Perm(10)` 会生成一个 0 到 9 的随机排列。

4. `Shuffle(n int, swap func(i, j int))`：将长度为 n 的切片随机洗牌。`swap` 函数用于自定义交换元素的行为。例如，可以使用 `rand.Shuffle(len(slice), func(i, j int) { slice[i], slice[j] = slice[j], slice[i] })` 来随机打乱切片中的元素顺序。

5. `Seed(seed int64)`：用指定的种子初始化随机数生成器。如果不手动设置种子，`rand` 包会使用系统时间作为默认种子。

需要注意的是，`rand` 包生成的随机数是伪随机数，其实际上是通过一个确定性算法生成的。为了在每次运行程序时都得到不同的随机序列，可以使用不同的种子进行初始化。在通常情况下，不需要手动调用 `Seed` 函数，因为默认使用系统时间作为种子。

以下是一个简单的示例，展示了 `rand` 包的使用：

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	// 使用当前时间作为种子
	rand.Seed(time.Now().UnixNano())

	// 生成一个随机整数
	randomInt := rand.Intn(100)
	fmt.Println("Random Integer:", randomInt)

	// 生成一个随机浮点数
	randomFloat := rand.Float64()
	fmt.Println("Random Float:", randomFloat)

	// 生成一个随机排列
	randomPerm := rand.Perm(10)
	fmt.Println("Random Permutation:", randomPerm)

	// 随机洗牌切片
	slice := []int{1, 2, 3, 4, 5}
	rand.Shuffle(len(slice), func(i, j int) {
		slice[i], slice[j] = slice[j], slice[i]
	})
	fmt.Println("Shuffled Slice:", slice)
}
```

在上述示例中，我们首先使用 `time.Now().UnixNano()` 作为种子来初始化随机数生成器。然后，通过 `rand.Intn` 生成一个随机整数，使用 `rand.Float64` 生成一个随机浮点数，使用 `rand.Perm` 生成一个随机排列，最后通过 `rand.Shuffle` 随机洗牌切片。运行程序时，会输出生成的不同的随机数和随机排列的结果。

`rand` 包提供了基本的随机数生成功能，可以满足大部分的随机数需求。如果需要更高质量的随机数，可以考虑使用加密安全的随机数生成器，如 `crypto/rand` 包。