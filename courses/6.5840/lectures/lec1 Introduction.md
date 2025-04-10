main topics:

infrastructure for applications.
storage(着重), 
communication
computation(着重)

--> abstractions
目标就是为 app 建立抽象接口, 将复杂的分布式存储或计算细节隐藏起来

Topic: fault tolerance
大型系统大型网络多个机器总是会有机器 failed,
但是希望见到的是隐藏这些 failures
并在出现 failures 后还可以继续提供服务(高可用(high availability))

big idea: replicated servers, 复制接力


Topic: consistency
大型的系统都要有比较好的行为(统一的行为)
例如: 访问同一数据需要统一
但在 replicated servers 下很难保证完全的统一相同


Topic: performance
大型系统要求性能是可拓展的, 越多机器越好性能
但越多机器越难拓展


Topic: tradeoffs
Fault-tolerance, consistency, and performance are enemies.
需要在这些之间权衡
例如: fault tolerance 和 consistency 都需要 communication, 但 communication 开销大难拓展, 因此就需要权衡哪个更重要


Topic: implementation
RPC, threads, concurrency control, configuration.


# case study: MapReduce
