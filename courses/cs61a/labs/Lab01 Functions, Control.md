# WWPD
- Python 默认返回值是 `None`
- Python 内建的 `print()` 语句可以打印 `None`
- `print(None, None)` 语句会将打印出 `None None`, 默认分隔符是空格, 也可以指定分隔符, `sep='...'` 参数


# DEBUG
- `print()` 是最好用的 debug 工具, 为了防止 `debug print` 的内容被 `ok` 误读, 需要在 `print()` 中加上 `DEBUG:` 前缀, `print("DEBUG:...")`
