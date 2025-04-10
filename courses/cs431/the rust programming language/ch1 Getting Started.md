# 1.1 installation
The rust tool-chain would be installed in `~/.cargo/bin`, some important executables:
- `cargo`, rust package manager and build tool
- `rustfmt`, rust source file formator
- `rust-lldb, rust-gdb`, debugger
- `rustup`, for rust tool-chain (version) management
- `rustc`, rust compiler


# 1.2 hello world
Rust hello world program
```rust
fn main() {
    println!("Hello, World");
}

```

- `fn main` main function of this program, entrance of execution
- `println!`, a macro of rust which prints the content string to screen
	- `println` might be a normal function
	- Functions end up with `!` is a macro in rust, macros are slightly different from normal functions
	