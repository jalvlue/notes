# 0. abstract
zookeeper: 关键的基础设施, 用于协调分布式应用程序进程的服务

```ad-note

当提到“用于协调分布式应用程序进程的服务”，它指的是一种中间件或基础设施，用于管理和协调在分布式系统中运行的多个应用程序进程之间的交互和通信。


以下是一些示例，说明了协调服务在分布式应用程序中的应用：

1. 分布式锁：协调服务可以提供分布式锁，确保在分布式环境中对共享资源的互斥访问。例如，在一个分布式任务调度系统中，多个任务处理进程可能需要互斥地访问某个共享资源，通过分布式锁的协调，只允许一个进程同时访问该资源。

2. 配置管理：协调服务可以用于管理分布式应用程序的配置信息。例如，当某个应用程序的配置参数发生变化时，协调服务可以通知所有相关的应用程序进程更新其配置，以确保它们都使用最新的配置。

3. 领导者选举：在某些分布式系统中，需要选择一个领导者来负责协调和处理特定的任务。协调服务可以提供领导者选举的机制，确保只有一个进程被选为领导者，并在领导者发生故障时重新选举新的领导者。

4. 分布式队列：协调服务可以提供分布式队列，用于在多个应用程序进程之间传递消息或任务。例如，一个分布式消息处理系统可以使用分布式队列来接收和分发消息，确保消息的有序处理和可靠传递。

5. 分布式协调原语：协调服务可以提供一些基本的协调原语，如分布式计数器、分布式位图等，用于实现更复杂的分布式算法和协议。

总之，协调服务在分布式系统中起到了连接和协调多个应用程序进程的重要作用，帮助它们实现共享资源管理、数据一致性、任务调度和通信等功能。ZooKeeper 就是这样一种协调服务，它提供了一组功能强大的原语和接口，使得开发者可以方便地构建和管理复杂的分布式应用程序。
```

- 提供简单但是高性能的核心实现, 让 client 可以构建更复杂的协调原语
- 群组消息传递、共享寄存器、分布式锁服务集成到复制的集中式服务中
- 提供的 API 具有共享寄存器的无等待特性, 以及类似分布式文件系统的缓存失效的事件驱动机制
提供了简单而强大的协调服务

# 1. introduction
大规模分布式系统需要多种不同形式的协调, 协调是指在分布式系统中的多个进程之间进行交互和通信，以实现一致性、资源管理、任务调度等目标:
1. 配置管理：分布式应用程序可能需要动态管理配置参数，包括系统的操作参数和运行时参数
2. 组成员和领导者选举：分布式系统中，进程需要了解其他进程的状态和负责的任务，以及选择一个领导者来协调和处理特定的任务
3. 锁机制：分布式系统中，进程之间需要协调对共享资源的互斥访问，以避免数据竞争和冲突。

```ad-note
在文章的背景下，"coordination primitive"（协调原语）指的是一组基本的操作或接口，用于实现分布式系统中的协调需求。它们是构建协调服务的基础组件，可以用来管理共享资源、实现一致性、进行通信和协调进程之间的交互。

协调原语可以被看作是一种抽象的编程接口，它提供了一些操作和语义，使得开发人员可以通过这些接口来实现复杂的分布式协调功能。例如，常见的协调原语包括分布式锁、分布式队列、分布式计数器等。

可以将协调原语理解为协调服务的构建块。协调服务则是基于这些原语的实现，提供了更高级别的功能和语义，以满足具体的协调需求。因此，协调服务可以被视为在协调原语之上构建的一种更高层次的抽象。

在ZooKeeper的背景下，ZooKeeper提供了一组协调原语，包括有序节点、临时节点、观察机制等，用于构建分布式协调功能。这些原语允许开发人员实现各种高级别的协调方案，如领导者选举、分布式锁、配置管理等。因此，协调原语在这里可以被视为ZooKeeper提供的接口和操作，用于实现分布式系统中的协调性质和需求。

总结来说，协调原语是一组基本的操作或接口，用于实现分布式系统中的协调功能，而协调服务则是在这些原语之上构建的更高级别的抽象，提供了特定的协调功能和语义。
```

传统协调方法是为每个不同的协调需求开发专门的服务, 

zookeeper 的设计理念是: 
选择在服务端提供一个 API，允许应用程序开发人员实现自己的协调原语。这种设计决策使得 ZooKeeper 成为一个协调服务内核，为开发人员提供了构建更高级别协调原语的灵活性。这种方法允许根据应用程序的需求实现多种形式的协调，而不限制开发人员使用固定的原语集合。

同时 zookeeper 放弃了 `blocking primitives` 阻塞原语, 进而提供了:
`Our system, Zookeeper, hence implements an API that manipulates simple waitfree data objects organized hierarchically as in file systems. In fact, the ZooKeeper API resembles the one of any other file system.`

对所有客户端操作提供了 `FIFO` 保证和可线性化写保证

本文贡献:
- coordination kernel: zookeeper
- coordination recipes: zookeeper for building higher level coordination primitives
- zookeeper performance and experience

# 2. the zookeeper service
### 2.1 service overview
![[Pasted image 20240311144029.png]]
ZK 服务器内部存储一个 `data-tree` 来组织 `wait-free data objects`
客户端与 ZK 服务器通过 `session, request` 进行交互, 客户端通过调用 ZK API 来修改 `data-tree, znode`

客户端可以创建两种 `znode`:
1. Regular: 客户端显式掌握创建和删除
2. Ephemeral: 客户端创建, 显式删除或让系统自动删除==???==

request flags:
1. sequential: ==???==
2. watch: 带有 watch 标志的, 例如 getData, ZK 会帮助 client 监视这个数据, 并在数据发生更改时通知 client(避免了 client 轮询)

data model:
树形结构可以很好地区分子树, 然后每个子树都可以用来做特定的服务, 例如上面 Figure1, `/app1` 可以是一个 `group membership` 服务, 所有挂载的 `/app1/p_1...` 都可以看作是一个在线的成员
znode 并不用作通用存储, 只支持存储一些元数据和配置信息

session:
client 通过建立 session 来与 ZK 服务器进行交互, 
```ad-note
在ZooKeeper中，一个会话代表着客户端与ZooKeeper服务器之间的连接和交互。当客户端与ZooKeeper建立会话时，它可以执行各种操作，比如创建、读取、更新和删除数据节点等。

ZooKeeper集群由多个服务器组成，这些服务器共同协作来提供高可用性和容错性。每个服务器都存储了相同的数据副本，并且在集群中彼此通信以保持数据一致性。

通过会话机制，客户端可以在ZooKeeper集群中透明地从一个服务器切换到另一个服务器。这意味着客户端可以在不知道具体服务器地址的情况下，继续与ZooKeeper进行交互。无论客户端连接到哪个服务器，它都可以执行相同的操作，并且在整个集群中保持一致的状态。

当客户端与一个服务器建立会话时，它会持续接收来自该服务器的状态变化通知。这些状态变化反映了客户端操作的执行结果，如数据节点的创建、更新或删除。客户端可以根据这些状态变化来判断操作的成功与否。

因此，会话使得客户端可以在ZooKeeper集群中切换服务器，而不会丢失会话状态或操作的一致性。无论客户端连接到哪个服务器，它都可以继续操作，并且对于客户端来说，这个过程是透明的。这种持久性和透明性是通过会话机制实现的。
```

### 2.2 client API
一组 ZK API:
- create(path, data, flags)
- delete(path, version): Deletes the znode path if that znode is at the expected version;
- exists(path, watch): Returns true if the znode with path name path exists, and returns false otherwise. The watch flag enables a client to set a watch on the znode;
- getData(path, watch): Returns the data and meta-data, such as version information, associated with the znode. The watch flag works in the same way as it does for exists(), except that ZooKeeper does not set the watch if the znode does not exist;
- setData(path, data, version): Writes data[] to znode path if the version number is the current version of the znode;
- getChildren(path, watch): Returns the set of names of the children of a znode;
- sync(path): Waits for all updates pending at the start of the operation to propagate to the server that the client is connected to. The path is currently ignored.

```ad-note
所有方法在API中都有同步和异步版本可用。当应用程序只需要执行单个ZooKeeper操作且没有并发任务需要执行时，它可以使用同步API。在这种情况下，应用程序会进行必要的ZooKeeper调用，并阻塞等待操作完成。

然而，异步API允许应用程序同时执行多个未完成的ZooKeeper操作和其他任务。使用异步API，应用程序可以在执行ZooKeeper操作的同时并行执行其他任务。

ZooKeeper客户端保证为每个操作调用相应的回调函数，并按照操作的顺序进行调用。这意味着无论异步操作的顺序如何，ZooKeeper客户端都会确保在应用程序中按照操作的顺序调用相应的回调函数。

总结起来，同步API适合于单个操作且无并发任务的场景，而异步API适合于并发执行多个操作以及同时执行其他任务的场景。异步API通过回调函数的方式来处理操作的结果和状态，并确保按照操作的顺序调用相应的回调函数。
```

znode version: version 是每一个 znode 的元数据, 有一些操作需要对 znode version 进行检查


### zookeeper guarantees
1. Linearizable writes: all requests that update the state of ZooKeeper are serializable and respect precedence;
2. FIFO client order: all requests from a given client are executed in the order that they were sent by the client

