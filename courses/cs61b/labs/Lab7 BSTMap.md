编写底层实现是 BST 的 map
具体来说要让 BSTMap 要实现这个接口
```java
package bstmap;

import java.util.Set;

/* Your implementation BSTMap should implement this interface. To do so,
 * append "implements Map61B<K,V>" to the end of your "public class..."
 * declaration, though you can use other formal type parameters if you'd like.
 */
public interface Map61B<K, V> extends Iterable<K> {

    /** Removes all of the mappings from this map. */
    void clear();

    /* Returns true if this map contains a mapping for the specified key. */
    boolean containsKey(K key);

    /* Returns the value to which the specified key is mapped, or null if this
     * map contains no mapping for the key.
     */
    V get(K key);

    /* Returns the number of key-value mappings in this map. */
    int size();

    /* Associates the specified value with the specified key in this map. */
    void put(K key, V value);

    /* Returns a Set view of the keys contained in this map. Not required for Lab 7.
     * If you don't implement this, throw an UnsupportedOperationException. */
    Set<K> keySet();

    /* Removes the mapping for the specified key from this map if present.
     * Not required for Lab 7. If you don't implement this, throw an
     * UnsupportedOperationException. */
    V remove(K key);

    /* Removes the entry for the specified key only if it is currently mapped to
     * the specified value. Not required for Lab 7. If you don't implement this,
     * throw an UnsupportedOperationException.*/
    V remove(K key, V value);

}
```

# coding
- `clear(), containsKey(key), get(key), put(key, val)` 这几个方法比较简单, 都是之前写的了
- 着重记录 `keySet(), iterator(), remove(key)` 者几个方法

### iterator
- 编写一个私有类, 实现了 `iterator<T>` 接口
- 由于要返回一个能够通过 `next()` 方法不断迭代的迭代器, 因此就不能使用递归了
- 所以使用一个栈来模拟递归操作
```java
    private class BSTIterator implements Iterator<K> {
        private final Stack<BSTNode> stack;

        public BSTIterator(BSTNode root) {
            this.stack = new Stack<>();
            this.pushAllLeft(root);
        }

        private void pushAllLeft(BSTNode node) {
            while (node != null) {
                this.stack.push(node);
                node = node.left;
            }
        }

        public boolean hasNext() {
            return !this.stack.isEmpty();
        }

        public K next() {
            BSTNode node = this.stack.pop();
            pushAllLeft(node.right);
            return node.key;
        }
    }

    @Override
    public Iterator<K> iterator() {
        return new BSTIterator(this.root);
    }
```

### keySet
- 返回一个包含所有 key 的 `Set<K>`
- 这里选择直接用标准库里的 `HashSet`
- 配合上面实现的迭代器将所有 key 放进 `HashSet` 中

```java
    @Override
    public Set<K> keySet() {
        HashSet<K> keys = new HashSet<>();
        Iterator<K> iter = new BSTIterator(this.root);

        while (iter.hasNext()) {
            keys.add(iter.next());
        }

        return keys;
    }
```

### remove
- 删除节点使用递归实现删除, 分三种情况讨论
	- 删除叶子节点, 直接返回 null, 让它的父节点不再引用它
	- 删除只有一个子树的节点, 直接返回该非空子树, 让它的父节点跳过它直接引用它的子树
	- 删除有两个子树的节点, 选出左子树最大节点作为新节点, 串联好引用关系, 返回新节点

```java
    @Override
    public V remove(K key) {
        if (key == null) {
            throw new IllegalArgumentException();
        }

        V ret = this.get(key);
        if (ret == null) {
            return null;
        }

        this.size -= 1;
        this.root = this.removeKeyNode(this.root, key);
        return ret;
    }

    private BSTNode removeKeyNode(BSTNode curr, K key) {
        if (curr == null) {
            return null;
        }

        int cmp = key.compareTo(curr.key);
        if (cmp == 0) {
            // remove this node
            return deleteNode(curr);
        } else if (cmp < 0) {
            // key < curr.key
            curr.left = removeKeyNode(curr.left, key);
        } else {
            // key > curr.key
            curr.right = removeKeyNode(curr.right, key);
        }

        return curr;
    }

    private BSTNode deleteNode(BSTNode node) {
        if (node.left == null) {
            return node.right;
        }
        if (node.right == null) {
            return node.left;
        }

        BSTNode newRoot = detachMax(node.left);
        if (newRoot == node.left) {
            newRoot.right = node.right;
            return newRoot;
        }

        newRoot.left = node.left;
        newRoot.right = node.right;
        return newRoot;
    }

    private BSTNode detachMax(BSTNode node) {
        BSTNode prev = node;
        while (node.right != null) {
            prev = node;
            node = node.right;
        }

        if (prev == node) {
            return node;
        }

        prev.right = node.left;
        return node;
    }
```

