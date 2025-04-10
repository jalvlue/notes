- 关于数据处理的一切

# 4.1 introduction
- 数据很多很大, 对数据的处理分析可以以一种前所未有的方式帮助认识人类和世界的种种
- 现代数据处理通常非常复杂, 处理方式通常是搭建一套流水线, 将数据看成是一个巨大流, 通过流水线的各个组件对流经期间的数据进行处理
- `ch4` 会学习一些有关数据处理的套件


# 4.2 implicit sequences
- 隐式序列, `lazy computation`
- 实际上就是一种逻辑序列, 序列中的值不需要提前计算并在内存中保存, 而是可以按需计算, 需要的使用的时候就计算出来
- `range` 对象就是一个典型的隐式序列, 根据计算公式按需求取
```python
>>> r = range(10000, 1000000000)
>>> r[45006230]
45016230
```


### 4.2.1 iterators
- 对于所有的容器 `container`, 都有配套的迭代器类型, 通过 `iter()` 内置函数可以创建该容器的迭代器对象, 并利用该对象在容器上进行迭代, 不同迭代器对象之间是相互独立的
- 迭代器提供两个功能
	- 获取下一个元素
	- 判断是否还有元素
- 迭代器的好处就是为显式序列 `containers` 提供了一种隐式序列计算方式, 虽然没有显式序列的随机存取能力, 但是在顺序操作时有优势
- 对于大型流式数据的处理和分析软件来说, 迭代器的顺序操作有很明显的作用
- `python` 内置的 `next()` 函数就是用于获取迭代器下一个元素的
```python
>>> primes = [2, 3, 5, 7]
>>> type(primes)
>>> iterator = iter(primes)
>>> type(iterator)
>>> next(iterator)
2
>>> next(iterator)
3
>>> next(iterator)
5
```


### 4.2.2 iterables
- 任何可以通过 `iter()` 函数获得一个迭代器的类型, 都称为是一个 `iterable`, 可迭代的
```python
>>> d = {'one': 1, 'two': 2, 'three': 3}
>>> d
{'one': 1, 'three': 3, 'two': 2}
>>> k = iter(d)
>>> next(k)
'one'
>>> next(k)
'three'
>>> v = iter(d.values())
>>> next(v)
1
>>> next(v)
3
```
- 生成了迭代器后, 再去修改原容器的内容, 再次迭代该迭代器时, 会导致迭代器报错
- 也就是说, 迭代器实际上是数据的一个视图, 并且在使用的过程中, 不允许底层的数据发生改变
```python
>>> d.pop('two')
2
>>> next(k)
       
RuntimeError: dictionary changed size during iteration
Traceback (most recent call last):
```


### 4.2.3 built-in iterators
- `python` 中内置了许多对序列做一些处理, 然后获取一个处理后的迭代器的高阶函数
- `map(fn, iterable)`, 返回一个迭代器, 在迭代原有容器的基础上为其中的元素加上映射
```python
>>> def double_and_print(x):
        print('***', x, '=>', 2*x, '***')
        return 2*x
>>> s = range(3, 7)
>>> doubled = map(double_and_print, s)  # double_and_print not yet called
>>> next(doubled)                       # double_and_print called once
*** 3 => 6 ***
6
>>> next(doubled)                       # double_and_print called again
*** 4 => 8 ***
8
>>> list(doubled)                       # double_and_print called twice more
*** 5 => 10 ***
*** 6 => 12 ***
[10, 12]
```
- `filter()`
- `zip()`
- `reversed()`
- ...


### 4.2.4 for statements
- `python for ... in ...` 是在 `iterable` 上进行迭代的语法糖, 在内部, 实际上会通过该 `iterable` 获得一个 `iterator`, 然后在该 `iterator` 上做迭代
- 在实践上, 通常推荐让 `iterator` 也实现 `iterable` 接口, 直接返回它自己, 这样就可以直接向 `for` 传递一个 `iterable` 或 `iterator`
```ad-tip
The for statement in Python operates on iterators. Objects are iterable (an interface) if they have an __iter__ method that returns an iterator. Iterable objects can be the value of the <expression> in the header of a for statement:

for <name> in <expression>:
    <suite>
To execute a for statement, Python evaluates the header <expression>, which must yield an iterable value. Then, the __iter__ method is invoked on that value. Until a StopIteration exception is raised, Python repeatedly invokes the __next__ method on that iterator and binds the result to the <name> in the for statement. Then, it executes the <suite>.
```

```python
>>> counts = [1, 2, 3]
>>> for item in counts:
        print(item)
1
2
3

>>> items = counts.__iter__()
>>> try:
        while True:
            item = items.__next__()
            print(item)
    except StopIteration:
        pass
1
2
3
```


### 4.2.5 generators and yield statements
- 对于简单的序列数据, 使用索引之类的东西记住上次遍历到的位置, 从而来制作一个 `iterator` 是可行的, 但是对于复杂的数据结构处理起来就有点复杂, 因此引入了 `generator` 和 `yield statement`
- `generator` 本质上也是一个 `iterator`, 可以直接拿来迭代
- `generator` 由 `generator function` 隐式返回
- `generator function` 就是使用 `yield` 而不是 `return` 的函数
- 在 `generator function` 中, `python interpreter` 会记住上次 `yield` 执行的位置, 每次调用 `generator.__next__()`, 都会从生成它的 `generator function` 中的上一个 `yield` 位置继续执行下去, 相当于解释器为我们做了记忆索引这一步, 我们只需要关注遍历逻辑, 然后使用 `yield` 来生成下一个元素就好
```python
>>> def letters_generator():
        current = 'a'
        while current <= 'd':
            yield current
            current = chr(ord(current)+1)
>>> for letter in letters_generator():
        print(letter)
a
b
c
d

>>> letters = letters_generator()
>>> type(letters)
<class 'generator'>
>>> letters.__next__()
'a'
>>> letters.__next__()
'b'
>>> letters.__next__()
'c'
>>> letters.__next__()
'd'
>>> letters.__next__()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```


### 4.2.6 iterable interface
- 上面提到的 `iterable` 是一个接口, 实现 `__iter__()` 函数返回一个 `iterator` 即可实现这个接口, 然后可以使用 `for ... in ... ` 语法糖
```python
>>> class Letters:
def __init__(self, start='a', end='e'):
    self.start = start
    self.end = end
def __iter__(self):
    return LetterIter(self.start, self.end)

>>> b_to_k = Letters('b', 'k')
>>> first_iterator = b_to_k.__iter__()
>>> next(first_iterator)
'b'
>>> next(first_iterator)
'c'
>>> second_iterator = iter(b_to_k)
>>> second_iterator.__next__()
'b'
>>> first_iterator.__next__()
'd'
>>> first_iterator.__next__()
'e'
>>> second_iterator.__next__()
'c'
>>> second_iterator.__next__()
'd'

>>> caps = map(lambda x: x.upper(), b_to_k)
>>> next(caps)
'B'
>>> next(caps)
'C'
```


### 4.2.7 creating iterables with yield
- 使用 `yield` 的 `generator function` 来作为一种类型的 `__iter__()` 函数, 从而实现 `iterable` 接口
```python
>>> class LettersWithYield:
        def __init__(self, start='a', end='e'):
            self.start = start
            self.end = end
        def __iter__(self):
            next_letter = self.start
            while next_letter < self.end:
                yield next_letter
                next_letter = chr(ord(next_letter)+1)

>>> letters = LettersWithYield()
>>> list(all_pairs(letters))[:5]
[('a', 'a'), ('a', 'b'), ('a', 'c'), ('a', 'd'), ('b', 'a')]
```


### 4.2.8 iterator interface
- 上面提到的 `iterator` 也是一个接口, 需要具体的实现类实现 `__next__()` 方法
```python
>>> class LetterIter:
        """An iterator over letters of the alphabet in ASCII order."""
        def __init__(self, start='a', end='e'):
            self.next_letter = start
            self.end = end
        def __next__(self):
            if self.next_letter == self.end:
                raise StopIteration
            letter = self.next_letter
            self.next_letter = chr(ord(letter)+1)
            return letter

>>> letter_iter = LetterIter()
>>> letter_iter.__next__()
'a'
>>> letter_iter.__next__()
'b'
>>> next(letter_iter)
'c'
>>> letter_iter.__next__()
'd'
>>> letter_iter.__next__()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 12, in next
StopIteration
```


### 4.2.9 streams
TODO


### 4.2.10 python streams
TODO
