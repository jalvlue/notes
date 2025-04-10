Raft 是一种共识算法(consensus algorithm), 用于在分布式(存储)系统中各个 node 之间达成共识(表现出相同的状态, 从而实现一致性)

# 1. introduction
Raft 的特点是逻辑上简单易懂, 工程上容易构建

创新特点:
- Strong leader: log entry 只通过 leader 同步到各个 node
- Leader election: 使用了随机初始化的方式来构建 node 的 election timeout
- Membership change: 

# 2. replicated state machine
replicated state machine, 复制状态机, 可以简单理解为备份, 它与主机器(正在运行的服务器有相同的状态(相同的数据)),
在主机器 crashes 时, 可以马上候补上位继续提供相同的服务,
为系统提供了高可用性

共识算法通常就是用在同步多个 replicated state machine 上

replicated state machine 通常是使用 replicated log(复制日志)实现的
一个 log entry 中包含几个 command, 
state machines 顺序执行 command, 最后获得相同的状态(state)
![[Pasted image 20231122134837.png]]
这里, 共识算法的关键就是保证每一个 state machine 中的 log 都完全一致,
每个 state machine 的 consensus module 之间会进行通信, 确保大部分的 state machine log 都是正确的, 然后开始执行写入 log

# 3. what's wrong with Paxos
Paxos is difficult to understand, so the authors came up with Raft...

# 4. designing for understandability
1. Decomposing: leader election, log replication, safety and membership changes
2. Reducing the number of states

# 5. the Raft consensus algorithm
大概流程:
1. 选出 leader
2. Leader 从 client 接收 log
3. Leader 通知 followers 写入 log
4. Leader 通知 followers 什么时候执行 log

只有一个 leader, 并且 leader 拥有很大权力, 极大简化了流程, 因此每一个阶段(term)必须有一个且只能有一个 leader, leader failed 就进入下一个 term 并选出新 leader

Raft 将流程解构成了 3 个主要步骤:
1. Leader election
2. Log replication
3. Safety


总览:
![[Pasted image 20231122141815.png]]
![[Pasted image 20231122141835.png]]
![[Pasted image 20231122141953.png]]
![[Pasted image 20231122141937.png]]

##### 5.1 raft basics
每个 node 有三种状态:
1. Leader: 每个 term 只有一个
2. Candidate: follower 经过 electionTimeout 后开始一个新的选举周期并成为 candidate 
3. Follower: 每一个 server 最开始的状态

node state transfer:
![[Pasted image 20231122143127.png]]

term iteration:
![[Pasted image 20231122143252.png]]
每一个新的 term 都由新的选举(origin leader failed)产生

terms 就像是 raft 中的一个逻辑钟
旧 term 服从新 term
一旦发现自己处于旧 term (发生错误然后又恢复过来了)
就会立即向 leader 同步 log
额旧 term 的 candidate 或 leader 则会转变成 follower

Node 之间通过 RPC 通信
基本上有两种 RPC
1. RequestVote RPCs(RV): candidate 向 node 要投票
2. Append-Entries RPCs(AE): leader 向 followers 同步 log entry

```ad-note
当然 term 不是衡量一个 server 状态新旧的唯一标准, 
考虑这样的一种情况:
一个server与其他server之间发生了网络隔离, 在这种情况下无法接收到其他server的RV和AE, 同时也无法送达自己的RV和AE,
此时这个server就不断地达到electionTimeout, 开始新的选举周期
...
不断重复, 最后重新回到网络时可能已经处在一个很高的term了, 但是空有高term,
并没有任何来自用户的log, 
因此还需要使用log来进行状态同步
```

[[courses/6.5840/Raft|Raft]]
##### 5.2 leader election
leader election 发生在 follower 没有在 election timeout 之内收到来自 leader 的 heartbeat(leader failed 或者只是之间的网络故障)
此后 follower 就成了 candidate, 自增 term, 并发出 RV RPCs,

如果在 candidate 阶段收到了来自另外一个 leader 的 heartbeat, 它会检测 leader 的 term, 大于等于 candidate term, 则 candidate 立即转会 follower, 小于则 candidate 拒绝 heartbeat, 并返回信息

随机初始化 election timeout, 并在各个 timeout 之间保持一定的时间间隔(150-300ms), 让 `split vote`, 也就是多个 follower 同时成为 candidate 并获得了相同数量的选票发生的概率大大降低, 并保证了在发生后可以快速恢复并选出 leader 来


##### 5.3 log replication
leader 选出来后, 就代表所有 node, 向 client 提供服务, client 向 leader 发出请求(命令), leader 将命令作为一个新的 entry 写入它的 log 中, 
并向 followers 发出 AE, 让 followers 也往他们的 log 中写入这个 entry
NOTE: AE 和 heartbeatRPC 是同体的, 只有在 heartbeat 时, 才将 entries 同步

大多数 followers 都写入 log 后, 返回成功信息给 leader, leader 执行 log,
并要求 followers 也执行 log, 然后将新的 state machine 结果返回给 client, 

在这个过程中, 如果 follower 因为某些原因一直没有回复, 那么 leader 会不断地给他发 RPC, 即使大多数 followers 执行了 log, 并且已经将结果返回给 client 了, 还会继续尝试与没回复 follower 通信

log 结构:
![[Pasted image 20231124100414.png]]
log index 指示不同的 entry (命令), 
每个 entry 包括 term, 和 command

entry 可以被执行时(node 执行, 成为新的 state machine)
就认为这个 entry is committed
这是由 leader 决定的

leader 通过查看各个 follower 对某个 entry 的复制成功情况决定是否 commit, 只有大多数 follower 都复制了这个 entry(将这个 entry 写入 log ), leader 才能 commit 这个 entry, 例如上面的 entry7,

同时, entry commit 有点类似网计网中的累计确认, 只要这个 entry 被 commit 了, 那么它之前的 entry 也一定被 commit 了, follower 如果收到后面序号的 entry, 那么将会丢弃这个 entry, 等待它期待的 entry 到来

follower 知道哪个 entry 被 commit 后, 也就可以将这个 entry 执行, 进入新的 state

以上都是在系统刚启动, 第一个 leader 一切都好的情况下,
接下来考虑可能的, 各个 node 不一致的情况
![[Pasted image 20231124102236.png]]

在处理不一致 log 上, raft 采取的是强制让 follower 丢弃原先的 log, 完全转为与 leader 相同的 log,
这就要求选出来的 leader 拥有最新的 log (5.4 对 election 加入了限制来保证这个的实现)

做法分为三步:
1. 找出 leader 和 followers 出现分歧的 log entries
2. 删除 followers 这个 entry 及之后
3. 把 leader 这个 entry 及之后同步给 followers
(当然每个 follower 与 leader 不一致的点可能不同, 简略起见)

具体实现:
leader 会为每一个 follower 维护一个 nextIndex, 表示要发送给这个 follower 的下一个 entry index,
新 leader 会将这个 nextIndex 初始化为它的 log 中的下一个 entry,
例如 figure7 中 leader for term8 在初始化时就会把所有 nextIndex 设置为第 11 个 entry,

然后在 heartbeatRPC, 也就是 appendEntriesRPC 中进行 consistency-check, 检测 nextIndex 之前的 entries 是否一致, 不一致则返回失败,
leader 知道失败后, 就会将与这个 follower 的 nextIndex 减一, 然后再做 RPC, 重复上述过程, 直到对每一个 follower 都找到同步的 nextIndex, 然后进行同步(删除 follower 后面的 entry, 换成 leader 后面的 entry )

然后就恢复系统一致性了
在这些机制的加持下, 只要大多数机器(一半以上)可以正常运转, 系统就可以正常运转, 无论此后是新机器加入还是就机器损坏或恢复重新进入, 同步都不是很大的问题

##### 5.4 safety
上述机制已经建立起了一个很好的算法, 但仍然存在一些漏洞
例如一个因为某些原因未写入 committed entry 的 node 成为了 leader,
然后把其他 node 中的 committed entry 通过同步通通删掉...

因此需要一些额外的限制:
![[Pasted image 20231203153615.png]]

###### 5.4.1 election restriction
raft 保证了在 election 时, 只有拥有所有 committed entries 的 candidate 可以胜出成为 leader, 避免了上面的问题

通过 candidate requestVoteRPC 检查 candidate 和被申请的 node 谁更 `up-to-date, 拥有更新的entry` 来决定是否投票, 

检测谁更 `up-to-date` 的方法:
查看最后一个 log 的 term,
如果 voter 最后一个 log 的 term 比 candidate 的大, 那么 voter 是更加 `up-to-date` 的, 因此不投票,
如果 term 相同, 谁有更多的 log 谁就更 `up-to-date`



###### 5.4.2 committing entries from previous terms
还有一种情况是, leader 成功将一个 entry 复制到大多数 follower 中了, 但此时 leader 死了, 造成了本应该 commit 的 entry 保持 uncommitted, 对于新 leader, 应该把这个 entry 视为即将要 commit 的, 并 commit 它
![[Pasted image 20231124134606.png]]

例如图中(c)(d), entry2 多了一个 follower 持有后, 应该被 commit, 但 S5 拥有更新的 entry, 因此可以选举成为 leader, 然后就会开始覆盖应该要 commit 的 entry2, 造成错误,

raft 给出的解决方案是不帮助之前的 leader 进行 commit, 而是使用累计确认的方式, 在新的 entry 添加的同时, 把旧 entry 补上并一起 commit,
累计确认保证了后面的 entry commit 了则前面的也都 commit 了

图中(e)展示了这种情况, 在 entry4 commit 后, S5 也无法选举成为 leader, 解决了这个问题

###### 5.4.3 safety argument
论证了 raft 确实是对的而且是安全的...


##### 5.5 follower and candidate crashes
follower 和 candidate caches 很简单, 只是收不到 AE 以及 RV-reply 而已, 在恢复后, 同样的 AE 会被重新发一遍, 并让他回到正轨, 而 RV-reply 就简单丢掉了

##### 5.6 timing and availability
![[Pasted image 20231219170658.png]]
broadcastTime 是一个 server 发给他的所有 peers 一个 rpc 来回的平均用时, 一般为 5-20ms

MTBF 是一般情况下, 一个 server 发生两次 crashes 之间的时间间隔, 一般为一两个月

因此, 设置 electionTimeout 为 500ms 比较合适

# 6. Cluster membership changes
略...


# 7. Log compaction
在上面所阐述的系统中, log 是不断增长的, 不加限制会耗尽系统资源, 因此需要一些策略来淘汰那些不需要的 logs(也就是 log compaction)

snapshotting 是最简单的方式, 做法就是将当前整个机器的状态(machine state)存储到机器的可靠存储中(磁盘之类的 NV-storage), 然后直接把所有 log 都丢掉(看起来就像给系统做了一个检查点)


![[Pasted image 20231219172052.png]]
简单讲就是把已经 committed 的 log 都进行压缩, 只保留最后的, 有效的 state, 并保留元数据, 例如 lastIncludedIndex, lastIncludedTerm

一旦一个 server 完成一个新的 snapshot 之后, 他就可以把新 snapshot 之前的 log 以及 snapshot 都删了, 只保留这个, 以及新 snapshot 之后的 log

每个 raft 服务器单独进行 snapshot, 只对已经 committed 的 log 进行

同时, 如果有 server 掉线太久, 落后了太多 log, leader 一次次 AE 同步会带来极大开销, 更坏的是, leader 创了 snapshot, 把这个 server 缺失的 log 删了, 就无法重新发给他了

因此引入 installSnapshot RPC, 发 leader 的 snapshot 给落后太多的 server, 提高效率
![[Pasted image 20231219173413.png]]


# 8. Client interaction
