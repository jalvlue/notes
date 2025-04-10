- 这里的场景是编译器后端并发编译函数的过程
- 需要编译的函数放在 `compilequeue`
- 并发模式是进行并发控制, 限制并发量
- 三种携程
	- 主携程: 负责将 `compilequeue` 中的函数包装成 `work` 丢进 `workq` 中, 然后使用 `wg.Wait()` 等待任务完成
	- 协调携程: 长时运行, 从 `workq` 中拿到需要执行的任务, 从 `done` 中拿到空闲的工作携程 id, 如果有空闲的工作携程 id 以及待执行任务, 就启动一个工作携程执行该任务
	- 工作携程: 同步执行 `work`, 完成后将自己的 id 放入 `done` 中, 逻辑上将这个携程释放, 然后退出
- 两个通道
	- `workq`: 用于从主携程中接受封装好的任务
	- `done`: 用于工作携程完成任务之后将该逻辑工作携程释放, 返回给协调携程, 以供下一次调度
- 两个队列
	- `pending`: 等待完成的任务队列
	- `ids`: 空闲的逻辑携程 id
	- 协调携程会频繁从这两个队列中截断出任务和对应的工作携程
- 这里有携程泄露的风险, 协调携程没有退出逻辑, 可以在主携程等待所有任务完成后将 `workq` 关闭, 然后在协调携程中进行判断
- 
- 除了并发模式外, 高阶函数的写法也值得学习

```go
// compileFunctions compiles all functions in compilequeue.
// It fans out nBackendWorkers to do the work
// and waits for them to complete.
func compileFunctions() {
	if race.Enabled {
		// Randomize compilation order to try to shake out races.
		tmp := make([]*ir.Func, len(compilequeue))
		perm := rand.Perm(len(compilequeue))
		for i, v := range perm {
			tmp[v] = compilequeue[i]
		}
		copy(compilequeue, tmp)
	} else {
		// Compile the longest functions first,
		// since they're most likely to be the slowest.
		// This helps avoid stragglers.
		sort.Slice(compilequeue, func(i, j int) bool {
			return len(compilequeue[i].Body) > len(compilequeue[j].Body)
		})
	}

	// By default, we perform work right away on the current goroutine
	// as the solo worker.
	queue := func(work func(int)) {
		work(0)
	}

	if nWorkers := base.Flag.LowerC; nWorkers > 1 {
		// For concurrent builds, we allow the work queue
		// to grow arbitrarily large, but only nWorkers work items
		// can be running concurrently.
		workq := make(chan func(int))
		done := make(chan int)
		go func() {
			ids := make([]int, nWorkers)
			for i := range ids {
				ids[i] = i
			}
			var pending []func(int)
			for {
				select {
				case work := <-workq:
					pending = append(pending, work)
				case id := <-done:
					ids = append(ids, id)
				}
				for len(pending) > 0 && len(ids) > 0 {
					work := pending[len(pending)-1]
					id := ids[len(ids)-1]
					pending = pending[:len(pending)-1]
					ids = ids[:len(ids)-1]
					go func() {
						work(id)
						done <- id
					}()
				}
			}
		}()
		queue = func(work func(int)) {
			workq <- work
		}
	}

	var wg sync.WaitGroup
	var compile func([]*ir.Func)
	compile = func(fns []*ir.Func) {
		wg.Add(len(fns))
		for _, fn := range fns {
			fn := fn
			queue(func(worker int) {
				ssagen.Compile(fn, worker)
				compile(fn.Closures)
				wg.Done()
			})
		}
	}

	types.CalcSizeDisabled = true // not safe to calculate sizes concurrently
	base.Ctxt.InParallel = true

	compile(compilequeue)
	compilequeue = nil
	wg.Wait()

	base.Ctxt.InParallel = false
	types.CalcSizeDisabled = false
}
```
