# 1. if 流程控制和 match 模式匹配
- 控制流
	- if
	- match

```rust
fn main() {
    let age = 50;
    if age < 50 {
        println!("You are young");
    } else {
        println!("You are old");
    }
  
    // if 的表达能力很弱
    let scores = 70;
    if scores > 90 {
        println!("Good!!!");
    } else if scores > 60 {
        println!("You are OK!");
    } else {
        println!("Bad!!!");
    }
  
    // 三元表达式
    let msg = if age > 50 { "old" } else { "young" };
    println!("You are {msg}");
  
    // match
    let num = 90;
    // 精准匹配
    match num {
        80 => println!("80"),
        90 => println!("90"),
        _ => println!("Some else"),
    }
  
    // 范围匹配
    match num {
        25..=50 => println!("25 ... 50"),
        51..=100 => println!("51 ... 100"),
        _ => println!("Some else"),
    }
  
    // 或精准匹配
    match num {
        25 | 50 | 75 => print!("25 or 50 or 75"),
        100 | 200 => println!("100 or 200"),
        _ => println!("Some else"),
    }
  
    // 嵌套 if
    match num {
        x if x < 60 => println!("bad"),
        x if x == 60 => println!("luck"),
        _ => println!("Some else"),
    }
    let num = 60;
    let res = match num {
        x if x < 60 => "bad".to_owned(),
        y if y == 60 => "luck".to_owned(),
        _ => "Some else".to_owned(),
    };
    println!("res value : {res}");
}
```

# 2. 循环, break, continue 与迭代
