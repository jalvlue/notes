- Plan re-learn cs
	- `hugo + github.io hosting`
	- `cs61a: sicp`
	- `15-213/csapp: ics`
	- `cs61c: computer architecture`
	- `6.1810: operating system`
	- `15445 + 6.5830: database system`
	- `cs144: computer network`
	- `6.5840: distributed system`
	- `6.858: computer system security`


```ad-info
可以写一个简单的rpc，从server对client的高性能处理开始，你会学到go关于底层tcp的处理，也能学到常见web框架是怎么处理http请求的。接下来处理client，你能学到客户端是怎么在rpc里保持连接并接受rpc调用结果的过程。接下来你需要设计一个序列化/反序列化的方法，这里面你能熟悉io/buf的各种util使用方法。再设计一个简单的tlv协议，这时候你就会对所谓的tcp沾包问题嗤之以鼻了。一个简单的rpc实现完之后，你可以给它设计一套服务发现体制，你会了解到怎么去适配cousul，etcd等分布式kv库，或者自己搞一套简单的服务发现，你会学到心跳机制的保持，多播的作用吧啦吧啦。最后你会设计一个负载均衡，这时候背的八股文一致性hash又能派上用处啦。。。这样下来，一个rpc设计完毕，你面试字节也没啥问题了。

要是觉得复杂，写个di也很有意思，可以熟悉go的反射机制。或者写个orm，这会让你熟悉对不同数据库和sql的抽象。

再不济，写个基于paxos，raft的分布式kv也是非常容易入门的嘛。

```
