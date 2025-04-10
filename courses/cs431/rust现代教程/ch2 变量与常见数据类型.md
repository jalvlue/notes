# 1. 变量
- `let` 关键字用于声明变量
- Rust 支持类型推导, 但也可显式指定数据类型
	- `let y = 10;`, 将自动推导为 `i32` 类型
	- `let y: i32;`, 显式指定为 `i32` 类型, 但未初始化
- 在 rust 世界中, 命名规范:
	- 变量使用蛇形命名法 `snack_case`
	- 枚举和结构体使用帕斯卡命名法 `PascalCase`
	- 不需要的变量使用 `_` 加在变量名前占位, `_var_name`, 这样就不会报变量未使用
- 强制类型转换, 使用 `as` 关键字, 强制类型转换的规则 (截断, 填充等二进制位操作) 跟其他语言都类似
	- `let a = 3.1; let b = a as i32;`
- 变量打印
	- `println!("val: {}", x);`, 在字符传中嵌入占位符, 类似 `printf` 的写法, 但是没有指定类型
	- `println!("val: {x}");`, 直接将变量嵌入到字符串中, 新式写法
- Rust 的变量是默认不可变的
- 使用 `mut` 关键字让一个变量可变
	- `let mut y = 10; y = 20;`, 对可变变量的合法修改
- 变量不可变, 但是可以重新声明, 重新声明后, 原来的变量就会被销毁
	- `let y = 10; let mut y = "hello";`, 重新声明了变量 y

```rust
fn main() {
    // 不可变与命名
    let y: i32;
    y = 10;
    // y = 20;
    println!("{y}");
    let _nice_count = 100; // 自动推导i32
    let _nice_number: i64 = 54;
  
    // 声明可变
    let mut _count = 3;
    _count = 4;
  
    // Shadowing
    let x = 5;
    {
        // 命名空间
        let x = 10;
        println!("inner x : {}", x);
    } // 内部的x被销毁了
    println!("Outer x: {x}");
  
    // 在同一作用域下重新声明了x，最终覆盖了之前的x
    let x = "hello";
    println!("New x : {x}");
    // x = "hello";
  
    // 重新声明可以重定义类型 可变性
    let mut x = "this";
    println!("x: {x}");
    x = "that";
    println!("Mut x: {x}");
}
```


# 2. 常量与静态变量
- 常量使用 `const` 关键字声明定义, 必须是在编译期就确定的值, 不支持类型推导, 并且直接切入到生成的底层机器代码中
- 常量的生命周期在作用域内
- 静态变量使用 `static` 关键字声明定义, 在运行时才分配内存, 可以加 `mut`, 然后使用 `unsafe` 对静态变量进行修改 (只能使用 `unsafe` 进行操作, 在 `safe` 下无法对该变量进行任何操作)
- 静态变量的生命周期是整个程序的运行时间
- 常量和静态变量命名规范:
	- 推荐全部大写, 单词之间使用下划线隔开
	- `const A_CONST_VAL: i32 = 100;`
	- `static mut MY_MUT_STATIC: i32 = 42;`

```rust
static MY_STATIC: i32 = 42;
static mut MY_MUT_STATIC: i32 = 42;
  
fn main() {
    // const
    const SECOND_HOUR: usize = 3_600;
    const SECOND_DAY: usize = 24 * SECOND_HOUR; // compile-time constant
  
    {
        const SE: usize = 1_000;
        println!("{SE}");
    }
  
    // println!("{SE}");
    println!("{}", SECOND_DAY);
    // MY_STATIC = 2;
    println!("{MY_STATIC}");
  
    // println!("{MY_MUT_STATIC}");
    unsafe {
        MY_MUT_STATIC = 32;
        println!("{MY_MUT_STATIC}");
    }
    // println!("{MY_MUT_STATIC}");
}
```

# 3. 基础数据类型
- 整型, `i8, i16, i32, i64, i128`, 默认推导为 `i32`
- 无符号整型, `u8, u16, ...`
- 浮点型, `f32, f64`, 默认推导为 `i64`
- 由平台决定的类型
	- `usize`
	- `isize`
- 布尔型, `true, false`
- 字符型, `'c'`, 支持 Unicode

```rust
fn main() {
    // 进制的字面量
    let a1 = -125;
    let a2 = 0xFF;
    let a3 = 0o13;
    let a4 = 0b10;
    println!("{a1} {a2} {a3} {a4}");
    // Max Min
    println!("u32 max: {}", u32::MAX);
    println!("u32 min: {}", u32::MIN);
    println!("i32 max: {}", i32::MAX);
    println!("i32 min: {}", i32::MIN);
    println!("usize max: {}", usize::MAX);
    println!("isize is {} bytes", std::mem::size_of::<isize>());
    println!("usize is {} bytes", std::mem::size_of::<usize>());
    println!("u64 is {} bytes", std::mem::size_of::<u64>());
    println!("i64 is {} bytes", std::mem::size_of::<i64>());
    println!("i32 is {} bytes", std::mem::size_of::<i32>());
  
    // float
    let f1: f32 = 1.23234;
    let f2: f64 = 9.88888;
    let f3 = 9.9;
    // 带一点格式化的输出, 这里表示整数部分全部, 小数部分前面两位
    println!("Float are {:.2} {:.2}", f1, f2);
    println!("f3: {f3}");
  
    // bool
    let is_ok = true;
    let can_ok: bool = false;
    println!("is ok? {is_ok} can ok? {can_ok}");
    println!(
        "is ok or can ok ?{}, can ok and is ok? {}",
        is_ok || can_ok,
        is_ok && can_ok
    );
    // char
    let char_c = 'C';
    let emo_char = '😀';
    println!("You Get {char_c} feel {emo_char}");
    println!("{}", emo_char as usize);
    println!("{}", emo_char as i32)

}
```


# 4. 元组和数组
- 元组和数组都是复合类型, `compound types`, 长度都是固定 (长度也是类型的一部分, 跟 go 类似)
- Tupes, 不同数据类型的元素集合, 默认就是可变的, 不需要加 `mut` 关键字也能直接修改元组内的元素, 没有 ` len() ` 方法, 通过 ` tup.0, tup.1 ` 访问元组内的元素
- Arrays, 同一数据类型的元素集合, 有 `len()` 方法获取数组长度, 默认是不可变的通过加 `mut` 关键字让数组可变, 通过下标表达式获取数组元素, ` arr[0], arr[1] `
- 声明定义方式
	- 元组: `let tup = (0, "hi", 3.4); tup.1 = "f";`
	- 数组:
		- `let mut arr = [11, 12, 34]; arr[0] = 999;`
		- `let mut ar = [2; 3];`, 也可以通过 `[初始值; 长度]` 的方式声明定义数组
- 数组和元组都是赋值时复制的类型 (==??? 那字符串数组怎么说==), 区别于 string 或者其他复杂的结构体是交出所有权

```rust
fn main() {
    // tuple
    let tup = (0, "hi", 3.4);
    println!("tup elements {} {} {}", tup.0, tup.1, tup.2);
  
    let mut tup2 = (0, "hi", 3.4);
    println!("tup2 elements {} {} {}", tup2.0, tup2.1, tup2.2);
    tup2.1 = "f";
    // tup2.1 = 1;
    println!("tup2 elements {} {} {}", tup2.0, tup2.1, tup2.2);
  
    // ()
    let tup3 = ();
    println!("tup3 {:?}", tup3);
    // println!("tup3 {}", tup3);
  
    println!("Array");
  
    let mut arr = [11, 12, 34];
    arr[0] = 999;
    println!("arr len {} first element is {}", arr.len(), arr[0]);
  
    for elem in arr {
        println!(" {}", elem);
    }
    let ar = [2; 3];
    for i in ar {
        println!("{}", i);
    }
  
    // ownership
    let arr_item = [1, 2, 3];
    let tup_item = (2, "ff");
    println!("arr: {:?}", arr_item);
    println!("tup: {:?}", tup_item);
    // clone
    let arr_ownership = arr_item;
    let tup_ownership = tup_item;
    // arr_item and arr_ownership is two individual array
    println!("arr: {:?}", arr_item);
    println!("tup: {:?}", tup_item);
  
    // clone
    // copy
    let a = 3;
    let b = a;
    println!("{a}");
  
    // string assignment would move the ownership
    let string_item = String::from("aa");
    let string_item_tt = string_item; // String 类型就把Ownership 进行move操作
                                      // println!("{string_item}"); // value borrowed here after move
}
```


