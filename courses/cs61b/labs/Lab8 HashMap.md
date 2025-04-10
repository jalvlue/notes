编写底层实现是哈希表的 map
具体来说就是编写 HashMap, 跟 lab7 一样实现 `Map61B` 接口

# coding


### 核心私有成员变量
- `private Collection<Node>[] buckets`, 哈希桶, 存储实际节点
- `private int numItems`, 当前表中 kv 对的数量
- `private int numBuckets`, 当前表中哈希桶的个数
- `private double loadFactor`, 表的负载因子


### void clear()
- 将 `numItems, numBuckets` 设置回默认
- 重新给 `buckets` 一个初始化桶数组, 原来的交给 GC


### boolean containsKey(key)
- 直接调用 `get(key)` 来判断


### V get(key)
- 通过 `key.hashCode()` 获取对应存储节点 `node` 所在的桶
- 在桶中获取节点, 然后取回节点值 `node.value`


### int size()
- 直接返回 `numItems`


### void put(key, value)
- 调用 `get(key)` 尝试获取, 判断是更新还是新插入
- 根据 `key.hashCode()` 计算哈希桶位置
- 调用桶的方法, 构造一个节点让桶插入
- 这里实现了一个私有的 `resize()` 函数, 负载因子超过阈值之后进行哈希桶扩容 (翻倍)
- `resize()` 利用了迭代器, 将原桶内数据全都复制到了新桶内


### Set<> keySet()
- 直接使用 `java.util.HashSet`
- 利用迭代器将 `key` 放进集合中


### iterator()
```java
    private class HashIterator implements Iterator<K> {
        private int currCount;
        private int bucketOffset;
        private int iterNumItems;
        private Iterator<Node> bucketIter;

        public HashIterator() {
            this.currCount = 0;
            this.bucketOffset = 0;
            this.iterNumItems = numItems;
            this.bucketIter = buckets[this.bucketOffset].iterator();
        }

        @Override
        public boolean hasNext() {
            return this.currCount < this.iterNumItems;
        }

        @Override
        public K next() {
            this.currCount += 1;
            if (this.bucketIter.hasNext()) {
                return bucketIter.next().key;
            }

            do {
                bucketOffset += 1;
            } while (buckets[this.bucketOffset].isEmpty());

            bucketIter = buckets[this.bucketOffset].iterator();
            return bucketIter.next().key;
        }
    }
```

- 实现私有迭代器
	- 内部使用每个 `bucket` 的迭代器对每个桶进行迭代


### remove(key)
- 先调用 `get(key)` 尝试获取
- 如果拿到了, 就到对应的桶内把节点删了
- 这里有个小坑: 由于在桶内直接调用 `Collection<Node> bucket.remove()`, 因此需要传入该删除节点的引用, 桶内部的实现似乎是根据引用判断, 传入新构建的, 数值完全相同的节点无法删除
