# 2.1 introduction
- 第一章使用函数, 原始数据类型, 原始运算符, 高阶函数来构建强大的抽象
- 第二章专注于使用多种的数据类型来构建抽象

### 2.1.1 native data types
- `python` 是强面向对象的, 所有的值(字面量)其实都是一个对象, 属于某一个类
	- `int`
	- `float`
	- `complex`
	- ...
- 但是 `python` 的问题是他是弱类型的, 任何不当的处理都可能会改变一个变量的类型, 特别是对一个 `int` 做除法, 很可能就把他除成一个 `float` 了


# 2.2 data abstraction
- 数据抽象的典型是将多个独立的数据组合起来, 形成一个整体, 从而使得这个整体可以刻画某些复杂的现实世界事物, 在整体的同时, 这些数据还可以被独立地处理
- 在使用时应该关注数据抽象中的假设, 而具体的实现是完全独立的 (接口实现分离, 使用方只关注接口提供的功能, 而不去管具体的实现)


### 2.2.1 example: rational numbers
- 分数形式表示有理数
	- 可以直接使用两个变量来表示
	- 也可以使用一个列表
	- 也可以使用一个类
	- 区别只是数据的相关性以及对外表现的能力
```python
def rational(n, d):
    pass


def numer(x):
    pass


def denom(x):
    pass


def add_rationals(x, y):
    nx, dx = numer(x), denom(x)
    ny, dy = numer(y), denom(y)
    return rational(nx * dy + ny * dx, dx * dy)


def mul_rationals(x, y):
    return rational(numer(x) * numer(y), denom(x) * denom(y))


def print_rational(x):
    print(numer(x), "/", denom(x))


def rationals_are_equal(x, y):
    return numer(x) * denom(y) == numer(y) * denom(x)

```
- 在确定了数据的组织方式, 以及如何获取对应的子数据之后 (`rational(), numer(), denom()`), 就可以直接实现与之相关的一系列操作, 无需考虑具体的数据结构实现细节

### 2.2.2 pairs
- `python list` 是一个内置的类型, 就类似是数组之类的东西, 支持下标随机存取
- 因此可以使用 `list` 将有理数的两部分数据组织起来, 作为一种具体实现
- 而其他的函数例如 `print_rational()` 不需要改动, 因为它依赖的是一种抽象的生命, 任何实现了该声明的具体数据结构都可以完美地像一个积木模块随时插入抽出
```python
def rational(n, d):
    return [n, d]


def numer(x):
    return x[0]


def denom(x):
    return x[1]

```

### 2.2.3 abstraction barriers
- 数据抽象的底层思想是将数据结构, 以及直接操作数据的函数作为一组实现, 后续的其他功能都依赖与最底层的直接操作数据函数, 从而构建了抽象的边界
- ![[Pasted image 20241008162959.png]]
- 违反抽象边界的例子
```python
def square_rational(x):
	return mul_rational(x, x)

def square_rational_violating_once(x):
	return rational(numer(x) * numer(x), denom(x) * denom(x))

def square_rational_violating_twice(x):
	return [x[0] * x[0], x[1] * x[1]]
```
- 违反抽象边界使得代码难以维护


### 2.2.4 the properties of data
```python
def pair(x, y):
    def get(index):
        if index == 0:
            return x
        elif index == 1:
            return y

    return get


def select(p, i):
    return p(i)

```
- 一种函数式的实现, 虽然跟直接使用 `list` 有点区别, 但是都实现了数据抽象的属性


# 2.3 sequences
- 顺序表, 最重要的数据结构之一, 有点类似数据结构中的定义, 是一种 `adt`, 有很多的具体实现
	- `length`
	- `element selection`
- `list` 是 `python` 中, 顺序表最重要的一种实现


### 2.3.1 lists
- `python` 内置的列表, 类似数组, 顺序表的实现之一
- 列表常见操作无需多言
```python
>>> digits = [1, 8, 2, 8]
>>> len(digits)
4
>>> digits[3]
8
>>> [2, 7] + digits * 2
[2, 7, 1, 8, 2, 8, 1, 8, 2, 8]
>>> pairs = [[10, 20], [30, 40]]
>>> pairs[1]
[30, 40]
>>> pairs[1][0]
30
```

### 2.3.2 sequence iteration
- 序列迭代 `for ... in ...`
```python
>>> def count(s, value):
        """Count the number of occurrences of value in sequence s."""
        total = 0
        for elem in s:
            if elem == value:
                total = total + 1
        return total

>>> count(digits, 8)
2
```
- 范围迭代 `range()`, 返回一个 `range` 对象
```python
>>> range(1, 10)  # Includes 1, but not 10
range(1, 10)
>>> list(range(5, 8))
[5, 6, 7]
>>> list(range(4))
[0, 1, 2, 3]

>>> for _ in range(3):
        print('Go Bears!')

Go Bears!
Go Bears!
Go Bears!
```

### 2.3.3 sequence processing
- 其实这里的内容跟 sicp 的联系不是很大, 主要是介绍 python 语法, 简略看一眼
- `list comprehensions`, 就是在对原列表的所有元素做一些处理后, 生成一个新列表
```python
>>> odds = [1, 3, 5, 7, 9]
>>> [x+1 for x in odds]
[2, 4, 6, 8, 10]

>>> [x for x in odds if 25 % x == 0]
[1, 5]
```
- `aggregation`, 就是对原列表的元素做一些筛选后生成新列表
```python
>>> def divisors(n):
        return [1] + [x for x in range(2, n) if n % x == 0]

>>> divisors(4)
[1, 2]
>>> divisors(12)
[1, 2, 3, 4, 6]

>>> [n for n in range(1, 1000) if sum(divisors(n)) == n]
[6, 28, 496]
```

### 2.3.4 sequence abstraction
- `membership`
```python
>>> digits
[1, 8, 2, 8]
>>> 2 in digits
True
>>> 1828 not in digits
True
```
- `slicing`
```python
>>> digits[0:2]
[1, 8]
>>> digits[1:]
[8, 2, 8]
```

### 2.3.5 strings
- 字符串是最基础的数据类型, 同时也是顺序表的一种实现
- `python` 没有单个字符类型的概念, 全都是字符串
```python
>>> 'I am string!'
'I am string!'
>>> "I've got an apostrophe"
"I've got an apostrophe"
>>> '您好'
'您好'

>>> city = 'Berkeley'
>>> len(city)
8
>>> city[3]
'k'

>>> 'Berkeley' + ', CA'
'Berkeley, CA'
>>> 'Shabu ' * 2
'Shabu Shabu '

>>> 'here' in "Where's Waldo?"
True

>>> """The Zen of Python
claims, Readability counts.
Read more: import this."""
'The Zen of Python\nclaims, "Readability counts."\nRead more: import this.'

>>> str(2) + ' is an element of ' + str(digits)
'2 is an element of [1, 8, 2, 8]'
```


### 2.3.6 tree
- 树形结构的数据
- 使用 `list` 模拟的例子
```python
>>> def tree(root_label, branches=[]):
        for branch in branches:
            assert is_tree(branch), '分支必须是树'
        return [root_label] + list(branches)

>>> def label(tree):
        return tree[0]

>>> def branches(tree):
        return tree[1:]

>>> def is_tree(tree):
		if type(tree) != list or len(tree) < 1:
		    return False
		for branch in branches(tree):
		    if not is_tree(branch):
		        return False
		return True

>>> def fib_tree(n):
        if n == 0 or n == 1:
            return tree(n)
        else:
            left, right = fib_tree(n-2), fib_tree(n-1)
            fib_n = label(left) + label(right)
            return tree(fib_n, [left, right])
>>> fib_tree(5)
[5, [2, [1], [1, [0], [1]]], [3, [1, [0], [1]], [2, [1], [1, [0], [1]]]]]

```


### 2.3.7 linked lists
- 除了使用列表构造模拟 `tree`, 还可以模拟链表

```python
four = [1, [2, [3, [4, 'empty']]]]
```
![[Pasted image 20241102135513.png]]

- 链表是天然递归的结构
```python
>>> empty = 'empty'
>>> def is_link(s):
        """s is a linked list if it is empty or a (first, rest) pair."""
        return s == empty or (len(s) == 2 and is_link(s[1]))
>>> def link(first, rest):
        """Construct a linked list from its first element and the rest."""
        assert is_link(rest), "rest must be a linked list."
        return [first, rest]
>>> def first(s):
        """Return the first element of a linked list s."""
        assert is_link(s), "first only applies to linked lists."
        assert s != empty, "empty linked list has no first element."
        return s[0]
>>> def rest(s):
        """Return the rest of the elements of a linked list s."""
        assert is_link(s), "rest only applies to linked lists."
        assert s != empty, "empty linked list has no rest."
        return s[1]
>>> def len_link(s):
        """Return the length of linked list s."""
        length = 0
        while s != empty:
            s, length = rest(s), length + 1
        return length
>>> def getitem_link(s, i):
        """Return the element at index i of linked list s."""
        while i > 0:
            s, i = rest(s), i - 1
        return first(s)
>>> def len_link_recursive(s):
        """Return the length of a linked list s."""
        if s == empty:
            return 0
        return 1 + len_link_recursive(rest(s))

>>> def getitem_link_recursive(s, i):
        """Return the element at index i of linked list s."""
        if i == 0:
            return first(s)
        return getitem_link_recursive(rest(s), i - 1)
```


# 2.4 Mutable Data
- 控制复杂度是一大难题, 函数抽象和简单的数据抽象带来的模块化提供了部分解决方案
- 对于一些会随着时间而改变状态的数据(可变状态数据), 将他们组织起来为数据添加状态, 通过 `OOP` 的方式控制


### 2.4.1 the object metaphor
- `object`, 对象, `OOP` 中最重要的东西, 将数据和函数结合起来
	- 具有数据的表达能力
	- 还有函数的处理过程能力
	- 一个对象同时是信息和过程
- `attributes`, 属性, 就是对象中, 表达信息的那一部分
- `methods`, 方法, 其实算是函数类型的属性, 表达处理过程那一部分

```python
>>> tues = date(2014, 5, 13)
>>> tues.year
2014
>>> tues.strftime('%A, %B %d')
'Tuesday, May 13'

>>> '1234'.isnumeric()
True
>>> 'rOBERT dE nIRO'.swapcase()
'Robert De Niro'
>>> 'eyes'.upper().endswith('YES')
True
```


### 2.4.2 sequence objects
- `mutable data`, 可变的数据, 这种用对象来表示最合适不过了, 其行为可能会随着数据的变化而变化, 例如绝大多数的对象
- `immutable data`, 不可变数据, 例如一些原子类型的字面量
- `python list` 是一种可变数据, 也是一个内置的类
- 一个可变表达的例子
```python
>>> chinese = ['coin', 'string', 'myriad']  # A list literal
>>> suits = chinese                         # Two names refer to the same list
>>> suits.pop()             # Remove and return the final element
'myriad'
>>> suits.remove('string')  # Remove the first element that equals the argument
>>> suits.append('cup')              # Add an element to the end
>>> suits.extend(['sword', 'club'])  # Add all elements of a sequence to the end
>>> suits[2] = 'spade'  # Replace an element
>>> suits
['coin', 'cup', 'spade', 'club']
>>> suits[0:2] = ['heart', 'diamond']  # Replace a slice
>>> suits
['heart', 'diamond', 'spade', 'club']
>>> chinese  # This name co-refers with "suits" to the same changing list
['heart', 'diamond', 'spade', 'club']
```
- 使用 `list()` 复制一份新的列表
```python
suits = ['heart', 'diamond', 'spade', 'club']
nest = list(suits)
nest[0] = suits
suits.insert(2, 'Joker')
joke = nest[0].pop(2)

>>> suits is nest[0]
True
>>> suits is ['heart', 'diamond', 'spade', 'club']
False
>>> suits == ['heart', 'diamond', 'spade', 'club']
True
```
- `python tuples`, 是一种不可变的内置类型, 跟 `list` 非常相似, 只是不可变
```python
>>> 1, 2 + 3
(1, 5)
>>> ("the", 1, ("and", "only"))
('the', 1, ('and', 'only'))
>>> type( (10, 20) )
<class 'tuple'>
>>> code = ("up", "up", "down", "down") + ("left", "right") * 2
>>> len(code)
8
>>> code[3]
'down'
>>> code.count("down")
2
>>> code.index("left")
4
```


### 2.4.3 dictionary
- `python` 内置的哈希表, 写起来就像 `json` 一样, 没有声明定义, 直接键值对写起来
```python
>>> numerals = {'I': 1.0, 'V': 5, 'X': 10}
>>> numerals['X']
10
>>> numerals['I'] = 1
>>> numerals['L'] = 50
>>> numerals
{'I': 1, 'X': 10, 'L': 50, 'V': 5}
```
- `kv tuple pair list` 可以直接使用构造函数 `dict()` 转为一个哈希表, 这就是 py
```python
>>> dict([(3, 9), (4, 16), (5, 25)])
{3: 9, 4: 16, 5: 25}
```


### 2.4.4 local state
- 函数其实也是可以有状态, 通过闭包就可实现
- `python` 中不能直接使用闭包函数体外的变量(父函数的变量), 需要使用 `nonlocal` 关键字声明一下
```python
>>> def make_withdraw(balance):
        """Return a withdraw function that draws down balance with each call."""
        def withdraw(amount):
            nonlocal balance                 # Declare the name "balance" nonlocal
            if amount > balance:
                return 'Insufficient funds'
            balance = balance - amount       # Re-bind the existing balance name
            return balance
        return withdraw

>>> withdraw = make_withdraw(100)
>>> withdraw(25)
75
>>> withdraw(25)
50
>>> withdraw(60)
'Insufficient funds'
>>> withdraw(15)
35

```
- 通过这种闭包外维护的变量, 函数也可以有自己的状态


### 2.4.6 the cost of non-local assignment
- 使用 `non-local` 闭包函数来维护状态的缺陷是, 多个 `name` 可能底层是同一个函数, 从而导致状态意外丢失


### 2.4.7 implementing lists and dictionaries
- 使用带状态的函数来写一个链表
- 这里链表的状态完全保存在父函数的 `contents` 变量里, 闭包子函数通过 `nonlocal` 来读写这个状态
```python
def mutable_link():
    contents = "empty"

    def dispatch(message, value=None):
        nonlocal contents
        if message == "len":
            return len_link(contents)
        elif message == "getitem":
            return getitem_link(contents, value)
        elif message == "push_first":
            contents = link(value, contents)
        elif message == "pop_first":
            f = first(contents)
            contents = rest(contents)
            return f
        elif message == "str":
            return join_link(contents, ", ")

    return dispatch


def to_mutable_link(source):
    s = mutable_link()
    for element in reversed(source):
        s("push_first", element)
    return s

```

- 写一个字典, 状态完全由函数保持
```python
>>> def dictionary():
"""Return a functional implementation of a dictionary."""
records = []
def getitem(key):
    matches = [r for r in records if r[0] == key]
    if len(matches) == 1:
        key, value = matches[0]
        return value
def setitem(key, value):
    nonlocal records
    non_matches = [r for r in records if r[0] != key]
    records = non_matches + [[key, value]]
def dispatch(message, key=None, value=None):
    if message == 'getitem':
        return getitem(key)
    elif message == 'setitem':
        setitem(key, value)
return dispatch
```


# 2.5 Object-Oriented Programming
- `OOP` 将行为, 数据完全与对象绑定起来, 整个程序的运行被抽象成很多维护自己独立信息的对象之间的相互交互


### 2.5.1 Objects and Classes
- 对象和类
```python
class Account:
	def __init__(self, holder, ...)

a = Account("Kirk")
a.holder
a.balance
a.deposit(15)
a.withdraw(10)
a.balance
```


### 2.5.2 Defining Classes
```python
>>> class Account:
	def __init__(self, account_holder):
	    self.balance = 0
	    self.holder = account_holder
	def deposit(self, amount):
	    self.balance = self.balance + amount
	    return self.balance
	def withdraw(self, amount):
	    if amount > self.balance:
	        return 'Insufficient funds'
	    self.balance = self.balance - amount
	    return self.balance

>>> spock_account = Account('Spock')
>>> spock_account.deposit(100)
100
>>> spock_account.withdraw(90)
10
>>> spock_account.withdraw(90)
'Insufficient funds'
>>> spock_account.holder
'Spock'
```



### 2.5.3 Message Passing and Dot Expressions
- 内置的 `getattr(object, attr_name), hasattr(object, attr_name)`
- 一个方法可以直接通过类+对象访问, 也可以直接通过对象的 `dot expression` 调用
- 类的命名通常是大驼峰 `CapWords`, 对象的命名则遵循变量名规则, 通常是小写单词使用下划线分隔开
```python
>>> getattr(spock_account, 'balance')
10
>>> hasattr(spock_account, 'deposit')
True

>>> type(Account.deposit)
<class 'function'>
>>> type(spock_account.deposit)
<class 'method'>

>>> Account.deposit(spock_account, 1001)  # The deposit function takes 2 arguments
1011
>>> spock_account.deposit(1000)           # The deposit method takes 1 argument
2011
```


### 2.5.3 Class Attributes
- `class attributes`, 与类有关, 与对象没有强关联的属性 (其实就类似静态变量)
- 对于自己修改了静态变量的对象, 此后这个静态变量跟原始类中的那个没有关系
- 对于没有修改静态变量的对象, 每次读取都来自类中的那个, 如果类直接修改了, 那么他读取到的也会修改
```python
>>> class Account:
        interest = 0.02            # A class attribute
        def __init__(self, account_holder):
            self.balance = 0
            self.holder = account_holder
        # Additional methods would be defined here

>>> spock_account = Account('Spock')
>>> kirk_account = Account('Kirk')
>>> spock_account.interest
0.02
>>> kirk_account.interest
0.02

>>> Account.interest = 0.04
>>> spock_account.interest
0.04
>>> kirk_account.interest
0.04

>>> kirk_account.interest = 0.08
>>> kirk_account.interest
0.08
>>> spock_account.interest
0.04
>>> Account.interest = 0.05  # changing the class attribute
>>> spock_account.interest     # changes instances without like-named instance attributes
0.05
>>> kirk_account.interest     # but the existing instance attribute is unaffected
0.08
```


### 2.5.5 Inheritance
- 继承, 获取父类的属性和方法
```python
>>> ch = CheckingAccount('Spock')
>>> ch.interest     # Lower interest rate for checking accounts
0.01
>>> ch.deposit(20)  # Deposits are the same
20
>>> ch.withdraw(5)  # withdrawals decrease balance by an extra charge
14
```


### 2.5.6 Using Inheritance
- 继承的语法
```python
>>> class Account:
        """A bank account that has a non-negative balance."""
        interest = 0.02
        def __init__(self, account_holder):
            self.balance = 0
            self.holder = account_holder
        def deposit(self, amount):
            """Increase the account balance by amount and return the new balance."""
            self.balance = self.balance + amount
            return self.balance
        def withdraw(self, amount):
            """Decrease the account balance by amount and return the new balance."""
            if amount > self.balance:
                return 'Insufficient funds'
            self.balance = self.balance - amount
            return self.balance

>>> class CheckingAccount(Account):
        """A bank account that charges for withdrawals."""
        withdraw_charge = 1
        interest = 0.01
        def withdraw(self, amount):
            return Account.withdraw(self, amount + self.withdraw_charge)
```


### 2.5.7 Multiple Inheritance
- 多继承, 方法和属性来自多个父类
![[Pasted image 20241117152330.png]]
```python
>>> class SavingsAccount(Account):
        deposit_charge = 2
        def deposit(self, amount):
            return Account.deposit(self, amount - self.deposit_charge)

>>> class AsSeenOnTVAccount(CheckingAccount, SavingsAccount):
        def __init__(self, account_holder):
            self.holder = account_holder
            self.balance = 1           # A free dollar!

>>> such_a_deal = AsSeenOnTVAccount("John")
>>> such_a_deal.balance
1
>>> such_a_deal.deposit(20)            # $2 fee from SavingsAccount.deposit
19
>>> such_a_deal.withdraw(5)            # $1 fee from CheckingAccount.withdraw
13

>>> such_a_deal.deposit_charge
2
>>> such_a_deal.withdraw_charge
1
```


### 2.5.8 The Role of Objects
- `OOP` 实际上是对现实世界的建模, 对象消息交互是一种
