# A
![[Pasted image 20240328135105.png]]

# B

### TestStatic
不考虑 `shard migration` 已经能通过测试, 

现在考虑 `shard migration`

```
config [1]: shards[100, 100, ..., 100], groups[100]
config [2]: shards[101, 101, ..., 100], groups[100, 101]
```
从 `config[1]` 更新到 `config[2]` 时, 有 `shard migration` 需要进行

```
GID[100]: leaveShards[5,6,7,8,9]
GID[101]: inShards[0,1,2,3,4]
```


### TestMissChange
```
config[1]: shards[100,100,...,100], groups[100]
config[2]: shards[101,101,...,100], groups[100,101]
config[3]: shards[102 102 101 101 101 102 100 100 100 100], groups[100,101,102]
config[4]: shards[102 102 102 100 102 102 100 100 100 100], groups[100,102]
config[5]: shards[102,102,...,102], groups[102]
config[6]: shards[101,101,...,102], groups[101,102]
config[7]: shards
```

### TestConcurrent1
```
join(0)
config[1]: shards[100,100,...,100], groups[100]

join(1)
config[2]: shards[101,101,...,100], groups[100,101]

join(2)
config[3]: shards[102 102 101 101 101 102 100 100 100 100], groups[100,101,102]

leave(0)
config[4]: shards[102 102 101 101 101 102 101 102 101 102], groups[101,102]

config[5]: shards[102,102,...,102], groups[102]
config[6]: shards[101,101,...,102], groups[101,102]
config[7]: shards
```


### TestConcurrent3



### challenge 1: GC
```
			Cfg n      Cfg n+1
group A    lose [1]    [1]: StatusBePulling -> StatusInvalid
group B    gain [1]    [1]: StatusPulling -> StatusGCing -> StatusServing
```
获得 shard 的 group B leader 在 config change 中将 shard 状态转为 `StatusPulling`, 交由 migrate shards 处理, migrate shards 中将状态转为 `StatusGCing`, 交由 GC 处理, GC 通知 group A leader 迁移成功, 不需要再保留副本了转为 `StatusInvalid`, 最后通知成功后再将状态转为 `StatusServing`


![[Pasted image 20240401210324.png]]
