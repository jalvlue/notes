- 并发模式
	- `pipeline`
	- `cancellation`

# Introduction
- 如何通过 `pipeline` 更高效地利用 IO 和多核 CPU
- 如何取消或者处理错误


# What is a pipeline
- 流水线就是将整个流程分成几个阶段, 每一个阶段由一个函数逻辑组成, 多个 `goroutine` 并发执行同一个阶段的函数逻辑
- 从 `upstream channels` 中接受上一个阶段的数据
- 在本阶段的函数逻辑中并发处理数据
- 将数据发送到 `downstream channels`


# Squaring numbers
- 这个简单的例子构造了一个三阶段的 `pipeline`

### Stage 1
- `gen`
```go
func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)
    }()
    return out
}
```

### Stage 2
- `sq`
```go
func sq(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}
```

### Stage 3
- `main`
```go
func main() {
    // Set up the pipeline.
    c := gen(2, 3)
    out := sq(c)

    // Consume the output.
    fmt.Println(<-out) // 4
    fmt.Println(<-out) // 9
}
```

### Stage 4
- 由于上面处理流程拆成了一个个的阶段, 因此可以随意组合
- 一个 4 阶段的例子
```go
func main() {
    // Set up the pipeline and consume the output.
    for n := range sq(sq(gen(2, 3))) {
        fmt.Println(n) // 16 then 81
    }
}
```


# Fan-out, fan-in
- `fan-out`: 多个 `goroutine` 可以并发安全地从一个 `channel` 中读取数据, 因此很适合来通过一个 `channel` 将任务发布到多个 `worker goroutines` 中
- `fan-in`: 一个函数可以将多个输入 `channel` 的内容读取然后处理整合到一个输出 `channel` 中, 很适合对多个 `channel` 的数据进行中间处理
- 这个例子就使用了 `fan-in` 的模式, 只是没有做任何中间处理, 直接将来自多个 `input channel` 的数据整合后放到 `output channel` 中
```go
func main() {
    in := gen(2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(in)
    c2 := sq(in)

    // Consume the merged output from c1 and c2.
    for n := range merge(c1, c2) {
        fmt.Println(n) // 4 then 9, or 9 then 4
    }
}

func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c is closed, then calls wg.Done.
    output := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    // Start a goroutine to close out once all the output goroutines are
    // done.  This must start after the wg.Add call.
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```


# Stopping short
- 数据的流通的时机都是基于 `upstream, downstream channels` 是否关闭
	- 当前阶段会不断读取 `upstream channel`, 直到它被关闭
	- 当前阶段会在写完 `downstream data` 之后, 关闭 `downstream channel`
- 有时候下流可能不需要上流的所有数据, 因此需要快速退出


# Explicit cancellation
- 这个例子中, `out` 这个通道有两个值, 但是如果只读了一个值的话, 将会有一个上游携程等待我们的读取, 如果没处理就会造成携程泄露
- 这里利用了关闭携程后, 读取会立即返回的性质, 将信息广播给所有上有等待读取我们读取的携程, 更加通用, 如果一个个往 `done` 通道中发信息通知上游携程的话就太麻烦了
```go
func sq(done <-chan struct{}, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-done:
            	// 下游不再需要数据
                return
            }
        }
    }()
    return out
}

func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c or done is closed, then calls
    // wg.Done.
    output := func(c <-chan int) {
        defer wg.Done()
        for n := range c {
            select {
            case out <- n:
            case <-done:
            	// 下游不再需要数据
                return
            }
        }
    }
    // ... the rest is unchanged ...
}

func main() {
    // Set up a done channel that's shared by the whole pipeline,
    // and close that channel when this pipeline exits, as a signal
    // for all the goroutines we started to exit.
    done := make(chan struct{})

	// 神来之笔, 通知上游不再需要数据
    defer close(done)          

    in := gen(done, 2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(done, in)
    c2 := sq(done, in)

    // Consume the first value from output.
    out := merge(done, c1, c2)
    fmt.Println(<-out) // 4 or 9

    // done will be closed by the deferred call.      
}
```


# Digesting a tree
- 模仿 `md5sum` 这个程序, 计算文件的校验和

### 单携程版本
```go
//go:build OMIT
// +build OMIT

package main

import (
	"crypto/md5"
	"fmt"
	"io/ioutil"
	"os"
	"path/filepath"
	"sort"
)

func main() {
	// Calculate the MD5 sum of all files under the specified directory,
	// then print the results sorted by path name.
	m, err := MD5All(os.Args[1]) // HL
	if err != nil {
		fmt.Println(err)
		return
	}
	var paths []string
	for path := range m {
		paths = append(paths, path)
	}
	sort.Strings(paths) // HL
	for _, path := range paths {
		fmt.Printf("%x  %s\n", m[path], path)
	}
}

// MD5All reads all the files in the file tree rooted at root and returns a map
// from file path to the MD5 sum of the file's contents.  If the directory walk
// fails or any read operation fails, MD5All returns an error.
func MD5All(root string) (map[string][md5.Size]byte, error) {
	m := make(map[string][md5.Size]byte)
	err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error { // HL
		if err != nil {
			return err
		}
		if !info.Mode().IsRegular() {
			return nil
		}
		data, err := ioutil.ReadFile(path) // HL
		if err != nil {
			return err
		}
		m[path] = md5.Sum(data) // HL
		return nil
	})
	if err != nil {
		return nil, err
	}
	return m, nil
}
```

### 并发版本
- 这里将上面单协程的 `md5sum` 改成了并发版本, 对于每一个文件都会有一个单独的协程进行处理
- 通过两个 `channel` 进行数据传递
- 有一个非常巧妙的做法是: 
	- 在当前阶段创建通道, 然后另起一个协程往通道中生产, 同时立即返回, 将通道交给下流
	- 此时下流就可以在当前阶段生产的同时读取通道消费数据
	- 同时, 如果下流不再需要数据了, 直接关闭 `done` 通道, 广播通知上流的生产者协程
	- 例子中的上流 `func sumFiles(done <-chan struct{}, root string) (<-chan result, <-chan error)` 函数就是这样做的, 创建通道, 另起协程写通道, 立即返回将通道交给下流
	- 例子中的下流 `func walkFiles(done <-chan struct{}, root string) (<-chan string, <-chan error)` 在遇到不再需要数据 (遇到错误了, 提前退出返回) 时, 调用 `close(done)`, 广播通知上流

```go
package main

import (
	"crypto/md5"
	"errors"
	"fmt"
	"io/ioutil"
	"os"
	"path/filepath"
	"sort"
	"sync"
)

func main() {
	// Calculate the MD5 sum of all files under the specified directory,
	// then print the results sorted by path name.
	m, err := MD5All(os.Args[1])
	if err != nil {
		fmt.Println(err)
		return
	}
	var paths []string
	for path := range m {
		paths = append(paths, path)
	}
	sort.Strings(paths)
	for _, path := range paths {
		fmt.Printf("%x  %s\n", m[path], path)
	}
}

// A result is the product of reading and summing a file using MD5.
type result struct {
	path string
	sum  [md5.Size]byte
	err  error
}

// sumFiles starts goroutines to walk the directory tree at root and digest each
// regular file.  These goroutines send the results of the digests on the result
// channel and send the result of the walk on the error channel.  If done is
// closed, sumFiles abandons its work.
func sumFiles(done <-chan struct{}, root string) (<-chan result, <-chan error) {
	// For each regular file, start a goroutine that sums the file and sends
	// the result on c.  Send the result of the walk on errc.
	c := make(chan result)
	errc := make(chan error, 1)
	go func() { // HL
		var wg sync.WaitGroup
		err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
			if err != nil {
				return err
			}
			if !info.Mode().IsRegular() {
				return nil
			}
			wg.Add(1)
			go func() { // HL
				data, err := ioutil.ReadFile(path)
				select {
				case c <- result{path, md5.Sum(data), err}: // HL
				case <-done: // HL
				}
				wg.Done()
			}()
			// Abort the walk if done is closed.
			select {
			case <-done: // HL
				return errors.New("walk canceled")
			default:
				return nil
			}
		})
		// Walk has returned, so all calls to wg.Add are done.  Start a
		// goroutine to close c once all the sends are done.
		go func() { // HL
			wg.Wait()
			close(c) // HL
		}()
		// No select needed here, since errc is buffered.
		errc <- err // HL
	}()
	return c, errc
}

// MD5All reads all the files in the file tree rooted at root and returns a map
// from file path to the MD5 sum of the file's contents.  If the directory walk
// fails or any read operation fails, MD5All returns an error.  In that case,
// MD5All does not wait for inflight read operations to complete.
func MD5All(root string) (map[string][md5.Size]byte, error) {
	// MD5All closes the done channel when it returns; it may do so before
	// receiving all the values from c and errc.
	done := make(chan struct{}) // HLdone
	defer close(done)           // HLdone

	c, errc := sumFiles(done, root) // HLdone

	m := make(map[string][md5.Size]byte)
	for r := range c { // HLrange
		if r.err != nil {
			return nil, r.err
		}
		m[r.path] = r.sum
	}
	if err := <-errc; err != nil {
		return nil, err
	}
	return m, nil
}
```

# Bounded parallelism
- 有限并行, 在上面的例子中, 在 `filepath.Walk()` 的过程中, 对每一个遇到的文件都新开一个协程去处理, 如果文件过多, `filepath.Walk()` 过快, 可能导致太多创建太多 `goroutine`
- 这里其实也就是加上了并发控制
- 上流 `func walkFiles(done <-chan struct{}, root string) (<-chan string, <-chan error)` 创建通道 `errc, paths`, 另起协程执行 `filepath.Walk()`, 将搜集到的 `path` 放进 `paths` 中供下流消费, 然后快速返回, 将 `paths, errc` 交给下流
- 下流采用的并发控制方法就是单纯的开启固定数量的 `goroutine` 轮询上流的 `path` 通道

```go
package main

import (
	"crypto/md5"
	"errors"
	"fmt"
	"io/ioutil"
	"os"
	"path/filepath"
	"sort"
	"sync"
)

func main() {
	// Calculate the MD5 sum of all files under the specified directory,
	// then print the results sorted by path name.
	m, err := MD5All(os.Args[1])
	if err != nil {
		fmt.Println(err)
		return
	}
	var paths []string
	for path := range m {
		paths = append(paths, path)
	}
	sort.Strings(paths)
	for _, path := range paths {
		fmt.Printf("%x  %s\n", m[path], path)
	}
}

// walkFiles starts a goroutine to walk the directory tree at root and send the
// path of each regular file on the string channel.  It sends the result of the
// walk on the error channel.  If done is closed, walkFiles abandons its work.
func walkFiles(done <-chan struct{}, root string) (<-chan string, <-chan error) {
	paths := make(chan string)
	errc := make(chan error, 1)

	go func() { // HL
		// Close the paths channel after Walk returns.
		defer close(paths) // HL
		// No select needed for this send, since errc is buffered.
		errc <- filepath.Walk(root, func(path string, info os.FileInfo, err error) error { // HL
			if err != nil {
				return err
			}
			if !info.Mode().IsRegular() {
				return nil
			}
			select {
			case paths <- path: // HL
			case <-done: // HL
				return errors.New("walk canceled")
			}
			return nil
		})
	}()

	return paths, errc
}

// A result is the product of reading and summing a file using MD5.
type result struct {
	path string
	sum  [md5.Size]byte
	err  error
}

// digester reads path names from paths and sends digests of the corresponding
// files on c until either paths or done is closed.
func digester(done <-chan struct{}, paths <-chan string, c chan<- result) {
	for path := range paths { // HLpaths
		data, err := ioutil.ReadFile(path)
		select {
		case c <- result{path, md5.Sum(data), err}:
		case <-done:
			return
		}
	}
}

// MD5All reads all the files in the file tree rooted at root and returns a map
// from file path to the MD5 sum of the file's contents.  If the directory walk
// fails or any read operation fails, MD5All returns an error.  In that case,
// MD5All does not wait for inflight read operations to complete.
func MD5All(root string) (map[string][md5.Size]byte, error) {
	// MD5All closes the done channel when it returns; it may do so before
	// receiving all the values from c and errc.
	done := make(chan struct{})
	defer close(done)

	paths, errc := walkFiles(done, root)

	// Start a fixed number of goroutines to read and digest files.
	c := make(chan result) // HLc
	var wg sync.WaitGroup
	const numDigesters = 20
	wg.Add(numDigesters)
	for i := 0; i < numDigesters; i++ {
		go func() {
			digester(done, paths, c) // HLc
			wg.Done()
		}()
	}
	go func() {
		wg.Wait()
		close(c) // HLc
	}()
	// End of pipeline. OMIT

	m := make(map[string][md5.Size]byte)
	for r := range c {
		if r.err != nil {
			return nil, r.err
		}
		m[r.path] = r.sum
	}
	// Check whether the Walk failed.
	if err := <-errc; err != nil { // HLerrc
		return nil, err
	}
	return m, nil
}

```
