# GPT context
### zookeeper 是什么
```ad-note
Zookeeper 是一个开源的分布式协调服务，用于在大规模分布式系统中管理和协调各种任务和服务。它由 Apache 软件基金会开发和维护。Zookeeper 为分布式应用程序提供了一个高度可靠的协调基础，它主要用于解决分布式系统中的一致性、配置管理、命名服务、分布式锁和分布式队列等问题。

Zookeeper 的设计目标是提供一个简单而强大的分布式协调服务，使得开发人员可以专注于应用程序的逻辑而无需过多关注底层的分布式细节。它采用了一种层次化的命名空间结构，类似于文件系统，可以创建节点（称为 znode）来存储数据，并通过监听机制实现对节点的状态变化进行通知。

Zookeeper 的核心概念包括：

1. 命名空间：Zookeeper 使用类似于文件系统的层次化命名空间来组织数据节点。

2. 数据模型：每个数据节点可以存储一个小于 1 MB 的数据块。

3. Watch 机制：Zookeeper 支持对数据节点的监听，当节点的状态发生变化时，可以触发事件通知给订阅该节点的客户端。

4. 顺序节点：Zookeeper 可以为节点生成全局唯一的顺序标识，用于实现分布式锁和队列等场景。

Zookeeper 在很多分布式系统中被广泛应用，例如 Apache Hadoop、Apache Kafka、Apache HBase 等。它提供了高可用性和一致性，可以有效地帮助开发人员构建可靠的分布式系统。
```

### 分布式协调服务
```ad-note
分布式协调服务是用于管理和协调分布式系统中各个节点之间的通信和合作的一种服务。在分布式系统中，由于节点的数量众多、网络通信的延迟和不确定性等因素，节点之间需要进行信息交换、协作和同步以实现一致性和可靠性。分布式协调服务提供了一组机制和算法，用于解决分布式系统中的一致性、配置管理、状态同步、锁机制和故障恢复等问题。

分布式协调服务的主要作用包括：

1. 一致性管理：分布式协调服务可以帮助系统中的节点达成共识，确保数据的一致性和可靠性。它提供了分布式锁、分布式事务和分布式共享状态等机制，确保节点之间的操作按照一定的顺序和规则进行。

2. 故障检测与恢复：分布式协调服务可以监控和检测分布式系统中的节点故障，并采取相应的措施进行故障恢复。它可以通过心跳机制、选举算法和故障转移等方式来保证系统的可用性和容错性。

3. 配置管理：分布式协调服务可以统一管理分布式系统中的配置信息，包括节点的地址、端口、参数设置等。它提供了配置的动态更新和分发机制，使得系统的配置变更能够快速生效并在所有节点之间保持一致。

4. 命名服务：分布式协调服务可以提供分布式系统中的命名服务，使得节点可以通过名称来访问和识别其他节点或服务。它提供了类似于 DNS（Domain Name System）的机制，将节点的名称映射到对应的网络地址。

5. 分布式队列和任务调度：分布式协调服务可以实现分布式队列和任务调度的功能，用于协调和分配系统中的任务和工作负载。它可以提供队列的创建、入队和出队操作，以及任务的分发和执行跟踪等功能。

总之，分布式协调服务在分布式系统中起着关键的作用，帮助节点之间进行信息交换、协作和同步，以实现系统的一致性、可靠性和可扩展性。它提供了一组通用的机制和算法，简化了开发人员在构建分布式系统时需要处理的复杂问题。
```

### 分布式锁、分布式事务、分布式共享状态
```ad-note
分布式锁、分布式事务和分布式共享状态是分布式系统中常见的机制和概念，用于实现一致性和并发控制。

1. 分布式锁：分布式锁是一种用于协调分布式系统中并发访问共享资源的机制。在分布式系统中，多个节点可能同时访问共享资源，为了避免竞争条件和数据不一致的问题，可以使用分布式锁来保证在任意时刻只有一个节点能够获得对共享资源的独占访问权。分布式锁通常基于分布式协调服务实现，如 Zookeeper 提供了分布式锁的机制。

2. 分布式事务：分布式事务用于确保分布式系统中一组操作的原子性和一致性。在分布式环境中，不同的节点可能需要执行涉及多个资源或数据库的操作，并要求这些操作要么全部成功，要么全部回滚，以保持数据的一致性。分布式事务通过协调并发操作的提交和回滚，以实现全局一致性。常见的分布式事务协议包括两阶段提交（Two-Phase Commit，2 PC）、三阶段提交（Three-Phase Commit，3 PC）和基于日志的补偿性事务（Compensating Transaction）等。

3. 分布式共享状态：分布式共享状态是一种用于在分布式系统中共享和同步数据状态的机制。在分布式系统中，多个节点可能需要访问和修改共享的状态数据，为了保持数据的一致性，需要使用分布式共享状态来协调节点之间的读写操作。分布式共享状态可以是分布式缓存、分布式数据库或分布式存储系统等，它提供了一致性模型和同步机制，确保节点之间的数据访问和更新按照一定的顺序和规则进行。

这些机制在分布式系统中起着重要的作用，帮助解决了分布式环境下的并发控制、一致性和可靠性等关键问题。它们提供了一组标准化的方法和算法，使得开发人员能够更方便地构建可靠的分布式应用程序。
```

### zookeeper 和 raft
```ad-note
Zookeeper 和 Raft 是两个不同的概念，它们在分布式系统中扮演不同的角色。

Zookeeper 是一个分布式协调服务，用于管理和协调分布式系统中的各个节点。它提供了一组机制和算法，用于解决一致性、配置管理、命名服务、分布式锁和分布式队列等问题。Zookeeper 使用 ZAB（Zookeeper Atomic Broadcast）协议来实现一致性，该协议是一种基于原子广播的协议，用于保证节点之间的状态一致性和可靠性。Zookeeper 并不是一种算法，而是一个分布式协调服务的实现。

Raft 是一种共识算法，用于解决分布式系统中的一致性问题。它是一种领导者选举和日志复制的协议，通过确保多个节点之间的一致性来保证系统的可靠性。Raft 算法定义了节点如何选举领导者、如何复制日志以及如何处理故障等情况。它的设计目标是使得分布式系统的一致性问题更易于理解和实现。Raft 算法可以被用于实现分布式协调服务，例如 Zookeeper 就可以使用 Raft 算法来确保节点之间的一致性。

综上所述，Zookeeper 是一个实现了分布式协调服务的软件，而 Raft 是一种共识算法，可以用于实现分布式系统中的一致性。Zookeeper 可以使用 Raft 算法来实现其内部的一致性机制。
```

# linearizability
![[Pasted image 20240311154056.png]]
这里是以 client 的视角给出的, 第一个竖线表示 client request 发出的事件, 第二个竖线是 client response 收到的事件, 中间的网络延迟, 服务器副本, 服务器处理对于 client 都是非透明的
只是对于这样的一系列 `request-response`, 服务器有能力做到 `linearizability`

![[Pasted image 20240311155551.png]]
另一组序列: 对于实现 `linearization` 的系统, 这是不可能出现的, 但是对于有问题的系统, 例如有一个副本没有复制这个信息, 但这个副本就使用过时的信息回复 client, 那么这个系统是非 `linearization` 的

# zookeeper
zookeeper 提供了类似 raft 的服务, 

### zookeeper guarantees
