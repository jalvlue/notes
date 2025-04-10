`!!`: bangbang 将这一部分替换为上次使用的命令
例如某个命令第一次没加权限, 第二次就可以直接 
`sudo !!`, 让 `!!` 替换为上次执行的命令

`cd -`: 移动到上次所在位置

`> <`: 重定向符号, 重定向到文件或重定向到流

`|`: 管道, 把一个程序的输出接到下一个程序作为输入
```bash
ls -l / | tail -n1
```

`tee file content`: 将内容写入文件中, 同时还在屏幕上输出

`tldr`: 简单的, 例子导向的 `man`, 可以更直观地看到一个命令的用法

`find`: 根据名字查找文件夹, 文件
```bash
find . -name src -type d
find . -name '**/type/*.py' -type f
```
还有很多有用的 flag
`fd`: `find` 的简单版本

`grep`: 寻找文件中的指定文本模式
`rg`: ripgrep, 类似 `grep` 的寻找命令

这些程序都可以通过设置很多 flag 来实现很多功能, 保持探索

`ctrl+R`: 对 shell 历史进行反向搜索
按了 `ctrl+R` 后, 输入要寻找的命令的一部分, 就可以一直按 `ctrl+R` 不断上翻包含这部分命令的历史, 方便寻找历史命令


### shell 变量
```bash
> foo=bar
> echo $foo
bar

> echo "foo is $foo"
foo is bar

> echo 'foo is $foo'
foo is $foo
```
直接使用不带空格的等号复制, 带了空格 shell 会解释为执行某个程序, 使用 `$` 获得变量的值
使用双引号 `""` 阔着字符串时, shell 可以读取并替换 `$` 变量值, 使用单引号 `''` 则不会

### shell 脚本
在 shell 中, 脚本参数在脚本中呈现为 `$0~$9`, 就类似 C,
其中 `$0` 代表脚本名称, 其他则为顺序传入的参数,
`$_` 表示上次命令使用的参数
`$?` 表示上次命令的错误码
```bash
mcd () {
	echo "script name: $0"
	mkdir -p "$1"
	cd "$1"
}

> mcd test

```

大合集例子
```bash
#!/bin/bash

echo "Starting program at $(date)"

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
	grep foobar "$file" > /dev/null 2> /dev/null

	if [[ "$?" -ne 0 ]]; then
		echo "File $file does not having any foobar, adding one"
		ehco "# foobar" >> "$file"
	fi
done
```

### 通配符以及拓展符
类似正则表达式, `*` 匹配任意字符串, 但是 `?` 匹配一个字符
`{}` 会做拓展
```bash
touch foo{,1,2}/src/s{a,b}.py
touch foo/src/sa.py foo/src/sb.py foo1/src/sa.py foo1/src/sb.py foo2/src/sa.py foo2/src/sb.py 
```
两个命令是等价的, `{}` 自动做了拓展

