[sync](https://pkg.go.dev/sync)

# Cond
- 条件变量
- 直接用 `channel` 模拟就好了, 没必要使用这个


# Map
- 并发安全的 `map[interface]interface`

`sync.Map` 是 Go 语言标准库中提供的一种并发安全的键值对集合类型。它可以在多个 Goroutine 之间安全地进行读取和写入操作，而无需额外的锁机制。
`sync.Map` 的设计目标是针对读多写少的场景，其中并发读取是高效的，而并发写入则相对较慢。它的实现采用了一种特殊的数据结构和算法，以实现高效的并发访问，同时避免了使用全局锁或细粒度锁。

下面是 `sync.Map` 的一些关键原理：
1. 分段锁：`sync.Map` 内部使用了一种分段锁的机制。它将整个键值对集合划分为多个段（segments），每个段由一个互斥锁（Mutex）保护。这样，多个 Goroutine 可以同时对不同的段进行读取或写入操作，从而实现并发性。
2. 无锁读取：在读取操作时，不会涉及锁的竞争。每个段维护一个只读的映射表（read-only map），Goroutine 可以直接从映射表中读取数据。这样可以避免锁的开销，提高并发读取的效率。
3. 锁粒度控制：在写入操作时，只需要获取对应段的锁，而不会涉及整个 `sync.Map` 的锁。这样可以降低锁的竞争范围，提高并发写入的效率。
4. 背景清理：`sync.Map` 内部会定期检测和清理不再使用的段，以释放资源。这样可以避免无限制地增长内部数据结构，减少内存占用。

需要注意的是，`sync.Map` 的设计目标是提供一种简单且高效的并发安全集合，适用于特定的使用场景。它适用于读多写少的情况，但对于大规模并发的高写入负载，可能不如其他数据结构（如基于锁的 `sync.Mutex` 或读写锁的 `sync.RWMutex`）效果好。

在使用 `sync.Map` 时，需要注意以下几点：
- `sync.Map` 的键和值类型必须是可比较的（comparable）。
- 由于 `sync.Map` 的设计目标是简化使用并提供高效的并发性，因此它没有提供像普通的 map 类型那样的迭代器功能，也没有提供检查集合大小的方法。
- `sync.Map` 的读写操作是并发安全的，但其单个操作并不保证原子性。

总而言之，`sync.Map` 提供了一种简单且高效的并发安全集合类型，适用于读多写少的场景。通过采用分段锁和无锁读取的机制，它能够在多个 Goroutine 之间实现高效的并发访问。
希望这个介绍能够帮助你理解 `sync.Map` 的原理和使用。如果你还有其他问题，请随时提问！


# Locker
```go
type Locker interface {
	Lock()
	Unlock()
}
```


# Mutex/RWMutex
- 就是标准库实现的一个高效的 `Locker`
- `m.Lock()`
- `m.TryLock()`
- `m.Unlock()`


# Once
- `func (o *Once) Do(f func())`, 提供了让某个函数只执行一次的承诺

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var once sync.Once
	onceBody := func() {
		fmt.Println("Only once")
	}
	done := make(chan bool)
	for i := 0; i < 10; i++ {
		go func() {
			once.Do(onceBody)
			done <- true
		}()
	}
	for i := 0; i < 10; i++ {
		<-done
	}
}
```

# Pool
`sync.Pool` 是 Go 语言标准库中提供的一个对象池（Object Pool）实现，用于管理和复用临时对象，以提高性能和减少内存分配的开销。
`sync.Pool` 的设计目标是针对短期临时对象的分配和回收，适用于频繁创建和销毁的场景。它通过维护一个对象池，将不再使用的对象保存起来，以便后续的重复使用，而不是每次都重新创建新的对象。

下面是 `sync.Pool` 的一些关键原理和行为：
1. 对象复用：`sync.Pool` 内部维护了一个对象队列，用于存储可复用的对象。当需要获取一个对象时，首先从对象队列中尝试获取。如果队列中有可用的对象，则直接返回；如果队列为空，则创建一个新的对象返回。
2. 对象生命周期：`sync.Pool` 并不保证对象的生命周期。对象可以在任何时候被池化（放入对象池）或释放（从对象池中移除）。当一个对象被放入对象池时，`sync.Pool` 并不对其进行引用计数或保留拷贝，因此对象可能在任何时候被垃圾回收。
3. 池的大小和回收策略：`sync.Pool` 的大小并不是固定的，它会根据需要动态地扩展和收缩。当获取对象时，如果对象队列为空，则会创建新的对象。当放入对象池时，如果对象队列已满，则对象会被丢弃，而不是放入队列。这样可以避免过多的对象积累和内存占用。
4. 并发安全：`sync.Pool` 是并发安全的，可以在多个 Goroutine 之间并发使用。对于每个 Goroutine，`sync.Pool` 会维护一个本地的对象池副本，以减少竞争和锁的开销。这样可以提高并发性能。

需要注意的是，`sync.Pool` 并不适用于所有场景。它适用于对临时对象进行频繁的创建和销毁的情况，例如对象的初始化开销较大，但在后续使用过程中，对象的状态不会发生变化。在这种情况下，通过对象池可以避免频繁的内存分配和垃圾回收，提高性能。

以下是 `sync.Pool` 的基本用法：
1. 创建对象池：使用 `sync.Pool` 的零值创建一个对象池。
   ```go
   var pool sync.Pool
   ```

2. 添加和获取对象：使用 `Put` 方法将对象放入对象池，使用 `Get` 方法从对象池获取对象。
   ```go
   obj := pool.Get()       // 从对象池获取对象
   // 对象使用和处理
   pool.Put(obj)           // 将对象放回对象池
   ```

通过合理利用 `sync.Pool` 可以在一定程度上提高程序的性能，特别是在频繁创建和销毁临时对象的场景下。然而，使用 `sync.Pool` 时需要注意对象的生命周期和池的大小，以避免潜在的问题。


# WaitGroup
- `wg.Wait()`
- `wg.Done()`
- `wg.Wait()`

```go
package main

import (
	"sync"
)

type httpPkg struct{}

func (httpPkg) Get(url string) {}

var http httpPkg

func main() {
	var wg sync.WaitGroup
	var urls = []string{
		"http://www.golang.org/",
		"http://www.google.com/",
		"http://www.example.com/",
	}
	for _, url := range urls {
		// Increment the WaitGroup counter.
		wg.Add(1)
		// Launch a goroutine to fetch the URL.
		go func(url string) {
			// Decrement the counter when the goroutine completes.
			defer wg.Done()
			// Fetch the URL.
			http.Get(url)
		}(url)
	}
	// Wait for all HTTP fetches to complete.
	wg.Wait()
}

```