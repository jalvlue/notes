MapReduce: Simplified Data Processing on Large Clusters

# overview/abstract
mapreduce 是个 `处理(processing)` 和 `生成(generating)` 大数据集的一个编程模型(programming model)和相关实现(associated implementation)

大概模型:
mapreduce 就像他的名字一样主要包括两个部分: `map` 和 `reduce`
map: 用户指定一个 map function, 使用这个 map function, 通过一个原始的键值对(a key/value pair)输入, 生成一组中间键值对(a set of intermediate key/value pairs)
reduce: 也是用户指定的一个 reduce function, 并通过它将所有具有相同中间键(intermediate key) 的中间值(intermediate values) 合并

现实世界中的很多任务都可以通过这个模型表示, 因此是很有效的

有了 mapreduce 这个框架, 完成并发任务只需要指定 map, reduce function 就能很好地使用分布式资源, 而无需关注或很懂分布式系统
并且使用 mapreduce 框架写出来的程序是自动并行的, 是在大集群普通商用计算机上运行的(a large cluster of commodity machines(广泛, 便宜, 可替换性强))
mapreduce(run-time system)会做好分割输入数据, 分配任务, 处理机器错误(machine failures), 管理机器之间必要的数据交换

mapreduce 还是高度可拓展的, Google 每天都在上千台机器的集群中用很多 mapreduce 程序处理 tb 级别的数据(文章背景下), 随时根据情况增减机器都是非常方便的


# 1. introduction
##### motivation and problem objective:
Google 需要处理数据(比如从已爬虫的文档中总结每个 host 爬虫数, 这些跟课程关系不大, 只需直到都是一些简单的, 很直接的任务)
但数据量大了之后, 为了保证完成时间, 就要让任务分布式完成

为此 Google 写了很多专用(special-purpose)计算程序来处理各种任务
这里平衡计算, 分配数据和处理故障让本来简单的代码变成大量复杂代码

mapreduce 的动机就是设计一个编程模型把所有乱七八糟的并行(parallelization), 错误处理(fault-tolerance), 数据分配(data distribution)和负载均衡(load-balancing)都抽象起来成为一个 `library`

因此目标就是让分布式任务的处理更加简单

重新执行(re-execution)是 mapreduce 的主要容错机制(primary mechanism for fault tolerance)

##### contribution
实现了简单又有用的接口(interface), 能够自动化并行, 分布式大规模计算
提供了实现, 在大集群普通计算机上性能很好


# 2. Programming Model
basic programming model

mapreduce 的计算就是用一大组 `input` 键值对生成一大组 `output` 键值对,
在 mapreduce 的加持下, 只需要两个函数就能完成计算: map 和 reduce 

map:
输入是一个 input file
输出是一组 intermediate K/V pairs

reduce:
输入是一个 intermediate key 以及一组对应 key 的 intermediate values
输出是一组尽可能小的 values(通常大小是 0 或 1)

map 和 reduce 之间的 intermediate K/Vs 由一个 `iterator` 实现

##### 2.1 example: word counting in a large collection of documents
```python
map(String key, String value):  
	// key: document name  
	// value: document contents  
	for each word w in value:  
		EmitIntermediate(w, "1");

reduce(String key, Iterator values):  
	// key: a word  
	// values: a list of counts  
	int result = 0;  
	for each v in values:  
		result += ParseInt(v);  
	Emit(AsString(result));
```
map 函数将文件中的所有 word 都生成(word, 1)的 intermediate K/V
mapreduce library 将所有同一个 key 的 K/V 的 value 都收集到 iterator 中, 传给 reduce 函数
reduce 再将所有值合并, 完成 word count


##### 2.2 types
![[Pasted image 20231029130442.png]]


##### 2.3 more examples
distributed Grep:
map 函数 emit grep 的结果,
reduce 函数是一个 identity function, 直接将结果输出就好

count of URL access frequency:
map 将 web page logs 作为输入, emit (URL, 1)
reduce 函数跟 word count 类似, 就是将所有 intermediate values 加起来, emit (URL, total access)

...


# 3. Implementation
implementation of the mapreduce tailored towards cluster-based computing environment

mapreduce 的具体实现依赖于具体的计算环境

##### 3.1 execution overview
1. mapreduce library 首先将输入文件切割成 M 块(每块通常是 16-64mb), 然后启动集群中计算机的程序副本
2. master 是一个特殊的副本, 负责管理分配, 其余的则是 worker. 在 M 个 map 任务, R 个 reduce 任务时, master 会从集群中选中 M+R 个空闲的计算机, 给他们分别分配任务
3. 分配到 map 任务的 worker, 将分配到的输入文件块放进 map function 中, 并将得到的 intermediate K/V pairs 缓存在内存中
4. 缓存定时写入到本地存储中, 并通过 mapreduce 的分区函数 (partitioning function)分成 R 个区间, 磁盘上的地址会返回给 master, 以便让 master 交给 reduce worker
5. reduce worker 收到 master 的提示后, 就会开启一个远程程序读取 map worker 磁盘中的 intermediate kv. 并对 key 进行排序, 让同一个 key 的 kv 都在一个集合中
6. reduce worker 接下来遍历 intermediate kv, 把一个 key, 以及它对应的一组 values 传递给 reduce function, 输出被保存在这个 reduce partition 的输出文件中
7. 所有 map 和 reduce workers 都完成任务后, master 唤醒用户程序, 然后返回到用户程序 (带着 R 个 reduce partition output files)

自此, 就完成了一个 mapreduce 计算
此时用户通常不会将 R 个文件合并, 
而是可以将这些 output files 放进一个新的 mapreduce 程序, 或其他的程序中做下一步的操作
![[Pasted image 20231029135819.png]]


##### 3.2 master data structures
master 需要保存很多信息从而实现全局掌控
对每一个 map/reduce task 保存任务的状态(idle, in-progress, completed...)
对集群中的机器保存运行状态(是不是 idle 的)
master 还作为 intermediate kvs 的导管, 保存 R 个 intermediate file regions location


##### 3.3 fault tolerance
在大集群上做任务, mapreduce library 必须容忍机器错误(tolerate machine failures gracefully)

Worker Failure:
master 通过定时访问(ping)worker 来查看 worker 的状态, 如果超过一定时间还没回应, 那么 master 将这个 worker 设成 failed, 将所有这个 worker 已完成的和未完成的 map tasks 设成 idle (因为 map 后 intermediate 存在 worker 本地, 无法取出了, 只能重新执行)
对于 reduce tasks, 已完成的任务最终输出存储在了 global file system, 因此不用重新执行, 未完成的需要设置为 idle, 重新执行

一个 map task failed, 所有的正在执行的 reduce tasks 都要重新执行

因此 mapreduce 是很有弹性的, 一个机器 failed, 就换机器再执行, 持续把任务推进下去, 直到完成任务


Master Failure:
对于 master, 可以定时设置 checkpoint,
如果 master failed, 新的 master 就可以冲 checkpoint 开始继续运行

但在这里的实现中, 只有一个机器运行 master, 因此几乎不可能 failed,
所以采取的策略是, 如果 master failed, 终止 mapreduce, 并给用户失败信息, 让用户决定是否重新运行


Semantics in the Presence of Failures:
在


##### 3.4 locality
带宽在文中的计算环境下是稀缺资源

由于输入数据(由 [[GFS]] 管理, GFS 将数据分块, 并把每一块的副本都存在几个计算机中)都是存储在集群中的计算机中的, 因此可以利用这个优势,
让 master 在 schedule 时把这个局部性信息也考虑上
尽量把 task 分配在有数据的机器上, 或距离进的机器上, 
利用 locality, 减少带宽使用


##### 3.5 task granularity
对于分配任务 M 和 R, 应该远大于机器数
让每个机器都执行多个任务, 有利于提高 dynamic load balancing, 机器 failed 时加速恢复(==不懂==)


##### 3.6 backup tasks
一个通常影响程序总执行时间的是掉队机器: 执行最后一部分 map 或 reduce 的机器执行得非常慢, 异常慢
机器变成掉队者的原因可能有很多: 机器内部问题, 多个任务竞争等...

一个有效的解决方法:
在所有任务快完成时, master 对所有还在进行的任务(可能成为掉队者),
发布一个 backup execution(让别的计算机也来执行)
backup 和原始的两个中哪个完成了, 也就把这个人物完成了

# 4. Refinements
several useful refinement

基础的功能已经很有效了, 但还有一些额外的有用的改善

##### partitioning function
在 intermediate kvs 分为 R 个 regions 时, 通常使用哈希函数
`hash(key) mod R` 
来保证键值对均匀分布

但有些任务, 比如 key 是 URL, 那么最好是在分区时就将同一个 URL 的分在一起, 为后续 sort 做准备
因此就可以使用 mapreduce library 中的
`hash(Hostname(urlkey)) mod R` 
让同一 url 键值对分布在同一部分中


##### ordering guarantees


##### combiner function


##### input and output types


##### side-effects


##### skipping bad records


##### local execution


##### status information


##### counters


# 5.
performance measurement


# 6.
explores the ues of mapreduce within Google


# 7. 
related and future work


# 8. Conclusions
