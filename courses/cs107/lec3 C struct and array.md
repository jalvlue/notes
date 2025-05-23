在 C 中, 结构和数组都是指向起点的指针, 可以对一个地址解释为不同类型(解释不同的长度)
```c
int arr[10];
arr === &arr[0]
arr+4 === &arr[4]
```

### 数组和结构体
- 数组使用方括号[]表示,是同类型元素的集合,使用下标访问元素
- 结构体使用struct定义一个自定义的数据类型,包含不同类型的成员。结构体变量分配的内存,等于其所有成员大小之和
    - 结构体变量会连续打包所有成员占用的内存,第一个成员的地址等于整个结构体的地址
    - 可以使用.访问结构体成员
    - 取结构体指针, 指针指向整个结构体, 通过->访问成员

```c
struct fraction { int Num; int Den; };
```

### 指针与引用
- 指针使用\*取地址操作符获取变量地址,&取值操作符解引用指针获取实际值
    - 例如:
```c
int a = 10;
int *p = &a; // p指向a的地址
```
        
- 引用是另一个名字绑定到同一个变量。初始化后同名意思
    - 例如:    
```c
int a = 10;
int &b = a; // b和a引用同一个变量
```

- 取地址与引用可以作用在结构体上,获取结构体整体或成员的地址
- 可以通过移位操作符\*和->从指针访问结构体成员
- 直接取值与引用值没有区别,都获得相同的实际值
- 不当移动内存会导致非法访问和崩溃问题

### 数组与结构体间的转换

- 可以将结构体强制转换成其他类型来访问内存
    
    - 例如将结构体转换成字符型取整个结构体的地址
        
```c
struct Test *p = &test;
char *pc = (char*)p;
```
        
- 也可以通过指针arithmatic移动指针内存地址来解引用不同类型
    
    - 例如通过字符串指针偏移解引用结构体
        
```c
struct Test *p = &test;
char *pc = (char*)p;
struct Test *pt = (struct Test*)(pc + sizeof(int));
```
        
- 这类操作存在安全隐患,只有在确定内存布局情况下才可以这样做
    
- 一般通过结构体和数组的定义来正确访问他们之间的转换
