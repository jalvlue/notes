# 1.1 Getting Started
- `computer science is built upon an elegant and powerful set of fundamental ideas`
- 所有的计算过程都可以概括成三部分
	- 表示数据
	- 规定数据处理逻辑
	- 设计抽象来控制复杂度
- 这个就是计算机的根本工作原理, 也是 `cs fundamental idea`, 了解计算机如何解释程序可以更好的理解计算过程


### 1.1.1 Programming in Python
- 高级编程语言是定义计算过程的载体
- `cs61a` 选择 `python` 来阐述定义计算过程, `python` 有很多的特性, 并且支持多种编程风格
- 在持续发扬 SICP 的精神的同时顺便教一些 python


### 1.1.2 Installing Python 3
skip


### 1.1.3 Interactive Sessions
- python 是脚本语言, 直接在终端中运行 `python` 会进入 python 交互会话, 可以直接一句一句执行语句
- `ctrl+p / up`
- `ctrl+n / down`
- `ctrl+d`


### 1.1.4 First Example
- `statements & expressions`: 计算机指令可以广泛地归为两类
	- `compute some value`, `expression` 表达式通常是进行一些计算的
		- `2 + 3`
		- `len("hello")`
	- `carry out some action`, `statement` 语句通常就是描述一些动作 (逻辑) 的
		- `x = 5`
		- `if x > 10:`
- `functions`: 函数封装了对数据操作的逻辑, 函数是抽象, 控制复杂度, 复用的很重要的一部分
	- 使用者只需要一行函数调用即可实现复杂的功能, 因为逻辑都被封装在函数内了
	- `shakespeare = urlopen("http://composingprograms.com/shakespeare.txt")`
- `objects`: 对象
	- 一个对象可以存储一些描述对象性质的属性
	- 一个对象可以有很多种与之相关联的方法
	- 因此对象是将数据和操作数据的逻辑无缝衔接起来的一种抽象, 提供了非常强的复杂度管理能力
	- `words = set(shakespeare.read().decode().split())`
- `interpreters`: 解释器
	- 解释器是一种复杂的程序, 同样包含很多逻辑
	- 作用是仔细地处理计算所有的复杂表达式

### 1.1.5 Errors
- 程序出错是非常正常的, 解决错误的过程就是 debug
	- `test incrementallly`, 将复杂软件模块化, 组件化, 做增量单元测试
	- `isolate errors`, 定位错误
	- `check your assumptions`, 大胆假设, 小心求证
	- `consult others`


# 1.2 Elements of Programming
- 高级语言更像是一种约定俗成的代码框架, 主要是给人看的, 给机器执行是顺带的
- 高级语言提供了非常强有力的基本构建模块, 并且能够将这些基本模块组合成复杂的程序
	- `primitive expressions and statements`: 基本模块
	- `means of combination`: 将基本模块组合成复杂程序的能力
	- `means of abstraction`: 抽象能力, 控制程序复杂度
- Elements of Programing
	- `functions`: 操作逻辑
	- `data`: 操作的数据


### 1.2.1 Literal Expressions
- 一切字面量是最简单的表达式, 利用操作符可以创造出复杂的组合表达式, 然后 python 会 `evaluate` 表达式的最终结果, 得出一个值
	- `42`
	- `-1 - -1`
	- `2 * 7`


### 1.2.1 Call Expressions
- 函数调用表达式是另外一种组合表达式的方法, 将一切参数输入, 然后执行函数逻辑, 返回一个处理参数后的值
	- `max(7.5, 9)`
	- `pow(100, 2)`
	- `max(min(1, -2), min(pow(3, 5), -4))`
	- ![[Pasted image 20240810110936.png]]
- 函数的另一个优势是, 在计算非常复杂的时候, 比字面量直接组合更加容易调用 (使用者)


### 1.2.3 Importing Library Functions
- `python` 有很多最常用的内置函数
- 其他的常见函数按照功能被划分到不同模块中, 所有这些模块组成了 python 标准库, 按需导入使用
- `import ...`
- `from ... import ...`

```python
from math import sqrt
sqrt(256)

from operator import add, sub, mul
add(100, 2)
sub(199, mul(7, add(8, 4)))
```

### 1.2.4 Names and the Environment
- 高级编程语言的另一个关键特性是可以使用一个 `name` 去代表一个计算对象 `computational object`, 也就是, 可以使用变量来将计算过程或计算结果存储下来
- 这个过程通过赋值语句 `assign statement` 进行
- 赋值表达式是最基本的抽象, 允许我们用一个 `name` 绑定一个复杂操作的结果, 或者一个复杂操作本身 `f = max`
- 复杂程序就是通过这样逐渐叠加复杂度后建造的
- 解释器需要维护一个环境 `environment` 来允许上面 `name binding`
- Python 是弱类型解释性语言, 类型不是赋值绑定的限制
- 赋值语句首先计算表达式的值, 然后做 `name binding`

```python
radius = 10
2 * radius

from math import pi
pi * 71 / 233

f = max
f(2, 3, 4)

x = 2
x = x + 1

area, circumference = pi * radius * radius, 2 * pi * radius

y, x = x, y
```


### 1.2.5 Evaluating Nested Expressions
- 对一个 `call expression`, 首先做的就是先计算出所有参数对应的表达式的值, 然后再将这些值作为参数传入函数中进行处理, 获得返回值
- 因此对于 `nested call expressions`, 这是天然递归的, 就像是一颗树一样 ![[Pasted image 20240810113239.png]]
- 递归总要有 `base case`, 在表达式递归树中, 就是
	- 字面量
	- `binding names`
- 因此函数调用的环境很重要


### 1.2.6 The Non-Pure Print Function
- 两种函数
	- `pure functions`: 根据输入获取输出, 没有其他任何副作用 ![[Pasted image 20240810114258.png]]
	- `non-pure functions`: 有副作用, 可以改建计算机或者解释器的状态 ![[Pasted image 20240810114338.png]]
- `pure functions` 虽然有很多限制, 但是它行为更加一致, 可控, 测试也更加简单, 在并发编程中, 也很依赖它的一致性



# 1.3 Defining New Functions
- 定义函数, 将复杂的操作逻辑封装成一个单元, 然后做 `name binding`

```python
from operator import mul


def square(x):
    return mul(x, x)


def sum_square(x, y):
    return mul(x, x) + mul(y, y)


print(sum_square(3, 4))

```


### 1.3.1 Environments
- 实际上, 在 `cs61a` 这种距离底层很远的语境中, 函数运行时的 `environment`, 就是该函数在进程栈中的栈帧 `frame`
- 函数中所涉及到的所有 `name binding` 都会首先从自己的栈帧里面找, 没找到再依次向外找 (闭包函数依次向外找, 如果只是普通的调用可能只会在自己的栈帧和全局栈帧里面找, 修正: 实际上只能在自己的栈帧和全局栈帧这两个中招, 闭包只是将变量在闭包函数体内加了一个引用而已, 实际上还是在自己的栈帧内) ![[Pasted image 20240810170612.png]] ![[Pasted image 20240810170621.png]]


### 1.3.2 Calling User-Defined Functions
- 调用一个函数时, 解释器会自动为该函数创建一个栈帧
- 首先会将函数调用的参数 `name binding` 放到该栈帧里面去
- 然后将该栈帧作为执行环境的开始来执行该函数 (函数自己的栈帧和全局栈帧组成它的执行环境)
- 这个例子就涉及到了同时从 `local frame + global frame` 读数据 ![[Pasted image 20240810171351.png]]


### 1.3.3 Example: Calling a User-Defined Function
- 一个调用示例, 在计算函数参数时调用了很多个其他函数, 从而创建了很多个栈帧
![[Pasted image 20240810221451.png]]


### 1.3.4 Local Names
- 局部变量, 例如函数参数等, 作用域仅限于这个函数体内, 也就是这个函数的栈帧之内


### 1.3.5 Choosing Names
- 给变量, 函数命名是很困难的, 但是遵循以下规则可以写出非常清晰的命名
	- 函数: 函数名小写, 单词之间下划线分开, 尽量使用多个单词创造描述性的函数名, 描述函数对数据的操作 (`add, mul...`), 描述函数的返回结果 (`max, abs...`) 
	- 变量: 小写, 单词之间下划线分开, 尽量使用一个单词描述, 尽量不要使用一个字符的变量名


### 1.3.6 Functions as Abstraction
- 函数是一种抽象, 在使用者看来只需理解函数的作用, 然后尽管使用就好, 不是很需要了解被调用函数的实现细节
- 函数抽象有三个性质
	- 输入域
	- 输出域
	- 函数作用/意图/输入输出的关系
- 了解函数性质对于使用该函数很重要


### 1.3.7 Operators
- Python 支持直接使用数学运算符组合成的中序表达式
- `(2 + 3) * (4 + 5)` 等同于 `mul(add(2, 3), add(4, 5))`
- 需要注意的运算符
	- Python 的除法会被计算为一个浮点数, 不论数学上结果是否整数, 因此可以使用整除运算符如果想得到整数

```python
>>> 5 / 4
1.25
>>> 8 / 4
2.0

>>> 5 // 4
1
>>> -5 // 4
-2

>>> from operator import truediv, floordiv
>>> truediv(5, 4)
1.25
>>> floordiv(5, 4)
1
```



# 1.4 Designing Functions
- 函数是进行计算的重要工具, 这里关注怎么写好一个函数
- 根本上, 优秀的函数都强调了函数是抽象的概念
	- 每个函数都应该只实现一个具体的功能, 并且该功能应该可以在一行描述之内表达出来
	- `DRY-don't repeat yourself`, 不要写重复的代码, 如果需要重复复制粘贴就应该考虑将逻辑抽象出来成为一个抽象函数了
	- 函数应该考虑泛化和通用性, 泛型


### 1.4.1 Documentation
- python 习惯的做法是给函数内编写注释文档, 称为 `docstring`
- 有这种格式的注释文档的函数可以使用 `help(function_name)` 来获得文档
- 单行注释也很重要

```python
>>> def pressure(v, t, n):
        """Compute the pressure in pascals of an ideal gas.

        Applies the ideal gas law: http://en.wikipedia.org/wiki/Ideal_gas_law

        v -- volume of gas, in cubic meters
        t -- absolute temperature in degrees kelvin
        n -- particles of gas
        """
        k = 1.38e-23  # Boltzmann's constant
        return n * k * t / v

>>> help(pressure)

```


### 1.4.2 Default Argument Values
- 函数参数默认值, 给调用者减轻很多心智负担
- 指导原则是对于功能比较复杂, 参数比较的的函数, 应该尽可能提供默认参数

```python
>>> def pressure(v, t, n=6.022e23):
        """Compute the pressure in pascals of an ideal gas.

        v -- volume of gas, in cubic meters
        t -- absolute temperature in degrees kelvin
        n -- particles of gas (default: one mole)
        """
        k = 1.38e-23  # Boltzmann's constant
        return n * k * t / v

>>> pressure(1, 273.15)
2269.974834
>>> pressure(1, 273.15, 3 * 6.022e23)
6809.924502
```


# 1.5 Control
- `control statement`, 控制流语句, 是一种 `statement`, 指导解释器下一步执行什么, 对编写复杂的函数逻辑非常有帮助


### 1.5.1 Statements
- 虽然之前已经说过很多语句了 `=, def, return`
- 但是表达式也可以作为语句, 例如只调用一个 `pure function`, 但是不做其他任何操作, 不保存返回值, 虽然这样一点作用都没有, 但这是一个语句
- 调用一个 `non-pure function` 的语句则很有意义, 例如 `print(square(x))`
- python 很多的工作其实都在计算 `expressions` 上, `statements` 则是起到指导作用


### 1.5.2 Compound Statements
- 复合语句, 将一些逻辑上相关的简单语句组合在一块, 实现复杂的功能
	- `def`
	- `if`
	- `while`
	- `for i in range(...)`
- 复合语句的 `header` 决定了组内简单语句的执行顺序的规则


```python
<header>:
    <statement>
    <statement>
    ...

<separating header>:
    <statement>
    <statement>
    ...
...
```


### 1.5.3 Defining Functions II: Local Assignment
- 函数一旦开始执行, 就可以在自己的栈帧内修改栈帧的环境
- 例如在自己栈帧内声明变量, 这种声明赋值只会修改函数自己栈帧的环境, 无法影响到其他栈帧

![[Pasted image 20240811110331.png]]


### 1.5.4 Conditional Statements
- `Boolean contexts`
	- `0, None, False` 的表达式都不会跳入控制语句内部
- `Boolean values`
	- `>, <, >=, <=, ==, !=`
- `Boolean operators`
	- `True and False`
	- `True or False`
	- `not False`


```python
if <expression>:
    <suite>
elif <expression>:
    <suite>
else:
    <suite>

def absolute(x):
    if x > 0:
        return x
    elif x == 0:
        return 0
    else:
        return -x

```

### 1.5.5 Iteration
- 迭代

```python
def fib(n):
    pred, curr = 0, 1
    k = 2
    while k < n:
        pred, curr = curr, pred + curr
        k = k + 1

    return curr

```


### 1.5.6 Testing
- `Assertions`: 使用 python 内置的 `assert` 关键字来验证函数的输出是否复合预期
- `Doctests`: 文档测试, 在函数的注释文档中使用特殊的语法来定义一些测试 
- 可以在 python 文件中导入包进行文档测试, 也可以在终端中使用选项进行文档测试 `python3 -m doctest <python_source_file>`

```python
def absolute_test():
    assert fib(1) == 0
    assert fib(3) == 1
    assert fib(8) == 13
    assert fib(50) == 7778742049


from doctest import testmod
from doctest import run_docstring_examples


def sum_naturals(n):
    """Return the sum of the first n natural numbers.

    >>> sum_naturals(10)
    55
    >>> sum_naturals(100)
    5050
    """
    total, k = 0, 1
    while k <= n:
        total, k = total + k, k + 1
    return total


absolute_test()

testmod()
run_docstring_examples(sum_naturals, globals(), True)

```

# 1.6 Higher-Order Functions
- 函数是一级公民, 可以被绑定到另外很多个名字中
- 被绑定后作为另外一个函数的参数或者返回值
- 这种用法带来了更强大的抽象能力, 称为高阶函数 `higher-order function`


### 1.6.1 Functions as Arguments
- 三个结构非常类似的函数的例子, 都是用于在迭代过程中求某种特定的和

```python
>>> def sum_naturals(n):
        total, k = 0, 1
        while k <= n:
            total, k = total + k, k + 1
        return total

>>> sum_naturals(100)
5050

>>> def sum_cubes(n):
        total, k = 0, 1
        while k <= n:
            total, k = total + k*k*k, k + 1
        return total

>>> sum_cubes(100)
25502500

>>> def pi_sum(n):
        total, k = 0, 1
        while k <= n:
            total, k = total + 8 / ((4*k-3) * (4*k-1)), k + 1
        return total

>>> pi_sum(100)
3.1365926848388144

def <name>(n):
    total, k = 0, 1
    while k <= n:
        total, k = total + <term>(k), k + 1
    return total

```
- 因此我们可以将求和方式抽象出来, 跟具体的计算方式解耦, 创造一种更抽象更通用的方式

```python
def summation(n, term):
	total, k = 0, 1
	while k <= n:
		total, k = total + term(k), k+1
	return total

def cube(x):
	return x*x*x

def sum_cubes(n):
	return summation(n, cube)

def identity(x):
	return x

def sum_natural(n):
	return summation(n, identity)

```


### 1.6.2 Functions as General Methods
- 有了高阶函数之后, 就可以利用高阶函数的特性, 在高阶函数内只定义结构, 计算模型, 具体细节则由参数函数决定
- 从而获得一些通用的能力(计算模型), 而不是只解决某个特定的问题, 所有解决问题的细节(计算细节)都被丢到参数函数中, (是不是很像 map reduce)
- 例如这里例子定义了获取黄金分割比 `φ` 的 `update, close` 函数

```python
>>> def improve(update, close, guess=1):
        while not close(guess):
            guess = update(guess)
        return guess

>>> def golden_update(guess):
        return 1/guess + 1

>>> def square_close_to_successor(guess):
        return approx_eq(guess * guess, guess + 1)

>>> def approx_eq(x, y, tolerance=1e-15):
        return abs(x - y) < tolerance

>>> improve(golden_update, square_close_to_successor)
1.6180339887498951

```

> This example illustrates two related big ideas in computer science. First, naming and functions allow us to abstract away a vast amount of complexity. While each function definition has been trivial, the computational process set in motion by our evaluation procedure is quite intricate. Second, it is only by virtue of the fact that we have an extremely general evaluation procedure for the Python language that small components can be composed into complex processes. Understanding the procedure of interpreting programs allows us to validate and inspect the process we have created.


### 1.6.3 Defining Functions III: Nested Definitions
- 上面的高阶函数还有两个问题
	- 全局栈帧中有太多的函数定义了, 每一个的名字还不能相同, 弄得很脏
	- 所有的参数函数的签名必须一致
- 这个例子就使用嵌套函数定义解决了这些问题
- 通用的高阶函数 `improve()` 接收的参数函数 `update` 只能有一个参数, 因此嵌套定义一个闭包函数 `sqrt_update()`, 由于是闭包, 可以访问外面函数的参数, 因此签名就可以满足 `update()` 的签名格式
- 局部的 `def` 定义只会出现并只能在局部栈帧内被访问使用
- 这样子在函数内部嵌套定义函数的好处是
	- 之前的函数的 `environment` 只有自己的栈帧和全局栈帧, 闭包函数可以访问到其父函数中的所有变量, 也就类似可以访问到其父函数的栈帧, 因此延长了环境链
	- 延长后, 寻找变量的顺序是从本地局部开始, 一直向外层, 直到全局栈帧
	- 只有父函数有访问权, 相当于是私有的

```python
>>> def average(x, y):
        return (x + y)/2

>>> def sqrt_update(x, a):
        return average(x, a/x)

>>> def sqrt(a):
        def sqrt_update(x):
            return average(x, a/x)
        def sqrt_close(x):
            return approx_eq(x * x, a)
        return improve(sqrt_update, sqrt_close)

```


### 1.6.4 Functions as Returned Values
- 闭包函数作为一等公民同样可以被返回, 而且返回的闭包函数同样有访问父函数的栈帧的能力
- 这里定义了一个闭包, 将父函数的两个参数组合起来成为一个新函数然后返回

```python
def square(x):
	return x*x

def successor(x):
	return x+1

def compose(f, g):
	def h(x):
		return f(g(x))
	return h

square_succssor = compose(square, successor)
result = square_successor(12)
```

### 1.6.5 Example: Newton's Method
- 这里用高阶函数来编写牛顿法求方根
- 例如求 `64^-2`, 转换成方程 `x*x - 64 = 0`, 从而转换成了求 `f(x)=x*x - 64` 的 `x`
- `update` 利用导数不断逼近

```python
def improve(update, close, guess=1):
    while not close(guess):
        guess = update(guess)
    return guess


def approx_eq(x, y, tolerance=1e-15) -> bool:
    return abs(x - y) < tolerance


def newton_update(f, df):
    def update(x):
        return x - f(x) / df(x)

    return update


def find_zero(f, df):
    def near_zero(x):
        return approx_eq(f(x), 0)

    return improve(newton_update(f, df), near_zero)


def square_root_newton(a):
    def f(x):
        return x * x - a

    def df(x):
        return 2 * x

    return find_zero(f, df)


def power(x, n):
    product = 1
    k = 0
    while k < n:
        product *= x
        k = k + 1

    return product


def nth_root_newton(n, a):
    def f(x):
        return power(x, n) - a

    def df(x):
        return n * power(x, n - 1)

    return find_zero(f, df)


x = square_root_newton(64)
print(x)

x = nth_root_newton(3, 64)
print(x)

x = nth_root_newton(6, 64)
print(x)

```

### 1.6.6 Currying
- 函数柯里化
- `f(x, y) === g(x)(y)`, 将单一函数转换为链式调用的过程称为柯里化
- 例如 `pow(x, y)` 可以通过柯里化变成更加专用的函数, 在柯里化函数的内部将 `x` 确定好
- 柯里化的关键之处在于函数只能够有一个参数时, 的链式化

```python
def curry_power(x):
    def h(y):
        return pow(x, y)

    return h


curry_power(2)(3)

def map_to_range(start, end, f):
    while start < end:
        print(f(start))
        start = start + 1


map_to_range(0, 10, curry_power(2))

```


### 1.6.7 Lambda Expression
- `lambda` 表达式可以创建匿名函数, 主要用于简单的函数或者不会重复使用的函数
- 但是 `lambda` 表达式也可以赋予到一个名字上, 成为一个有名字可以被调用的函数

```python
     lambda            x            :          f(g(x))
"A function that    takes x    and returns     f(g(x))"


def compose_lambda(f, g):
    return lambda x: f(g(x))


f = compose_lambda(lambda x: x * x, lambda y: y + 1)

square_lambda = lambda x: x * x
square_lambda(12)

```


### 1.6.8 Abstractions and First-Class Functions
- 一等公民函数
	- 可以被赋值到变量上
	- 可以作为参数被传递
	- 可以被返回
	- 可以被包含在数据结构中, 成为一部分
- `python` 的函数就是一等公民


### 1.6.9 Function Decorators
- 函数装饰器, `python` 提供的一种功能, 只需要在函数定义之上加上特定的装饰器语法就可以在该函数追加某些功能
- 有点类似中间件, 定义中间处理函数, 然后往下调用

```python
def trace(fn):
    def wrapped(x):
        print("->", fn, "(", x, ")")
        return fn(x)

    return wrapped


@trace
def triple(x):
    return 3 * x


x = triple(12)
print(x)

>>>
-> <function triple at 0x7fb33ff36ef0> ( 12 )
36
```


# 1.7 Recursive Functions
- 递归老朋友, 在函数体内直接或间接调用自己
- 写好递归主要就是:
	- 将问题分解成子问题, 一部分在递归处理, 一部分在当前函数处理
	- 相信子递归调用的承诺, 专注处理当前的处理
	- 设置好 `base case`
	- 不要思考太多子递归的细节, 相信子递归调用的承诺就好

```python
def sum_digits(n):
	if n < 10:
    	return n
	else:
    	last = n % 10
    	all_but_last = n // 10
    	return sum_digits(all_but_last) + last
```


### 1.7.1 the anatomy of recursive functions
- `base cases`, 递归的最深处, 通常是一些最小的, 不可再分解的子问题, 对于这些问题, 可以直接处理而不用再划分递归, 可以有很多个, 因此成为 `base cases`
- `base cases` 是递归的很重要的一部分, 子问题的规模收敛到并踩中 `base cases` 之后, 递归才能停止
- 之后在函数体内会将问题分为两块
	- 一部分直接解决,
	- 另一部分调用递归函数解决

```python
def fact_iter(n):
    total, k = 1, 1
    while k <= n:
        total *= k
        k += 1
    return total


def fact(n):
    if n == 0 or n == 1:
        return n
    return n * fact(n - 1)

```
- 迭代: 自底向上, 从 1 开始构建答案, 直到触碰到 n
- 递归: 自顶向下, 从 n 开始构建答案, 直到触碰到 `base cases`, 使用递归其实是更符合直觉, 更符合数学定义的

> While we can unwind the recursion using our model of computation, it is often clearer to think about recursive calls as functional abstractions. That is, we should not care about how fact(n-1) is implemented in the body of fact; we should simply trust that it computes the factorial of n-1. Treating a recursive call as a functional abstraction has been called a _recursive leap of faith_.
- 对子问题递归调用的信仰
- 递归函数相比于迭代函数不需要在函数体内维护过多的局部变量, 这些细节都在递归调用的 `environment` 中定义了, 因此可以更加专注于函数逻辑本身


### 1.7.2 mutual recursion
- 相互递归, 两个函数相互调用对方, 从而形成了间接递归调用
- 对一个整数进行递归奇偶判断是相互递归的最好例子
```python
def is_even(n):
    if n == 0:
        return True
    else:
        return is_odd(n - 1)


def is_odd(n):
    if n == 1:
        return True
    else:
        return is_even(n - 1)

```


### 1.7.3 printing in recursive function
- 在递归的过程中将中间问题打印出来是可视化递归的一种方式
```python
def cascade(n):
"""Print a cascade of prefixes of n."""
print(n)
if n >= 10:
    cascade(n // 10)
    print(n)

```

### 1.7.4 tree recursion
- 树形递归, 在函数体内多次进行递归调用, 从而形成递归调用树

```python
def fib(n):
	if n == 1:
	    return 0
	if n == 2:
	    return 1
	else:
	    return fib(n - 2) + fib(n - 1)

def cascade(n):
    if n < 10:
        print(n)
    else:
        print(n)
        cascade(n // 10)
        print(n)


def inverse_cascade(n):
    grow(n)
    print(n)
    shrink(n)


def f_then_g(f, g, n):
    if n:
        f(n)
        g(n)


grow = lambda n: f_then_g(grow, print, n // 10)
shrink = lambda n: f_then_g(print, shrink, n // 10)

```


### 1.7.5 example: partitions
- 计算 `n` 可以由 `1 ... m` 组成的组合数 `count_parttions(n, m)`
```python
def count_partitions(n, m):
	if n == 0:
	    return 1
	if n < 0:
	    return 0
	if m == 0:
	    return 0
	else:
	    return count_partitions(n - m, m) + count_partitions(n, m - 1)

```
- 为什么要按照是否包含 `m` 将问题分为两个子问题
- 第一类, 其实类似于 dfs, 当前的 `n` 足够容下至少一个 `m`, 因此先将这个 `m` 拿下, 剩余的 `n-m` 再继续探索可能性, 知道 `n==0` 时, 这种可能性走通了, 增加一个组合
- 第二类, 逐步降低 `m` 的容量

> To see why this is true, observe that all the ways of partitioning n can be divided into two groups: those that include at least one m and those that do not. Moreover, each partition in the first group is a partition of n-m, followed by m added at the end. In the example above, the first two partitions contain 4, and the rest do not. Therefore, we can recursively reduce the problem of partitioning n using integers up to m into two simpler problems: (1) partition a smaller number n-m, and (2) partition with smaller components up to m-1.

```ad-hint
将整数分区问题分为两类（使用当前最大部分和不使用当前最大部分）是为了通过递归方法有效地探索所有可能的分区组合。以下是这样分类的原因及其优势：

### 1. **明确的选择路径**

- **使用当前最大部分 (`m`)**:
  - 选择将 `m` 包含在分区中，意味着我们可以从 `n` 中减去 `m`，并继续寻找分区，这样可以保证分区中的最大数不超过 `m`。此时，我们仍然可以使用 `m` 作为下一部分的一部分，因此可以继续进行分区。

- **不使用当前最大部分 (`m`)**:
  - 选择不使用 `m`，意味着我们将 `m` 减少为 `m-1`，并继续寻找分区。这允许我们探索使用更小的整数来填补 `n`，确保我们不会错过可能的分区组合。

### 2. **避免重复计算**

通过将问题分为这两类，我们可以确保每种组合只计算一次。例如，考虑 `n = 5` 和 `m = 2` 的情况：

- 如果选择使用 `2`，我们可能会得到分区 `2 + 2 + 1`。
- 如果选择不使用 `2`，那么我们会考虑 `m = 1` 的情况，得到分区 `1 + 1 + 1 + 1 + 1` 和其他组合。

这种设计可以避免重复计算相同的分区，提升效率。

### 3. **递归的可组合性**

这种分解使得递归的层次分明，每个递归调用都在解决一个子问题。通过将问题拆分为两种情况，整个问题的解决变得更为系统化和有条理，便于理解和实现。

### 4. **动态规划的基础**

这种分类还为后续使用动态规划提供了基础。虽然当前实现是递归的，但可以很容易地转化为动态规划，通过记录已经计算过的结果来减少重复计算，从而提升效率。

### 总结

将整数分区问题分为两类（使用当前最大部分和不使用当前最大部分）是一种有效的策略，可以系统地探索所有可能的分区组合。这种方法不仅清晰明了，还有助于避免重复计算，为进一步的优化（如动态规划）奠定基础。通过这样的分类，函数能够以递归的方式高效地解决分区问题。
```

