raft 实际上是一个共识协议, 可以作用在 GFS, VM-FT 之类的实际系统中

reft 处理的问题是:
split brain: 一个系统中出现了多个 master/primary(可能是 network partition 导致的), 这种情况对系统的一致性有很大的伤害(client 不知道要寻找哪个 primary, 哪个 primary 是正确的)

raft 处理 split brain 的策略: 
1. odd number of servers: for majority(all servers include failed servers) vote
2. 2\*F + 1 servers ->> tolerate F servers failed
3. Majority overlap: 所有当前 term 的 majority servers 中至少有一个是上一 term 的 majority(majority 的特性) -> 当前 term 的 leader 通过 overlap 了解到上一 term leader 的 term number, 同时知道上一 term leader committed 了哪些 log

# raft library
raft 实际上是提供给应用的一个 library, 帮助需要 replication 的应用达成 consensus

例如: 一个 distributed K/V storage server
应用结构图:
![[Pasted image 20231122162836.png]]

假设一个 client 发送了一个 PUT(k, v)的请求,
kv server 收到请求之后, 就会把请求传到下层的 raft 中, 等待 raft 的返回,
raft 将这个请求的 log(command)复制到 majority 之后, 将结果返回给 kv server
kv server 收到 raft 的结果, 将结果返回给 client

时序图:
![[Pasted image 20231219135123.png]]

raft 作用在应用之下, 帮助应用维护 replication consensus

log 的作用: 为什么使用 log
对于一个 state machine, client request 可能并发到来, 因此不仅要执行所有并发 request, 还要按照顺序来, 因此使用只可以 append 的 log 就可以维持 request 的次序, 并顺序执行,
此外, log 可以视为一个暂存的区, 对于有些掉线一阵子的 follower, leader 可以从 log 中将 request 发给 follower


interface: raft 和 kv server 进行交互的渠道
![[Pasted image 20231219140543.png]]
kv server 收到 client request

kv server 通过调用 raft 提供的 Start(command)接口让 raft 开始同步某个 log(command), 然后 Start 立即返回

raft 将结果放到 applyCh 中(如果这个 log command 成功 commit), 

kv server 从 applyCh 中读出同步结果, 然后执行结果(apply), 然后把执行结果返回给 client


# raft details

### leader election
....


election restriction:
一个 server 只有在 candidate 有更新的 log 时, 才会把票投给他
这样做的实质是将大多数(majority servers)都拥有的 log 保存了下来, 
只有一些少数 server 拥有的 log 就可以在新的 leader term 中被删除

![[Pasted image 20231219145708.png]]
例如图中:
log[11]可能是未 committed 的, 但由于大多数 server 都拥有,
因此在 election 之后, 一定会被保留下来

why not longest log as leader?
![[Pasted image 20231219154716.png]]
在这个例子中, term_8 的 log 可能已经 committed 了, 因此如果直接选择拥有最长 log 的 server[S1],
将会导致[S1]在同步 log 时覆盖掉 term_8 log,
对于已经 committed 的 log 是不可接受的

解决方法:
只有在 lastLogTerm 比 candidate 的小
或相同, 但是 candidate 在这个 term 有不少于自己的 log 数量才投票给 candidate
```go
if (rf.lastLogTerm < candidate.lastLogTerm) ||
	(rf.lastLogTerm == candidate.lastLogTerm && rf.lastLogIndex <= candidate.lastLogIndex) {
	votedGranted = true	
}
```

### append entries
AE 就是 leader 向 follower 发送 heartbeat 并且同步 log 的过程


append entries and syncing log
最简单的想法中, 使用了一次回滚一个 log 的方式进行同步匹配,
因此如果有 server 落后太多, 就会造成很大的同步负担(要发很多 rpc, 并且很花时间)

fast backup strategy
因此可以在 AE reply 中加上一些 follower logs 的信息, 比如 lastLogIndex, lastLogTerm 之类的, 然后 leader 可以跳过一些 entries, 从自己的 log 中找到对应 term 的起始 log, 然后用 AE 发给 follower 
一个实现:
![[Pasted image 20240308155824.png]]



### persistence
- currentTerm
- voteFor
- logs
这三个属性是需要做持久化的, 因此每次发生修改都需要持久化, 以便在崩溃恢复时重新掌握这些状态, 快速恢复运行状态


### log compaction and snapshot
长期运行后, 服务器会积累很多 log, 占据太多磁盘和内存空间, 同时对崩溃恢复也会带来很多负担
因此做 `log compaction, snapshot` 是很有必要的

![[Pasted image 20240308165510.png]]
![[Pasted image 20240308165451.png]]

服务器的 log 和他的状态(snapshot)在某种程度上是等价的
在某个 index 做 snapshot, 然后将这个 snapshot 持久化存储,
之后服务器就再也不需要这些 logs 了


# linearizability
线性一致性--强一致性
这是希望 raft 服务器集群表现出来的性质
