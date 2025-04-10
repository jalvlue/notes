# distributed concensus
假设有一个 client 和一个存储单一键的 database server, 
只有单个 server 时, client 向 server 发送一个键值, 并跟 server 达成共识(concensus, 即将这个值存进 server 中), 是相对简单的(client-server 一对一信息传递)
![[Pasted image 20230927144355.png]] 
![[Pasted image 20230927144401.png]]

但很多 database 使用分布式架构, 因此在多个分布的 database 之间同步信息就是一个有一点困难的问题(称为 distributed concensus)


# Raft Protocol
raft 就是一个用来解决 distributed concensus 问题的协议


# High level overview
在 raft 协议中, node(分布式架构中一个节点, 例如上面的一个 database, 也可以是其他的分布式架构节点)
每个 node 有三个状态
1. Follower state
2. Candidate state
3. Leader state

### leader election
在最开始时, 所有的 node 都处于 follower state(无 leader 的 follower )

如果一个 follower 没有 leader, 那么它将自动成为一个 candidate

candidate 会向其他 nodes 发出投票请求(vote request), 其他 notes 会对这个请求做出回应(选/不选), 如果获得了多数的选票, 那么这个 candidate 就会成为一个 leader


### log replication
系统选出 leader 之后, 对系统的修改都要通过 leader 完成,
每次修改都会写入到 leader 的 log 的一个 entry 中

log 还没提交(commit)时, 修改不会写入到其他 nodes,

要提交时, leader 首先会将这个 log entry 复制到其他 nodes(followers) 
followers 再将复制完成信息发回给 leader

==大多数== followers 都复制完成后(对系统的状态达成共识),
leader 正式提交修改, 并改变它的数据
![[Pasted image 20230927150706.png]]
然后再通知 followers commit 信息, followers 正式提交并改变数据
![[Pasted image 20230927150803.png]]
此时, 就达成了 distributed concensus



# Leader Election
在 leader election 中, 有两个 timeout 设置
1. election timeout: follower 等待成为 candidate 的时间(随机设置在 150ms-300ms)(上面的 overview 中简单说了没有 leader 就会自动称为 candidate 是一种简化, 实际需要经过一个 timeout )
2. heartbeat timeout

##### election timeout
在第一个 follower 的 election timeout 结束后, 它成为第一个 candidate(黑虚圈), 并开始一个新的选举期(election term), 给自己投一票, 然后做出 vote request
如果其他节点在这个选举期还没投票, 他么它将会投给这个 candidate, 然后重置它的 election timeout
![[Pasted image 20231121190131.png]]
![[Pasted image 20231121190221.png]]
这个 candidate 获得多数票后, 它就成为了 leader (黑圈)

##### heartbeat timeout
成为 leader 后, 它将承担 leader 的责任, 所有 client 的修改请求都会以 log 的形式, 通过 leader 与其他 node 达成共识, 然后提交

它将会以 heartbeat timeout 为间隔向它的 followers 发送 Append Entries message, follower 则将会回复信息, 并重置 election timeout, 

这个选举期将一直持续, 直到 leader 不再发送 message(可能是 leader failure)
有一些 follower 达到 election timeout 成为一个 candidate, 然后进入一个新的选举期(Term 2)
选出新的 leader, 继续系统的工作
![[Pasted image 20231121190747.png]]


随机性导致了可能有多个 follower 同时成为 candidate, 并且获得相同多的选票,
![[Pasted image 20231121191011.png]]
那么他们不会成为 leader, 而是会继续执行 election timeout, 直到有 node 走完 election time 成为 candidate 并成为 leader 
![[Pasted image 20231121191057.png]]
![[Pasted image 20231121191137.png]]


这样的设计保证了在每个选举期内, 整个系统只有一个 leader 


# Log Replication
log replication 是 leader 向 followers 同步 log entries 的过程,
与 heartbeat 一起使用同一个消息传递(一次 leader-follower 网络信息传输会同时做 heartbeat 和 log replication)


具体过程:

1. client 向 leader 发出修改请求, 并在 leader 的 log 中新增一个 entry 
![[Pasted image 20231121191513.png]]

2. ==在下一次 heartbeat 中==, client 把这个 entry 附加在消息中, 发送给 followers, 让他们也写入一个 entry 
![[Pasted image 20231121191631.png]]

3. ==大多数== followers 都尝试将 entry 写入到 log , 并回复消息给 leader, 此时 leader 如果获得==大多数==followers 的成功回复, leader 就把这个 entry 认为 committed 
![[Pasted image 20231121191804.png]]

4. leader 将修改的 entry 认为 committed 后, 给 client 回复成功的消息, 并在下一次 heartbeat 中, 让 followers 也认为成功 committed 
![[Pasted image 20231121192214.png]]



# 可用性
raft 同时还保证了 node 之间网络出现隔离(分成几个小网络)
系统仍可正常运行
![[Pasted image 20231121192600.png]]
在这里, B 本来是全局 leader, 但是突然出现了网络隔离
上半部分重新选举并进入 Term2, 此时系统中出现了两个隔离的 leader

一旦故障被修复, 隔离结束后, 处于较低 term 的会回滚他们的 log(这些 log 一般都是无法 commit 的, 因为无法获得 majority ACK),
并采用高 term leader 的 log
