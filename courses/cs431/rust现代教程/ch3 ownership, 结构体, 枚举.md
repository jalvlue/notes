# 1. rust 内存管理模型
- 所有权系统 `ownership`
- 借用 `borrowing`, 也就是通过引用来访问变量, 而不是获取所有权或转让所有权

当你使用不可变借用（immutable borrowing）时，你可以通过 `&T` 来创建一个不可变的引用，允许你以只读方式访问值，但不能对其进行修改。这允许多个不可变引用同时存在。
下面是一个不可变借用的例子：
```rust
fn main() {
    let x = 5;
    let y = &x; // 不可变借用 x，创建一个不可变引用 y

    println!("x: {}", x);
    println!("y: {}", *y); // 通过解引用访问 y 所指向的值

    // x 和 y 在这里仍然可用
}
```
在这个例子中，变量 `x` 拥有整数值 5，然后我们创建了一个不可变引用 `y` 来借用 `x`。我们可以通过 `*y` 解引用 `y` 并访问其所指向的值。

而当你使用可变借用（mutable borrowing）时，你可以通过 `&mut T` 来创建一个可变的引用，允许你对值进行读写操作。但是，在特定作用域中只能存在一个可变引用，并且在该引用存在期间，不能有其他引用（无论是可变引用还是不可变引用）存在。
下面是一个可变借用的例子：
```rust
fn main() {
    let mut x = 5;
    let y = &mut x; // 可变借用 x，创建一个可变引用 y

    *y += 1; // 修改 y 所指向的值

    println!("x: {}", x);
    println!("y: {}", *y);

    // x 和 y 在这里仍然可用
}
```

在这个例子中，我们将变量 `x` 声明为可变的，然后创建了一个可变引用 `y` 来借用 `x`。通过 `*y` 解引用 `y` 并修改其所指向的值。注意，在可变引用存在期间，我们不能同时存在其他引用。
这些示例展示了不可变借用和可变借用的基本用法，它们允许你在不获取所有权的情况下对值进行读取或读写操作，同时遵循 Rust 的借用规则来保证内存安全。

```rust
fn main() {
    // for simple data types, assign would be a copy
    let c1 = 1;
    let c2 = c1;
    println!("{}", c1);
  
    // for complicated data types like string and struct, assign would move the ownership
    let s1 = String::from("value");
    let s2 = s1; // s1 的所有权转移给s2
                 //  println!("{s1}"); //  value borrowed here after move
  
    // use `clone()` to copy instead of move the ownership
    let s1 = String::from("s1");
    let s2 = s1.clone();
    println!("{}", s1);
  
    let len = get_length(s1);
    println!("{}", len);
    // the ownership of s1 have been moved to get_length and therefore destroyed
    // println!("{s1}");
  
    let s1 = String::from("s1");
    // pass by reference
    let len = get_length_ref(&s1);
    println!("{s1}");
  
    let s = "hello";
  
    let back = first_word("hello world");
    println!("{}", back);
    let back = first_word("we are the world");
    println!("{}", back);
}
  
fn get_length(s: String) -> usize {
    println!("String: {}", s);
    // 函数结束之后 main::s1也销毁了
    s.len()
}
  
fn get_length_ref(s: &str) -> usize {
    println!("String: {}", s);
    s.len()
}
  
fn dangle() -> String {
    "hello".to_owned()
}
  
// 静态的声明周期
fn dangle_static() -> &'static str {
    "jdkfj"
}
  
// String 与 &str vec u8 ref
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]

}
```


# 2. String 和 &str
- Rust 中的表示字符串的主要有两种类型
- String 是一个堆分配的可变字符串类型, 实际上就是一个 `u8` 类型的可变字符串
```rust
pub struct String {
	vec: Vec<u8>,
}
```

- &str 是字符串字面量, 也就是字符串切片引用, 也就类似是一个指针, 在栈上分配, 指向存储在其他地方的 utf-8 编码的字符串数据
- 参数传递:
	- 函数参数为 `&str` 类型时, 可以传 `&String, &str`
	- 函数参数为 `&String` 时, 只能传 `&String`

```rust
struct Person<'a> {
    name: &'a str,
    color: String,
    age: i32,
}
  
// &String and &str are both ok
fn print(data: &str) {
    println!("{}", data);
}
  
// &String only
fn print_string_borrow(data: &String) {
    println!("{}", data);
}
  
fn main() {
    // String &str conversion
    let name = String::from("Value C++");
    // String::from
    // to_string()
    // to_owned()
    let course = "Rust".to_string();
    let new_name = name.replace("C++", "CPP");
    println!("{name} {course} {new_name}");
    let rust = "\x52\x75\x73\x74"; // ascii
    println!("{rust}");
  
    // String &str
    let color = "green".to_string();
    let name = "John";
    let people = Person {
        name: name,
        color: color,
        age: 89,
    };
    // func
    let value = "value".to_owned();
    print(&value);
    print("data");
    // print_string_borrow("dd");
    print_string_borrow(&value);
}
```

# 3. 枚举和匹配模式
- 枚举就是表示离散值的用户自定义数据类型, 每种可能的值称为 `variant`, 变体
- 判断一个枚举类型时, 可以使用 `match` 语句进行匹配, 需要 cover 该枚举类型的所有变体
- 枚举类型的变体中还可以内嵌一些其他类型的数据作为补充
- 话不多说直接看代码

```rust
enum Color {
    Red,
    Yellow,
    Blue,
    // Black,
}
  
fn print_color(my_color: Color) {
    // 使用 match 来对枚举类型进行匹配的时候, 必须 cover 所有的枚举变体
    match my_color {
        Color::Red => println!("Red"),
        Color::Yellow => println!("Yellow"),
        Color::Blue => println!("Blue"),
        // Color::Black => println!("Black"),
    }
}
  
enum BuildingLocation {
    Number(i32),
    Name(String), // 不用&str
    Unknown,
}
  
impl BuildingLocation {
    fn print_location(&self) {
        match self {
            // BuildingLocation::Number(44)
            BuildingLocation::Number(c) => println!("building number {}", c),
  
            // BuildingLocation::Name("ok".to_string())
            BuildingLocation::Name(s) => println!("building name {}", *s),
  
            BuildingLocation::Unknown => println!("unknown"),
        }
    }
}
  
fn main() {
    let a = Color::Red;
    print_color(a);
    // let b = a;
  
    let house = BuildingLocation::Name("fdfd".to_string());
    house.print_location();
  
    let house = BuildingLocation::Number(1);
    house.print_location();
  
    let house = BuildingLocation::Unknown;
    house.print_location();
}
```


# 4. 结构体, 方法, 关联函数, 关联变量
- 方法就是跟结构体实例相关联的函数 (类方法)
- 关联函数就是跟结构体这个类型相关联的函数 (类静态函数)
- 关联变量就是跟结构体这个类型相关联的变量 (类静态变量)

```rust
enum Flavor {
    Spicy,
    Sweet,
    Fruity,
}
  
struct Drink {
    flavor: Flavor,
    price: f64,
}
  
impl Drink {
    const HELLO_DRINK: &str = "DRINK";
}
  
impl Drink {
    // 关联变量
    const MAX_PRICE: f64 = 10.0;
  
    // 方法
    fn buy(&self) {
        if self.price > Drink::MAX_PRICE {
            println!("I am poor");
            return;
        }
        println!("buy it");
    }
  
    // 关联函数
    fn new(price: f64) -> Drink {
        Drink {
            flavor: Flavor::Fruity,
            price,
        }
    }
}
  
fn print_drink(drink: Drink) {
    match drink.flavor {
        Flavor::Fruity => println!("fruity"),
        Flavor::Spicy => println!("spicy"),
        Flavor::Sweet => println!("sweet"),
    }
    println!("{}", drink.price);
}
  
fn main() {
    let coke = Drink {
        flavor: Flavor::Sweet,
        price: 6.0,
    };
    println!("{}", coke.price);
    print_drink(coke);
    let sweet = Drink::new(12.0);
    sweet.buy();
}
```


# 5. ownership 和结构体
- 所有值都有且只有一个所有者 `owner`
- 当 `owner` 离开作用域了, 这个值就会被销毁
- 当值从一个位置传递到另外一个位置的时候, `borrow checker` 都会重新评估所有权
	- `immutable borrow`, 不可变借用, 可以理解为传递常量引用, 所有权还是属于原来的所有者, 只是借用者可以通过这个引用来访问该值, 可以有多个不可变借用
	- `mutable borrow`, 可变借用, 可以理解为传递可修改的引用, 为了并发写安全, rust 只允许有一个唯一的可变借用
	- `move`, 所有权转移, 原来的所有者无法再访问该值

```rust
struct Counter {
    number: i32,
}
  
impl Counter {
    fn new(number: i32) -> Self {
        Self { number }
    }
  
    // 不可变借用
    // Counter::get_number(self: &Self)
    fn get_number(self: &Self) -> i32 {
        self.number
    }
  
    // 可变借用
    // Counter::add(self: &mut Self, increment:)
    fn add(self: &mut Counter, increment: i32) {
        self.number += increment;
    }
  
    // move
    fn give_up(self: Counter) {
        println!("free {}", self.number);
    }
  
    // move
    fn combine(c1: Self, c2: Self) -> Self {
        Self {
            number: c1.number + c2.number,
        }
    }
}
  
fn main() {
    let mut c1 = Counter::new(0);
    println!("c1 number {}", c1.get_number());
    println!("c1 number {}", c1.get_number());
    c1.add(2);
    println!("c1 number {}", c1.get_number());
    println!("c1 number {}", c1.get_number());
    c1.give_up();
    // println!("c1 number {}", c1.get_number());
  
    let c1 = Counter::new(2);
    let c2 = Counter::new(1);
    let c3 = Counter::combine(c1, c2);
    // println!("c1 number {}", c1.get_number());
    // println!("c2 number {}", c2.get_number());
  
    println!("c3 number {}", c3.get_number());
}
```


# 6. 堆和栈, copy 和 move
- 对栈上的数据通常可以直接访问
- 对于堆上的数据通常需要通过引用访问
- Box 是一个智能指针, 提供对堆分配内存的的所有权
- 默认保存在栈上
	- 基础数据类型
	- tuple 和 array
	- 只包含基础数据类型的 struct 和 enum
- 默认保存在堆上
	- Box
	- Rc
	- String
	- Vec
- 一般来说, 栈上的数据类型都默认实现了 copy 特质, 赋值时是直接复制
- 堆上保存的数据类型一般都是 move, 可以通过实现 copy 特质, 或者实现 clone 特质来进行复制

Box
```rust
struct Point {
    x: i32,
    y: i32,
}
  
fn main() {
    let boxed_point = Box::new(Point { x: 10, y: 20 });
    println!("x:{}, y:{}", boxed_point.x, boxed_point.y);
  
    let mut boxed_point = Box::new(32);
    println!("{}", *boxed_point);
    *boxed_point += 10;
    println!("{}", *boxed_point);
}
```

copy and clone
```rust
#[derive(Debug, Clone, Copy)]
struct Book {
    page: i32,
    rating: f64,
}
  
fn main() {
    let x = vec![1, 2, 3, 4];
    let y = x.clone();
    println!("{:?}", y);
    println!("{:?}", x);
  
    let x = "ss".to_string();
    let y = x.clone();
    println!("{:?}", x);
  
    let b1 = Book {
        page: 1,
        rating: 0.1,
    };
    let b2 = b1; // move
    println!("{:?}", b1);
}
```


