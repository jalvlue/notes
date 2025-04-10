# Fprint...
```go
func Fprintf(w io.Writer, format string, a ...any) (n int, err error)

...
n, err := fmt.Fprintf(os.Stdout, "%s is %d years old.\n", name, age)
fmt.Printf("%d bytes written.\n", n)

out>: ... is ... years old.
out>: ... bytes written.

```
Fprintf 使用 f 后缀格式化给定的字符串, 但是将格式后的字符串写入到 w 中, 返回的是成功写入的字节数和 error

# Fscan...
```go
func Fscanf(r io.Reader, format string, a ...any) (n int, err error)

...
r := strings.NewReader("5 true gophers")
n, err := fmt.Fscanf(r, "%d %t %s", &i, & b, &s)
fmt.Println(i, b, s, n)

out>: 5 true gophers 3

```
Fscan 与 Fprint 相反, 从 io.Reader 中读出东西, 可以存到变量中

# Print...
```go
func Printf(format string, a ...any) (n int, err error)

...
fmt.Printf("%s is %d years old.\n", "aly", 8)

out>: aly is 8 years old.

```
Print 就是将东西输出到 `os.StdOut` 中

# Scan...
```go
func Scanf(format string, a ... any) (n int, err error)
```
Scan 与 Print 相对应, 从 `os.StdIn` 中读入东西, 可以存入变量中


# Sprint...
```go
func Sprintf(format string, a ...any) string

...
s := fmt.Sprintf("%s is %d years old.\n", name, age)
fmt.Println(s)

out>: ... is ... years old.

```
Sprint 就是将给定的字符串格式化后返回

# Sscan...
```go
func Sscanf(str string, format string, a ...any) (n int, err error)

...
n, err := fmt.Sscanf("Kim is 22 years old", "%s is %d years old", &name, &age)
fmt.Println(name, age)

out>: Kim 22
```
Sscan 与 Sprint 相对应, 在给定的字符串中, 根据对应的格式, 读出其中的变量, 没什么用感觉

