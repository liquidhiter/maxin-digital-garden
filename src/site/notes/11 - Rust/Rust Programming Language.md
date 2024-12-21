---
{"dg-publish":true,"permalink":"/11-rust/rust-programming-language/","noteIcon":"","created":"2024-12-20T18:41:50.355+01:00","updated":"2024-12-21T12:25:26.932+01:00"}
---


#### online course
- bilibili: [2.1 - 猜数游戏：一次猜测_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hp4y1k7SV?spm_id_from=333.788.player.switch&vd_source=b4de78e2a4297863d5c19bd303c657b2&p=5)


#### quick learning 1
```rust
// use is used to bring a name into scope
use std::io::{self, Write};

// fn: function
fn main() {
    println!("Guessing game!");

    println!("Please input your guess.");

    // variables are immutable by default
    // #[cfg(feature = "debug")]
    // {
    //     let foo = 1;
    //     println!("foo: {}", foo);
    //     let foo = 2; // This will shadow the previous `foo` variable
    //     println!("foo: {}", foo);
    // }
    let mut foo = 1;
    let bar = foo;
    println!("foo: {}, bar: {}", foo, bar);
    foo = 2;
    println!("foo: {}", foo);

    // String is a growable, UTF-8 encoded bit of text
    // ::new is an associated function of the String type
    // :: is used for both associated functions and namespaces created by modules
    let mut guess = String::new();

    // io::stdout().flush().unwrap() is used to ensure the prompt is displayed before the user input
    std::io::stdout().flush().unwrap();

    // & indicates that this argument is a reference
    // & guess is immutable by default
    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");
    // iO::Result is an enum with variants Ok and Err

    // {} is a placeholder for the value of the variable
    println!("You guessed: {}", guess);
}

```

1. `prelude` 
2. variables are **immutable** by default
```rust
let foo = 1;
let bar = foo; // immutable by default
```
3. how to `#ifdef` the code block
```rust
    // variables are immutable by default
    #[cfg(feature = "debug")]
    {
        let foo = 1;
        println!("foo: {}", foo);
        let foo = 2; // This will shadow the previous `foo` variable
        println!("foo: {}", foo);
    }

```
4. `& guess` is immutable by default
5. `&` indicates that this argument is a reference

#### quick learning 2
```rust
use std::io;   // prelude
use rand::Rng; // trait

fn main() {
    println!("Hello, world!");

    let secret_number = rand::thread_rng().gen_range(1..101);

    println!("The secret number is: {}", secret_number);
}

```
1. crate
	- binary crate like the above code snippet compiled into a binary file
	- library crate which cant be executed directly
2. cargo.lock
	- record the versions of dependencies
	- ensure that same binary file can be generated with same source code and configurations
3. range: `1..100`, where `100` is exclusive
4. concepts of `prelude` and `trait`


#### quick learning 3
```rust
use std::{cmp::Ordering, io}; // prelude
use rand::Rng; // trait

// Prompt string
const PROMPT: &str = "Please input your guess:";

// Generate a random number between 1 and 100
fn gen_secret_number() -> u32 {
    return rand::thread_rng().gen_range(1..101);
}

// Print the input prompt
fn input_prompt() {
    println!("{}", PROMPT);
}   

// Read the user's guess from the console
fn read_guess() -> u32 {
    input_prompt();

    let mut guess = String::new();
    io::stdin().read_line(&mut guess).expect("Failed to read line");
    return guess.trim().parse().expect("Please type a number!");
}

// Compare the user's guess with the secret number
// Return true if the guess is correct, false otherwise
fn compare_guess(secret_number: u32, guess: u32) -> bool {
    match guess.cmp(&secret_number) {
        Ordering::Less => {
            println!("Too small!");
            false
        }
        Ordering::Greater => {
            println!("Too big!");
            false
        }
        Ordering::Equal => {
            println!("You win!");
            true
        }
    }
}

fn main() {
    println!("============= GUESS THE NUMBER =============");
    let secret_number = gen_secret_number();
    loop {
        let guess = read_guess();
        if compare_guess(secret_number, guess) {
            break;
        }
    }
}

```
> highlights
- import multiple preludes into the current scope `use std::{cmp::Ordering, io};`
- return value `-> u32`
- `match`
- `Ordering:Less =>` enum
- automatic deduction of the variable static types based on the context

#### quick learning 4
- variables
	- immutable by default
	- `mut` specifier needed to mutate the variable
- `i32` by default
	- type can be automatically inferred by rust
		- `let mut x: u32 = 5`
![Z - assets/images/Pasted image 20241220235558.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241220235558.png)
- constant
	- immutable after assignment
		- cant be described by `mut`
	- `const`
	- mandatory type specifier
- shadow
	- allow define (i.e. `let`) same variable name but different type
```rust
// Supress all warnings from the compiler
#![allow(warnings)]

// declare a constant
// const VAR_NAME: TYPE = VALUE;
const MAX_VAL: u32 = 100_000;
const MIN_TRY: i32 = 4;

fn main() {
    println!("Hello, world!");

    // declare a const variable in the function
    // shadowing is allowed
    const MAX_VAL: u32 = 100;

    let mut spaces = "   ";
    // spaces = spaces.len();
    // error[E0308]: mismatched types
    // expected `&str`, found `usize`
    spaces = "MAXIN";
    println!("spaces: {}", spaces);
    let spaces = spaces.len();
    println!("spaces: {}", spaces);

    let mut x: u32 = 5;
    // {} is a placeholder for the value of x
    // rust can automatically infer the type of x
    println!("The value of x is {}", x);
}

```
> highlights
- globally disable the warnings `#![allow(warnings)]`
![Z - assets/images/Pasted image 20241221002410.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221002410.png)
- static strong type
- ambiguous type infer requires explicitly type definition
```rust
    // error[E0284]: type annotations needed
    // compiler can't infer the type of y as it is ambiguous
    // expect method can return a value or panic
    let guess: u32 = match "42".parse() {
        Ok(num) => num,
        Err(_) => 0, // default value
    };
    println!("The value of guess is {}", guess);
```

#### quick learning 5
- 四种数据类型
	- 整数
	- 浮点数
	- 字符串
	- 布尔值
- 整数类型
![Z - assets/images/Pasted image 20241221100943.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221100943.png)
- 默认类型 `i32`
- 类型后缀
![Z - assets/images/Pasted image 20241221101315.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221101315.png)
- 整数溢出
	- 调试模式：整数溢出panic
	- release模式：整数溢出不会panic
		- truncation
- 浮点数
	- 默认`f64`
- 算术运算
	- operands相同类型(`u32` + `i32`非法)
	- 强制类型转换
		- `op2 as u32`
```rust
    let s: isize = 42;
    println!("The size of isize is: {} bytes", mem::size_of::<isize>());

    let f = 3.14; // f64
    println!("value of f: {}", f);

    let f1: f32 = 3.14; // f32
    println!("value of f1: {}", f1);

    /* Arithmetic operations */
    let op1: u32 = 5;
    let op2: i32 = 10;
    let sum = op1 + op2 as u32;
    // error[E0277]: cannot add `i32` to `u32`

    // let diff = op1 - op2 as u32; // overflow error
    let diff: i32 = op1 as i32 - op2;
    let op3 = 9.10; // f64
    let op4: f32 = 9.10; // f32
    let prod = op3 * op4;
```
- overflow errors
	- associated with types
		- `u32` overflow in the above example
```rust
    let op3 = 9.10; // f64
    let op4: f32 = 9.10; // f32
    let prod = op3 * op4;
```
> automatic infer the op3 to be f32 based on the context

- 字符类型
![Z - assets/images/Pasted image 20241221120700.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221120700.png)

- 复合类型
	- tuple
		- 多个类型的，多个值
		- 固定长度
	- 数组
		- 固定长度

- Tuple
```rust
    // Tuple type
    let tup: (i32, f64, u8) = (500, 6.4, 1);
    println!("The value of tup is: {:?}", tup);
    println!("The value of tup.0 is: {}", tup.0);

    // Can types in tuple be inferred?
    let tup1 = (500, 6.4, 1);
    println!("The value of tup1 is: {:?}", tup1);
```
- 获取Tuple的元素值
![Z - assets/images/Pasted image 20241221121321.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221121321.png)-
- 访问Tuple的元素
![Z - assets/images/Pasted image 20241221121351.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221121351.png)
- 数组
```rust
    // Array type
    let a = [1, 2, 3, 4, 5];
    println!("The value of a is: {:?}", a);
    // let b: [u32; 3] = [1, 2];
    // error[E0308]: mismatched types
    // expected an array with a fixed size of 3 elements, found one with 2 elements
    // println!("The value of b is: {:?}", b);
```
> array length should match with the number of elements
![Z - assets/images/Pasted image 20241221121712.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221121712.png)
![Z - assets/images/Pasted image 20241221121858.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221121858.png)