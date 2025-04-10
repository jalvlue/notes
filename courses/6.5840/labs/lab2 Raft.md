![[Pasted image 20240307201353.png]]

# 2A
![[Pasted image 20231213131931.png]]


# 2B
### AE
对于 `leader`, 
`nextIndex []int` 是保存每一个下一个要发给每一个 `peer` 的 `index`, 初始化为 `leader.commitIndex + 1`, 表示 `leader` 上任后假设所有人的 `log` 都跟 `leader` 一样, 后续再慢慢通过几轮 `AE` 调整

`matchIndex []int` 是保存每一个 `peer` 与 `leader` 最高相同的 `log entry index`,
因为采用累计确认, 可以保证 `matchIndex` 之前的所有 `log entry` 都与 `leader` 相同
初始化为 0, 后续通过几轮 `AE` 调整

##### AE: 0 系统刚启动, 新 leader 和所有 peer log 都是空的 empty log heartbeat...heartbeat
```go
0. leader.setLeader()
1. nextIndex[]设为 1, matchIndex[]设为 0
2. leader.doHeartBeat()
ni = 1
prevLogIndex = 0
prevLogTerm = -1
entries = []
matchIdx = 0
leaderCommit = 0
3. follower.AppendEntries()
reply.Success = true
4. leader.doHeartBeat() 收集结果
...
```


##### AE: 1 leader 收到第一个log
1. rf.Start 加入一个新的 log entry -> log[1]
2. rf.heartbeatTicker -> rf.doHeartbeat
3. args, reply -> sendAppendEntries
```go
AE leader args:
ni = 1
prevLogIndex = 0
prevLogTerm = -1
entries = [0:] ==> log[1]
```
4. rf.AppendEntries
```go
len(follower.log) == 0 ->
// just simply append leader's log
follower.log = append(follower.log, entries)
reply.Success = true
```
5. rf.doHeartbeat: 收集结果
```go
leader.matchIndex[follower] = ni + len(entries) - 1 ==> 1
range throuth entries, check mojority replicated for commit
```

##### AE: 2 leader 收到第二个 log
```go
0. leader.Start()
leader.log[1, 2]
1. leader.doHeartbeat()
ni = 2
prevLogIndex = 1
prevLogTerm = 1
entries = leader.log[1:] ==> leader.log[2]
leaderCommit = 1
2. follower.AppendEntries()
matchIdx = 1
reply.Success = true
replicate....
commit: 1
```


##### AE 2: 系统已经启动, 新 leader 跟 follower 之间可能有 mismatched log
...
leader: 
log[1，2，3，4，5，6] 
leaderCommit: 6
nextIndex[7，7]
matchIndex[0，0]

F 1: 
log[1，2，3，4，5，6]
followerCommit: 6

F 2: 
log[1]
followerCommit: 1
```go
1. leader.doHeartbeat()
ni = 7
prevLogIndex = 6
prevLogTerm = 1
entries = []
leaderCommit = 6
2. F2.AppendEntries() ==> reply.Success = false
3. leader.doHeartbeat() ==> leader.nextIndex[F2]--
......

4. leader.doHeartbeat()
ni = 2
prevLogIndex = 1
prevLogTerm = 1
entries = [2, 3, 4, 5, 6]
leaderCommit = 6
5. F2.AppendEntries()
matchIdx = 1

```

### RV
##### RV 1: 系统刚启动, 所有 follower 都没有 log
1. candidate.startElection
```go
RV args:
candidateTerm: 1
lastLogIndex: -1
lastLogTerm: -1
```
2. follower.RequestVote
比较谁的 last 更新(比 term, 再比index)


### TestBackup 2B
```go
000. 5个server, leader: 0, heartbeat...heartbeat
001. new log append to leader [0] ==> AE & commit
002. nextIndex[2...], matchIndex[1...], commitIndex[1...]

disconnect 3 followers [2 3 4]
leader[0] & follower[1] in partition
003. 50 new logs append to leader [0] ==> replicated to follower [1] but not commit
[0].commitIndex = 1, log[1,...,51]
[1].commitIndex = 1, log[1,...,51]
004. AE...AE...

disconnect leader [0] and follower [1]
reconnect follower [2 3 4] ==> re-election & commit logs
005. leader: 2, term: 3, nextIndex[2...], commitIndex = 1
heartbeat...heartbeat
006. 50 new logs append to leader [2] ==> replicated to follower[3 4] & commit
[2].commitIndex = 51, log[1,...,51]
[3].commitIndex = 51, log[1,...,51]
[4].commitIndex = 51, log[1,...,51]

disconnect follower [3]
put leader[2] & follower[4] in partition
007. 50 new logs append to leader [2] ==> replicated to follower[4] but not commit
[2].commitIndex = 51, log[1,...,101]
[4].commitIndex = 51, log[1,...,101]

disconnect all servers
re-connect leader[0] and follower [1 3]
008. 1 new log append to leader [0]
[0].commitIndex = 1, log = [1,...,52]
```

### data race
```go
goroutine 7227: created at raft.go:465
goroutine 7229: created at labrpc.go:239

```

logs
```go
P0: leader: [1], append & commit 1 log on all servers

P1: disconnect [3 4 0], leader[1] & follower[2] in partition
50 logs append to leader[1] & follower[2] but not commit
[1].commitIndex = 1, logs[1,..., 51]
[2].commitIndex = 1, logs[1, ..., 51]

P2: disconnect leader[1] & follower[2], reconnect [3 4 0] to re-election and commit logs
leader: [0], commitIndex = 1

50 logs append to leader[0] & followers and commited
[0].commitIndex = 51, logs[1, ..., 51]
[3].commitIndex = 51, logs[1, ..., 51]
[4].commitIndex = 51, logs[1, ..., 51]

P3: disconnect follower[3], put leader[0] & follower[4] in partition
50 logs append to leader[0] but not commit
[0].commitIndex = 51, logs[1,..., 51,..., 101]
[4].commitIndex = 51, logs[1,..., 51,..., 101],

P4: disconnect all servers, reconnect leader[1] & follower [2, 3], to append and commit logs
leader: [3] syncing log with follower[1, 2] + new log [52...]

[3].commitIndex = 53, logs[1,..., 103]
[1].commitIndex = 53, logs[1,..., 103]
[2].commitIndex = 53, logs[1,..., 103]


```

race scope
```go
[2] leader, term_3
[3] leader, term_33

AE: [2] --> [0]
AE: [2] --> [1]
AE: [2] --> [3]
AE: [2] --> [4]

AE: [3] --> [0]
AE: [3] --> [1]
AE: [3] --> [2]
AE: [3] --> [4]

[3] reply from [0]
[3] reply from [1]

[2] receive AE from [3]

[3] reply from [4]


```

![[Pasted image 20231220162349.png]]


# 2C

### figure82C
```go
leader: [0]
append & commit log[1, 1] on all servers

newLog[1, 2] append to leader[0]
crash [0]

leader: [2]
newLog[2, 2] append to leader[2]
crash [2]

leader: [4]
newLog[3, 2] append to leader[4]
crash [4]

restart [2]
leader: [2]
log[1, 2] append & commit to follower[1][3]
newLog[5, 3] append to leader[2]
crash [2]

start [4]



```


### faster backup
```go
case1:
log 1 2 3 4
S1: 4 5 5
L2: 4 6 6 6

L2 AE: lastLogTerm=6, lastLogIndex=4
S1 AE: 
entryIndex=3, entryTerm=5
reply.Success=false, 
reply.XTerm=5
reply.XIndex=3
reply.XLen=3

L2 doHeartbeat:
L2.nextIndex[S1]=2


case2:
log 1 2 3 4
S1: 4 4 4
L2: 4 6 6 6

case3:
log 1 2 3 4
S1: 4
L2: 4 6 6 6

case4:
log 1 2 3 4
S1: 4 6
L2: 4 6 6 6
```

![[Pasted image 20240307202417.png]]



# 2D

