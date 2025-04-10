# 01
- `divide`, 返回能整除的列表
- `python` 语法糖太多了受不了
```python
def divide(quotients, divisors):
	"""Return a dictonary in which each quotient q is a key for the list of
	divisors that it divides evenly.
	
>>> 	divide([3, 4, 5], [8, 9, 10, 11, 12])
	{3: [9, 12], 4: [8, 12], 5: [10]}
>>> 	divide(range(1, 5), range(20, 25))
	{1: [20, 21, 22, 23, 24], 2: [20, 22, 24], 3: [21, 24], 4: [20, 24]}
	"""
	return {q: [d for d in divisors if d%q==0] for q in quotients}

```


# 02
- `buy`, 购买需要的水果的方法
- 爆搜
```python
def buy(required_fruits, prices, total_amount):
    """Print ways to buy some of each fruit so that the sum of prices is amount.

    >>> prices = {'oranges': 4, 'apples': 3, 'bananas': 2, 'kiwis': 9}
    >>> buy(['apples', 'oranges', 'bananas'], prices, 12)
    [2 apples][1 orange][1 banana]
    >>> buy(['apples', 'oranges', 'bananas'], prices, 16)
    [2 apples][1 orange][3 bananas]
    [2 apples][2 oranges][1 banana]
    >>> buy(['apples', 'kiwis'], prices, 36)
    [3 apples][3 kiwis]
    [6 apples][2 kiwis]
    [9 apples][1 kiwi]
    """
    def add(fruits, amount, cart):
        if fruits == [] and amount == 0:
            print(cart)
        elif fruits and amount > 0:
            fruit = fruits[0]
            price = prices[fruit]
            for k in range(1, amount//price+1):
                add(fruits[1:], amount-price*k, cart+display(fruit,k))

    add(required_fruits, total_amount, '')


def display(fruit, count):
    """Display a count of a fruit in square brackets.

    >>> display('apples', 3)
    '[3 apples]'
    >>> display('apples', 1)
    '[1 apple]'
    """
    assert count >= 1 and fruit[-1] == 's'
    if count == 1:
        fruit = fruit[:-1]  # get rid of the plural s
    return '[' + str(count) + ' ' + fruit + ']'


```