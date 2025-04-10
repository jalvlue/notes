# Lecture



# K&R

### 2.9 Bitwise Operators
六种位操作符, 一般情况下只能作用于整数(char short int long)

![[Pasted image 20240430173954.png]]


### 5.1 Pointers and Addresses
对象被存储在内存中, 拥有自己的内存地址
- 变量
- 数组

指针中保存着对象的地址, 
- 通过 `&` 取地址
- 通过 `*` 解引用地址

对象可以怎样出现, `*p` 也可以在同样的场景出现

同时指针也是变量, 
可以独立出现运算

### 5.2 Pointers and Function Arguments
C 语言的函数参数都是传值的, 被调用函数只能获得一份参数副本, 因此不能在被调用函数内修改其外的内容
```c
void swap(int x, int y) /* WRONG */\
{
	int temp;

	temp = x;
	x = y;
	y = temp;
}
```
这是完全错误的, 参数只传值, 无法修改实际传入参数的内容

此时就可以使用指针捕获被调用函数之外的变量

![[Pasted image 20240430180349.png]]
```c
void swap(int *x, int *y)
{
	int temp;

	temp = *x;
	*x = *y;
	*y = temp;
}
```

由于传指针可以操作被调用函数之外的变量, 因此一个函数除了使用返回值, 还可以使用传指针参数对外部变量进行影响

例如这个 `int getint(int *pn)` 函数, 将从输入获得的整数存进函数参数中
```c
int getint(int *pn)
{
	int c, sign;

	while (isspace(c = getch()))
		;

	if (!isdigit(c) && c != EOF && c != '+' && c != '-') {
		ungetch(c);
		return 0;
	}

	sign = (c == '-') ? -1 : 1;
	if (c == '+' || c == '-')
		c = getch();
	for (*pn = 0; isdigit(c); c = getch()) 
		*pn = 10 * *pn + (c - '0')
	*pn *= sign
	if (c != EOF)
		ungetch(c);
	return c;
}
```

### 5.3 Pointers and Arrays
指针和数组关系紧密
```c
int a[10];
int *pa;
pa = &a[0];
// pa = a;
```
![[Pasted image 20240430182122.png]]
可以使用指针指向某个数组变量

同时指针可以做运算, 这个运算不论数组元素是什么类型都成立, 
![[Pasted image 20240430182229.png]]

从上面注释中可以看到, 实际上数组 `a` 的地址就是它第一个元素的地址
```c
pa = &a[0];
pa = a;
```
是完全等价的, 实际上, 使用 `a[i]` 获取数组元素时, 会转化为 `*(a+i)`, 也就是指针运算, 然后解引用
唯一的区别是:
- pa 是变量, 可以对它进行所有变量操作
- a 不是变量, 不能进行变量操作

```c
/* strlen: return length of string s */  
int strlen(char *s)  
{
	int n;  
	for (n = 0; *s != '\0', s++)  
		n++;  
	return n;  
}

strlen("hello, world"); /* string constant */  
strlen(array);  /* char array[100]; */  
strlen(ptr);  /* char *ptr; */
```
这个版本的函数参数是一个指针, 由于数组名就是一个地址, 因此完全可以作为参数, 

同时, 函数参数
```c
char s[];
char *s;
```
是完全等价的, 但是后者更好一些, 更明确是一个指针, 是地址


### 5.4 Address Arithmetic
在上面提到, 对指针和一个整数 n 进行加减, 那么指针将指向 n 个元素之后的的那个元素, 
同样的, 如果有两个指针, `pa, pb`, 均在同一个数组中, 那么 `pb-pa+1` 则可以得到一个整数, 表示两个数组之间一共有多少个元素

这里有另一个版本的 `strlen` 利用了这个性质
```c
int strlen(char *s)
{
	char *p = s;

	while (*p != '\0')
		p++;
	return p - s;
}
```

同时指针还可以进行比较
- >
- <
- ==

0 指针是一个特殊的指针, 内存中没有这个地址, 因此它被作为指针指向一个不存在的对象时的值, NULL 是 0 指针的助记符

### 5.5 Character Pointers and Functions
```c
char amessage[] = "now is the time";
char *pmessage = "now is the time";
```
![[Pasted image 20240430185742.png]]

这里区分了字符数组和字符串常量, 
字符串常量是匿名的, 不可修改的, 只能由一个指针指向它


### 5.6 Pointer Arrays; Pointers to Pointers
指针作为变量, 也可以成数组, 
同时由于组成了数组, 因此也可以有新的, 指向指针的指针来对这个指针数组进行操作
![[Pasted image 20240430190304.png]]


### 6.4 Pointers to Structures
