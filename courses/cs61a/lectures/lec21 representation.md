# Representation
- 一个对象向外界说明他是什么对象是很重要的
- 一般都是通过字符串传递信息
- `str`, 人类可读的字符串, 例如 `<1, 2>`
- `repr`, `python interpreter` 可识别的字符串, 也就是一个 `python expression`, 可以直接拿来放到程序中, 例如 `Link(1, Link(2))`

```python
>>> from fractions import Fraction
>>> half = Fraction(1, 2)
>>> half
Fraction(1, 2)
>>> repr(half)
'Fraction(1, 2)'
>>> print(half)
'1/2'
>>> str(half)
'1/2'
>>> eval(repr(half))
Fraction(1, 2)
```


# Ploymorphic functions
- 上述的 `str(), repr()` 函数都是多态函数, 实际上他们转发了类内部的 `__str__(), __repr()__`


# Special method names
- `__init__, __repr__, __bool__, __float__, ...` 这些都是特殊方法, 是实现了 `python` 内建的接口的方法, 用于跟 `python` 内建系统交互
