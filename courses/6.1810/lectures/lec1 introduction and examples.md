# Overview
Os purposes
1. Abstract hardware
2. Multiplex
3. Isolation
4. Sharing
5. Security
6. Performance
7. Range of uses

# OS architecture

### User & kernel space
User space: 运行用户程序, vi, cc, sh ...
Kernel space: 只运行一个程序, 就是 os kernel

Kerner 维护大量数据结构来管理各种硬件资源
Kernel 提供很多服务
1. Fs
2. Process control: schedule...
3. Memory alloc
4. Access control: access hareware resources

课程关注内核, 以及内核和用户程序之间的交互 `system call API`

### Kernel api
Kernel API: 内核和用户程序交互的接口 (内核给用户提供的服务)
一些例子: 
-  `fd = open("out", 1)`, 以特定权限打开一个文件, 返回文件描述符以供后续使用
-  `write(fd, "hello\n", 6)`
-  `pid = fork()`

这些都是系统调用, 虽然看起来很像是普通的函数调用
但是实际上在执行这些代码时, 系统会跳转进内核, 然后执行内核代码

普通函数调用和系统调用的区别:
系统调用拥有直接跟硬件打交道的权限, 可以进行直接物理磁盘读写之类的敏感操作, 普通函数调用是永远无法做到的


# Hard and interesting
- 底下就是硬件, 编程环境比较恶劣
- 为其他应用服务提供基础设施, 需要精心编写 API 以提供可靠的服务
- API 设计中, 简洁/强大/灵活的取舍
- ...


# Read, write, exit system calls
- `xv6`
- `riscv CPU`
- `QEMU`

### Example: copy. C
```c
#include "kernel/types.h"
#include "user/user.h"

int main(int argc, char const *argv[])
{
  char buf[64];

  while (1)
  {
    int n = read(0, buf, sizeof(buf));
    if (n <= 0)
    {
      break;
    }
    write(1, buf, n);
  }

  exit(0);
}

```
- 这里就利用了 `read(), write(), exit()` 三个系统调用
- 从标准输入读 (fd=0)
- 向标准输出写 (fd=1)
- 没读到数据时最后退出, (在标准输入中输入 `ctrl+D` 时, `read()` 会读到长度为 `0` 的数据块, 从而退出)

# Open system call
```c
#include "kernel/types.h"
#include "user/user.h"
#include "kernel/fcntl.h"

int main(int argc, char const *argv[])
{
  int fd = open("output.txt", O_WRONLY | O_CREATE);
  write(fd, "ooo\n", 4);

  exit(0);
}

```
- 这里就使用了 `open()` 系统调用, 并使用 `kernel/types.h` 中的特定读写权限打开
- 然后使用 `write()` 系统调用将某些内容通过打开后获得的文件描述符写到文件中
- 文件描述符是一个正整数
- 内核维护了每个运行进程的状态, 内核会为每一个运行进程保存一个表单, 表单的 key 是文件描述符
- 这个表单让内核知道，每个文件描述符对应的实际内容是什么
- 每个进程都有自己独立的文件描述符空间


# Shell
- `cli` 接口, 就像是计算机的一个外壳 (shell) 一样, 帮助我们调用我们想执行的程序例如 `ls`
- shell 除了可以运行程序之外, 还可以进行 IO 重定向, `ls > out.txt`, `grep x < out.txt`


# Fork system call
```c
#include "kernel/types.h"
#include "user/user.h"

int main(int argc, char const *argv[])
{
  printf("common start\n");

  int pid = fork();
  printf("fork() returned: %d\n", pid);

  if (pid == 0)
  {
    printf("child\n");
  }
  else
  {
    printf("parent\n");
  }

  exit(0);
}

```
- `fork()` 拷贝当前进程的内存 (指令+数据 (包括共享的文件描述符)), 并创建一个新的进程, 返回一个子进程的 `pid`
- 返回的进程会从 `fork()` 的调用之后开始执行


# Exec, wait system call
```c
#include "kernel/types.h"
#include "user/user.h"

int main()
{
  char *args[] = {"echo", "this", "is", "echo", 0};

  exec("echo", args);

  printf("exec failed\n");

  exit(0);
}

```
- `exec(const char *, char **)` 会从指定的文件读取数据(指令), 并加载到当前进程的内存中, 然后利用指定的参数执行加载的程序
- 上面的例子中, 就加载了 `echo` 程序, 并传递了一些参数
- `exec()` 将会保留进程的文件描述符表单, 因此在后续加载的程序中, 也可以使用之前的文件描述符 (这一点很有用, 可以玩很多 trick)
- `exec()` 如果成功将不会返回, 因为直接完全替换成了目标程序

```

学生提问：argv中的最后一个0是什么意思?

Robert教授：它标记了数组的结尾。C是一个非常低阶（接近机器语言）的编程语言，并没有一个方法来确定一个数组究竟有多长。所以为了告诉内核数组的结尾在哪，我们将0作为最后一个指针。argv中的每一个字符串实际上是一块包含了数据的内存指针，但是第5个元素是0，通常来说指针0是一个NULL指针，它只表明结束。
所以内核中的代码会遍历这里的数组，直到它找到了值为0的指针。

```

```c
#include "kernel/types.h"
#include "user/user.h"

int main(int argc, char const *argv[])
{
  int pid, status;
  pid = fork();

  if (pid == 0)
  {
    char *args[] = {"echo", "this", "is", "echo", 0};
    exec("echo", args);
    printf("exec failed\n");
    exit(1);
  }
  else
  {
    printf("parent waiting\n");
    wait(&status);
    printf("child exited with status %d\n", status);
  }

  exit(0);
}

```
- `fork() + exec() + wait()` 是非常常见的 `unix` 调用风格, shell 就是这样做的
- `shell` 主程序会保持不断运行, 因此所有传递进去的指令都会使用 `fork() + exec()` 新创子进程来执行我们指定的命令
- 然后 `shell` 调用 `wait(&childStatus)` 等待子进程的执行结束, 并将子进程的退出状态写进 `childStatus` 中
- 对于大型程序, `fork() + exec()` 会拷贝原程序的内存, 但是然后完全丢弃, 可能会有一些浪费和性能影响, 因此后面会有 `copy-on-write fork()` 来优化

```ad

学生提问：当我们说子进程从父进程拷贝了所有的内存，这里具体指的是什么呢？是不是说子进程需要重新定义变量之类的？

Robert教授：在编译之后，你的C程序就是一些在内存中的指令，这些指令存在于内存中。所以这些指令可以被拷贝，因为它们就是内存中的字节，它们可以被拷贝到别处。通过一些有关虚拟内存的技巧，可以使得子进程的内存与父进程的内存一样，这里实际就是将父进程的内存镜像拷贝给子进程，并在子进程中执行。

实际上，当我们在看C程序时，你应该认为它们就是一些机器指令，这些机器指令就是内存中的数据，所以可以被拷贝。

学生提问：如果父进程有多个子进程，wait是不是会在第一个子进程完成时就退出？这样的话，还有一些与父进程交错运行的子进程，是不是需要有多个wait来确保所有的子进程都完成？

Robert教授：是的，如果一个进程调用fork两次，如果它想要等两个子进程都退出，它需要调用wait两次。每个wait会在一个子进程退出时立即返回。当wait返回时，你实际上没有必要知道哪个子进程退出了，但是wait返回了子进程的进程号，所以在wait返回之后，你就可以知道是哪个子进程退出了。

```


# IO redirect
```c
#include "kernel/types.h"
#include "user/user.h"
#include "kernel/fcntl.h"

int main(int argc, char const *argv[])
{
  int pid;
  pid = fork();

  if (pid == 0)
  {
    close(1);
    open("redirect.txt", O_CREATE | O_WRONLY);

    char *argv[] = {"echo", "this", "is", "redirected", "echo", 0};
    exec("echo", argv);
    printf("exec failed\n");
    exit(1);
  }
  else
  {
    wait((int *)0);
  }

  exit(0);
}

```
- 这里演示了重定向的操作
- 在子进程中, 将标准输入关闭然后打开 `redirect.txt` 文件, 此时这个文件会被赋予描述符 `1`
- 同时, `echo` 默认向文件描述符 `1` 写入, 这样就完成了在子进程中, 将标准输出重定向到某个文件
- 这里的一个优点是: `echo` 根本意识不到文件描述符 `1` 发生了什么, 它只管向 `1` 中写东西


# Summary
- `system call` 接口相对是简单的, 但是在接口之下, 系统调用的实现是复杂的
- 简单的系统调用接口相互组合可以形成非常复杂的用例
