- 这里想要表达的变与不变量可以看成是引用类型和原子类型的区别


# 02
- `insert_itmes`, 迭代直接插入呗
- 注意这里会在遍历列表的同时修改列表, 简单地使用 `for ... in .., iterator` 可能会出问题, 因此使用下标索引操作

```python
def insert_items(s, before, after):
    """Insert after into s after each occurrence of before and then return s.

    >>> test_s = [1, 5, 8, 5, 2, 3]
    >>> new_s = insert_items(test_s, 5, 7)
    >>> new_s
    [1, 5, 7, 8, 5, 7, 2, 3]
    >>> test_s
    [1, 5, 7, 8, 5, 7, 2, 3]
    >>> new_s is test_s
    True
    >>> double_s = [1, 2, 1, 2, 3, 3]
    >>> double_s = insert_items(double_s, 3, 4)
    >>> double_s
    [1, 2, 1, 2, 3, 4, 3, 4]
    >>> large_s = [1, 4, 8]
    >>> large_s2 = insert_items(large_s, 4, 4)
    >>> large_s2
    [1, 4, 4, 8]
    >>> large_s3 = insert_items(large_s2, 4, 6)
    >>> large_s3
    [1, 4, 6, 4, 6, 8]
    >>> large_s3 is large_s
    True
    """
    "*** YOUR CODE HERE ***"
    i = 0
    while i < len(s):
        if s[i] == before:
            i += 1
            s.insert(i, after)
        i += 1
    return s

```



# 07
- `sprout_leaves`
- 这里的本质是遍历整个树, 在叶子节点上加东西, 考虑到最后需要返回一颗新的树, 因此每次处理后构造为新的树然后返回
- 两种写法
```python
def sprout_leaves(t, leaves):
    """Sprout new leaves containing the labels in leaves at each leaf of
    the original tree t and return the resulting tree.

    >>> t1 = tree(1, [tree(2), tree(3)])
    >>> print_tree(t1)
    1
      2
      3
    >>> new1 = sprout_leaves(t1, [4, 5])
    >>> print_tree(new1)
    1
      2
        4
        5
      3
        4
        5

    >>> t2 = tree(1, [tree(2, [tree(3)])])
    >>> print_tree(t2)
    1
      2
        3
    >>> new2 = sprout_leaves(t2, [6, 1, 2])
    >>> print_tree(new2)
    1
      2
        3
          6
          1
          2
    """
    "*** YOUR CODE HERE ***"
    if is_leaf(t):
        return tree(label(t), [tree(leaf) for leaf in leaves])

    return tree(label(t), [sprout_leaves(b, leaves) for b in branches(t)])

    # if is_leaf(t):
    #     return tree(label(t), [tree(leaf) for leaf in leaves])
    # new_branches = []
    # for b in branches(t):
    #     new_branches.append(sprout_leaves(b, leaves))

    # return tree(label(t), new_branches)

```
