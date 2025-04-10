# 3.1 introduction
- 函数抽象
- 数据抽象 (类和对象系统)
- 本章注重程序本身, 程序本质上只是一些文本, 让程序有意义的根本就是有一个解释/编译器, 来为这个程序文本做解释, 而解释器本身又是一个程序


### 3.1.1 programming languages
- 设计 `scheme` 解释器, 了解 `scheme` 程序执行的时候, 计算是如何被表示的
- 两个相互递归函数 `These functions are recursive in that they are defined in terms of each other: applying a function requires evaluating the expressions in its body, while evaluating an expression may involve applying one or more functions.`

# 3.2 functional programming
- `scheme` 的子集
- 计算模型类似 `python`, 但是没有 `statement`, 只由 `expression` 组成

### 3.2.1 expressions
- 两种表达式
	- `call expressions`, 函数调用 `(quotient 10 2), (+ (* 3 5) (- 10 6))`


# 3.3 exceptions
- 错误处理
- 没有处理, `python` 会直接结束程序, 有处理会接着往下执行
```python
>>> raise Exception('An error occurred')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
Exception: an error occurred


>>> try:
        x = 1/0
    except ZeroDivisionError as e:
        print('handling a', type(e))
        x = 0
handling a <class 'ZeroDivisionError'>
>>> x
0


>>> def invert(x):
        result = 1/x  # Raises a ZeroDivisionError if x is 0
        print('Never printed if x is 0')
        return result

>>> def invert_safe(x):
        try:
            return invert(x)
        except ZeroDivisionError as e:
            return str(e)

>>> invert_safe(2)
Never printed if x is 0
0.5
>>> invert_safe(0)
'division by zero'
```


### 3.3.1 exception objects
- `Exception` 是一个内置类, 也可以自己实现, 封装一些有用的错误信息
```python
>>> class IterImproveError(Exception):
        def __init__(self, last_guess):
            self.last_guess = last_guess

>>> def improve(update, done, guess=1, max_updates=1000):
        k = 0
        try:
            while not done(guess) and k < max_updates:
                guess = update(guess)
                k = k + 1
            return guess
        except ValueError:
            raise IterImproveError(guess)
```


# 3.4 interpreters for languages with combination
- 写个 `scheme` 解释器

### 3.4.1 a scheme-syntax calculator
- `scheme` 中, 不管是函数调用表达式, 还是简单的计算表达式, 都使用前缀表达式
```scheme
> (+ 1 2 3 4)
10
> (+)
0
> (* 1 2 3 4)
24
> (*)
1
> (- 10 1 2 3)
4
> (- 3)
-3
> (/ 15 12)
1.25
> (/ 30 5 2)
3
> (/ 10)
0.1
> (- 100 (* 7 (+ 8 (/ -12 -3))))
16.0
```


### 3.4.2 expression trees
- `scheme` 全是表达式, 而且形式较为统一的特性让语法树的构建非常符合直觉, `expression evaluation trees` 这个理论模型可以转化为实际的 `python` 数据, 然后真的进行 `scheme` 表达式的计算

```python
>>> s = Pair(1, Pair(2, nil))
>>> s
Pair(1, Pair(2, nil))
>>> print(s)
(1 2)
>>> len(s)
2
>>> s[1]
2
>>> print(s.map(lambda x: x+4))
(5 6)
>>> expr = Pair('+', Pair(Pair('*', Pair(3, Pair(4, nil))), Pair(5, nil)))
>>> print(expr)
(+ (* 3 4) 5)
>>> print(expr.second.first)
(* 3 4)
>>> expr.second.first.second.first
3
```

### 3.4.3 parsing expressions
- `lexical analysis, tokenizer` 词法分析器
```python
>>> tokenize_line('(+ 1 (* 2.3 45))')
['(', '+', 1, '(', '*', 2.3, 45, ')', ')']
```
- `syntactic analysis` 语法分析, 将 `tokens` 构建成语法树
```python
>>> lines = ['(+ 1', '   (* 2.3 45))']
>>> expression = scheme_read(Buffer(tokenize_lines(lines)))
>>> expression
Pair('+', Pair(1, Pair(Pair('*', Pair(2.3, Pair(45, nil))), nil)))
>>> print(expression)
(+ 1 (* 2.3 45))
```

### 3.4.4 calculator evaluation
- 构建了语法树之后, 就可以进行计算了
- 递归计算
- `reduce, map, simplify` 是非常有用的工具
```python
>>> def calc_eval(exp):
        """Evaluate a Calculator expression."""
        if type(exp) in (int, float):
            return simplify(exp)
        elif isinstance(exp, Pair):
            arguments = exp.second.map(calc_eval)
            return simplify(calc_apply(exp.first, arguments))
        else:
            raise TypeError(exp + ' is not a number or call expression')

>>> def calc_apply(operator, args):
        """Apply the named operator to a list of args."""
        if not isinstance(operator, str):
            raise TypeError(str(operator) + ' is not a symbol')
        if operator == '+':
            return reduce(add, args, 0)
        elif operator == '-':
            if len(args) == 0:
                raise TypeError(operator + ' requires at least 1 argument')
            elif len(args) == 1:
                return -args.first
            else:
                return reduce(sub, args.second, args.first)
        elif operator == '*':
            return reduce(mul, args, 1)
        elif operator == '/':
            if len(args) == 0:
                raise TypeError(operator + ' requires at least 1 argument')
            elif len(args) == 1:
                return 1/args.first
            else:
                return reduce(truediv, args.second, args.first)
        else:
            raise TypeError(operator + ' is an unknown operator')

>>> calc_apply('+', as_scheme_list(1, 2, 3))
6
>>> calc_apply('-', as_scheme_list(10, 1, 2, 3))
4
>>> calc_apply('*', nil)
1
>>> calc_apply('*', as_scheme_list(1, 2, 3, 4, 5))
120
>>> calc_apply('/', as_scheme_list(40, 5))
8.0

>>> print(exp)
(+ (* 3 4) 5)
>>> calc_eval(exp)
17

# REPL
>>> def read_eval_print_loop():
        """Run a read-eval-print loop for calculator."""
        while True:
            try:
                src = buffer_input()
                while src.more_on_line:
                    expression = scheme_read(src)
                    print(calc_eval(expression))
            except (SyntaxError, TypeError, ValueError, ZeroDivisionError) as err:
                print(type(err).__name__ + ':', err)
            except (KeyboardInterrupt, EOFError):  # <Control>-D, etc.
                print('Calculation completed.')
                **return**
```


# 3.5 interpreters for languages with abstraction
- 在实现 `scheme calculator` 的基础上实现 `general programming language`

### 3.5.1 structure
