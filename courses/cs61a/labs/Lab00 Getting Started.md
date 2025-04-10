# ok
- `ok` 测试程序尽量使用 `--local` 选项以跳过邮件部分
	- 交互测试: `python ok -q <test_case> -u --local`
	- Lab 测试: `python ok --local`
	- 函数测试 `python ok -q <function_name> --local`


# Python from cli
- 直接执行某个 python 程序 `python lab00.py`
- 交互执行某个 python 程序 (相当于将该文件之前的所有的函数, 变量定义都先导入了, 然后进入 python 交互命令行) `python -i lab00.py`
- 文档测试 `python -m doctest lab00.py`


# doctest
- 文档测试, 就是在某个文件内的 `python doc` 使用特殊的语法指定行为和对应的预期结果, 让 python 可以执行测试

```py
def twenty_twenty_four():

    """Come up with the most creative expression that evaluates to 2024

    using only numbers and the +, *, and - operators.

  

    >>> twenty_twenty_four()

    2024

    """

    return 2024
```

