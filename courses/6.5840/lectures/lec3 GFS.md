why distributed? --> performance
performance --> sharding
sharding --> failures
failures --> tolerance
tolerance --> replication
replication --> inconsistency
consistency --> low performance

well/strong consistency: 分布式系统的行为就像单个 server 一样

# write data: record append
在 append 中, 与直接 write 类似, 只是 client 并不知道一个 file 的末尾 chunk 位置, 因此需要向 master 询问这个位置