- 有一个求 `lcm` 的小问题, 回顾一下 `lcm, gcd`
```python
def multiple(a, b):
	"""Return the smallest number n that is a multiple of both a and b.
	
>>> 	multiple(3, 4)
	12
>>> 	multiple(14, 21)
	42
	"""
	"*** YOUR CODE HERE ***"
	
	def gcd(a, b):
	    while a % b != 0:
	        a, b = b, a % b
	    return b
	
	return a * b // gcd(a, b)

```

- 对一个值循环应用到函数上
```python
def cycle(f1, f2, f3):
"""Returns a function that is itself a higher-order function.

>>> 	def add1(x):
	...     return x + 1
>>> 	def times2(x):
	...     return x * 2
>>> 	def add3(x):
	...     return x + 3
>>> 	my_cycle = cycle(add1, times2, add3)
>>> 	identity = my_cycle(0)
>>> 	identity(5)
	5
>>> 	add_one_then_double = my_cycle(2)
>>> 	add_one_then_double(1)
	4
>>> 	do_all_functions = my_cycle(3)
>>> 	do_all_functions(2)
	9
>>> 	do_more_than_a_cycle = my_cycle(4)
>>> 	do_more_than_a_cycle(2)
	10
>>> 	do_two_cycles = my_cycle(6)
>>> 	do_two_cycles(1)
	19
	"""
	"*** YOUR CODE HERE ***"
	
	loc2func = [f1, f2, f3]
	
	def g(n):
	    def h(x):
	        ret = x
	        i = 0
	        while i < n:
	            ret = loc2func[i % 3](ret)
	            i += 1
	        return ret
	
	    return h
	
	return g

```
