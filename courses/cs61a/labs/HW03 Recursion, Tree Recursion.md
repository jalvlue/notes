# 01
- `num_eights(n)`, 计算 `n`  中有多少个 `8`
- 对表达式的使用有一些限制, 需要使用聪明的递归
```python
def num_eights(n):
"""Returns the number of times 8 appears as a digit of n.

>>> num_eights(3)
0
>>> num_eights(8)
1
>>> num_eights(88888888)
8
>>> num_eights(2638)
1
>>> num_eights(86380)
2
>>> num_eights(12345)
0
>>> num_eights(8782089)
3
>>> from construct_check import check
>>> # ban all assignment statements
>>> check(HW_SOURCE_FILE, 'num_eights',
...       ['Assign', 'AnnAssign', 'AugAssign', 'NamedExpr', 'For', 'While'])
True
"""
"*** YOUR CODE HERE ***"

if n % 10 == 8:
    return 1 + num_eights(n // 10)
elif n < 10:
    return 0
else:
    return num_eights(n // 10)
```


# 03
- `interleaved sum`, 计算两个函数相互调换的和
- 限制不能取模
- 可以使用相互递归来将奇偶信息传递出去
- 也可以直接使用一个递归, 在一次递归调用中同时处理奇偶
```python
def interleaved_sum(n, odd_func, even_func):
"""Compute the sum odd_func(1) + even_func(2) + odd_func(3) + ..., up
to n.

>>> identity = lambda x: x
>>> square = lambda x: x * x
>>> triple = lambda x: x * 3
>>> interleaved_sum(5, identity, square) # 1   + 2*2 + 3   + 4*4 + 5
29
>>> interleaved_sum(5, square, identity) # 1*1 + 2   + 3*3 + 4   + 5*5
41
>>> interleaved_sum(4, triple, square)   # 1*3 + 2*2 + 3*3 + 4*4
32
>>> interleaved_sum(4, square, triple)   # 1*1 + 2*3 + 3*3 + 4*3
28
>>> from construct_check import check
>>> check(HW_SOURCE_FILE, 'interleaved_sum', ['While', 'For', 'Mod']) # ban loops and %
True
"""
"*** YOUR CODE HERE ***"

def sum_from(k):
    if k > n:
        return 0
    elif k == n:
        return odd_func(k)
    else:
        return odd_func(k) + even_func(k + 1) + sum_from(k + 2)

return sum_from(1)

# def odd(m):
#     if m == n:
#         return odd_func(n)
#     return odd_func(m) + even(m + 1)

# def even(m):
#     if m == n:
#         return even_func(n)
#     return even_func(m) + odd(m + 1)

# return odd(1)
```


# 04
- `count_coins(n)`, 使用 `1,5,10,25` 找零 `n` 的方式个数
- 直接使用生活中找零的思路, 从大到小找 (从小到大找也是可以的)
- 本质上是一个深度优先爆搜
- 情况可以分解成两种
	- 当前 `n` 容得下当前最大面额 `k`, `count_coins(n-k, k)`
	- 当前 `n` 容不下当前最大面额 `k`, `count_coins(n, next_smaller_coin(k))`
	- 两种情况构成了完整的搜索空间
- 基本情况
	- `n == 0`: 刚好找完了, 方式数 +1
	- `n < 0`: 在目前的已使用硬币中找不了, 方式 +0
	- `k == None`: `n` 还没找完, 但是已经没有硬币了, 方式 +0

```python
def count_coins(total):
"""Return the number of ways to make change using coins of value of 1, 5, 10, 25.
>>> count_coins(15)
6
>>> count_coins(10)
4
>>> count_coins(20)
9
>>> count_coins(100) # How many ways to make change for a dollar?
242
>>> count_coins(200)
1463
>>> from construct_check import check
>>> # ban iteration
>>> check(HW_SOURCE_FILE, 'count_coins', ['While', 'For'])
True
"""
"*** YOUR CODE HERE ***"

def change_up_to_n(curr_total, n):
    if curr_total == 0:
        return 1
    if curr_total < 0:
        return 0
    if n == None:
        return 0

    return change_up_to_n(curr_total - n, n) + change_up_to_n(
        curr_total, next_smaller_coin(n)
    )

return change_up_to_n(total, 25)
```


# Extra
- 看看就好, 看也看不懂, 递归和函数式编程难度 level up

```python
def move_stack(n, start, end):
    """Print the moves required to move n disks on the start pole to the end
    pole without violating the rules of Towers of Hanoi.

    n -- number of disks
    start -- a pole position, either 1, 2, or 3
    end -- a pole position, either 1, 2, or 3

    There are exactly three poles, and start and end must be different. Assume
    that the start pole has at least n disks of increasing size, and the end
    pole is either empty or has a top disk larger than the top n start disks.

    >>> move_stack(1, 1, 3)
    Move the top disk from rod 1 to rod 3
    >>> move_stack(2, 1, 3)
    Move the top disk from rod 1 to rod 2
    Move the top disk from rod 1 to rod 3
    Move the top disk from rod 2 to rod 3
    >>> move_stack(3, 1, 3)
    Move the top disk from rod 1 to rod 3
    Move the top disk from rod 1 to rod 2
    Move the top disk from rod 3 to rod 2
    Move the top disk from rod 1 to rod 3
    Move the top disk from rod 2 to rod 1
    Move the top disk from rod 2 to rod 3
    Move the top disk from rod 1 to rod 3
    """
    assert 1 <= start <= 3 and 1 <= end <= 3 and start != end, "Bad start/end"
    "*** YOUR CODE HERE ***"
    if n == 1:
        print_move(start, end)
    else:
        other = 6 - start - end
        move_stack(n - 1, start, other)
        print_move(start, end)
        move_stack(n - 1, other, end)


from operator import sub, mul


# TODO:
def make_anonymous_factorial():
    """Return the value of an expression that computes factorial.

    >>> make_anonymous_factorial()(5)
    120
    >>> from construct_check import check
    >>> # ban any assignments or recursion
    >>> check(HW_SOURCE_FILE, 'make_anonymous_factorial',
    ...     ['Assign', 'AnnAssign', 'AugAssign', 'NamedExpr', 'FunctionDef', 'Recursion'])
    True
    """
    # damn!
    # return (lambda f: f(f))(lambda f: lambda x: 1 if x == 0 else x * f(f)(x - 1))
    return (lambda f: lambda k: f(f, k))(
        lambda f, k: k if k == 1 else mul(k, f(f, sub(k, 1)))
    )
```