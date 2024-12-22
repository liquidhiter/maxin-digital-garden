---
{"dg-publish":true,"permalink":"/11-rust/rust-programming-language/","noteIcon":"","created":"2024-12-20T18:41:50.355+01:00","updated":"2024-12-22T15:44:07.762+01:00"}
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

- vector
![Z - assets/images/Pasted image 20241221121712.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221121712.png)
- `let var: [type, length] = [];`
![Z - assets/images/Pasted image 20241221121858.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221121858.png)
- all elements have the same values
![Z - assets/images/Pasted image 20241221122755.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221122755.png)
- 使用索引来访问数组中的元素
	- runtime error: index out of bound
```rust
    // same value for all elements
    let c = [3; 5];
    println!("The value of c is: {:?}", c);
    let first = c[0];
    let second = c[1];
    // let six = c[5];
    // error: index out of bounds, compilation error
    // let index = [0, 1, 2, 3, 4, 5];
    // let six = c[index[5]];
    // error: index out of bounds, compilation error
    println!("The value of first is: {}, second is: {}", first, second);
```

#### quick learning 6
- function
	- `fn`
![Z - assets/images/Pasted image 20241221123654.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221123654.png)
- 函数的返回值
![Z - assets/images/Pasted image 20241221140308.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221140308.png)
- expression, statement
```rust
// function with return value
// -> i32: specify the return type
fn five() -> i32 {
    // last expression is the return value
    5
}

fn sive(x: u32) -> u32 {
    x + 6
}

```
- no implicit conversion between types
```rust
    let val = five();
    println!("The value of five is: {}", val);

    let val_1: u32 = sive(val as u32);
    println!("The value of sive is: {}", val_1);
```
- `if-else`
```rust
// check whether the input is a prime number
fn function(num: u32) -> bool {
    if num < 2 {
        // early return if the input is less than 2
        return false;
    } else {
        for i in 2..num {
            if num % i == 0 {
                return false;
            }
        }
        return true;
    }
}

fn is_even(num: u32) -> bool {
    match num % 2 {
        0 => true,
        _ => false,
    }
}
```
> condition must be boolean, no implicit conversion to boolean types
- loop / while / for
![Z - assets/images/Pasted image 20241221141717.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221141717.png)![Z - assets/images/Pasted image 20241221141746.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221141746.png)

#### quick learning 7
- 核心特性：所有权
	- Rust不使用GC
![Z - assets/images/Pasted image 20241221145650.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221145650.png)
- 内存安全检查
	- 编译时进行
- reference: [What is Ownership? - The Rust Programming Language](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#)
- ownership rules
```markdown
- Each value in Rust has an _owner_.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.
```
- `drop` function is automatically invoked when the variable goes out of scope
- moving semantics
```rust
    let mut s1: String = String::from("complex_data");
    println!("address of the variable s1 is {:?}", s1.as_ptr());
    let mut s2: String = s1;
    // borrow of moved value ???
    println!("address of the variable s2 is {:?}", s2.as_ptr());
    s2.push_str("OSLO");
```
> after `let mut s2: string = s1`, string s1 is considered as invalid
> WHY designing in this way?
- solving the issue of **freeing the same blob of memory twice**
> **Rust will never automatically create deep copies of data**

```rust
let mut s = String::from("hello"); 
s = String::from("ahoy");
```
- Rust also calls `drop` when assigning a completely new value to an existing variable
- `hello` goes out of scope and Rust free the associated heap memory
```rust
    // clone
    let s1: String = String::from("clone");
    // stack and heap data are copied
    let s2: String = s1.clone();
    println!("s1 = {}, s2 = {}", s1, s2);
```
- stack and heap data are copied
- `Copy` trait for types stored on the stack
- complex types
	- Tuple only contains types that also implements `Copy`
```markdown
Here are some of the types that implement `Copy`:

- All the integer types, such as `u32`.
- The Boolean type, `bool`, with values `true` and `false`.
- All the floating-point types, such as `f64`.
- The character type, `char`.
- Tuples, if they only contain types that also implement `Copy`. For example, `(i32, i32)` implements `Copy`, but `(i32, String)` does not.
```
- ownerships of function parameters / arguments
	- moved and get back by explicitly returning
```rust
fn main() {
    var_scope();
}

// understand the scope of variables
fn var_scope() {
    // ownership: simple data types
    // simple data types are stored in stack
    {
        let s: &str = "hello world";
        println!("{}", s);
        println!("address of the variable s is {:?}", s.as_ptr());
    }

    // s is not available here
    let s: &str = "hello rust";
    println!("{}", s);

    // ownership: complex data types
    let s: String = String::from("string_literal");
    println!("Address of the string variable s is {:?}", s.as_ptr());
    println!("Address of the String object s is {:p}", &s);

    // manipualte the string
    let mut s = String::from("hello");
    s.push_str("RUST");
    println!("{}", s);

    // Primitive data types
    // Value is copied instead of moving
    let mut x = 5;
    let y = x;
    println!("x = {}, y = {}", x, y);
    // change the value of x
    x = 10;
    println!("x = {}, y = {}", x, y);

    let mut s1: String = String::from("complex_data");
    println!("address of the variable s1 is {:?}", s1.as_ptr());
    let mut s2: String = s1;
    // borrow of moved value ???
    println!("address of the variable s2 is {:?}", s2.as_ptr());
    s2.push_str("OSLO");
    // String s1 is not available here as it is moved to s2

    // clone
    let s1: String = String::from("clone");
    // stack and heap data are copied
    let s2: String = s1.clone();
    println!("s1 = {}, s2 = {}", s1, s2);

    // ownership and functions
    let s: String = String::from("ownership_moved");
    take_ownership(s);

    // println!("{}", s);
    // ownership is moved to the function take_ownership, so s is not available here
    let s: String = String::from("ownership_back");
    let str_back: String = take_and_give_back(s);
    println!("{}", str_back);
}

fn investigate_string(s: String) -> String {
    println!("Address of the String object s is {:p}", &s);
    println!("Address of the variable s is {:?}", s.as_ptr());
    println!("{}", s);
    s
}

fn take_ownership(s: String) {
    // ownership is moved to the function take_ownership
    investigate_string(s);
}

fn take_and_give_back(s: String) -> String {
    let str_back: String = investigate_string(s);
    match str_back.len() {
        0 => String::from("empty"),
        _ => str_back,
    }
}

```
> ownerships moved for variables can't be taken back
- potential cons
	- pass arguments and return back even when only the value (heap data???) is needed
	- solution
		- references
- return multiple values
![Z - assets/images/Pasted image 20241221171314.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241221171314.png)
- references
	- an address which can be followed to access the data at that address, owned by other variables
	- guaranteed to point to a valid value of a particular type for the of that reference
	- Rust calls the action of creating reference `borrowing`
```rust
fn calculate_length(s: &String) -> usize {
    // s.push_str(", oops");
    // error[E0596]: cannot borrow *s as mutable as it is behind a & reference
    s.len()
}

fn insert_string(s: &mut String) {
    s.push_str(", oops");
}

let s_n: String = String::from("rust_king");
let len: usize = calculate_length(&s_n);
println!("The length of the given string is {}", len);

let mut s_m: String = String::from("rust_king");
insert_string(&mut s_m);
println!("{}", s_m);
```
- avoid data race by preventing multiple mutable references to the same data at the same time
- not allowed to have a mutable reference while we have an immutable one to the same value
```rust

    // multiple mutable references (error)
    // let mut s_p: String = String::from("rust_king");
    // let r1 = &mut s_p;
    // let r2 = &mut s_p;
    // println!("{}, {}", r1, r2);
    // error[E0499]: cannot borrow `s_p` as mutable more than once at a time

    // use scope to fix the above error
    let mut s_p: String = String::from("rust_king");
    {
        let r1 = &mut s_p;
        println!("{}", r1);
    }
    let r2 = &mut s_p;
    println!("{}", r2);

	// mutable and immutable references
    // let mut s_p: String = String::from("rust_king");
    // let r1 = &s_p; // immutable reference
    // let r2 = &s_p; // immutable reference
    // let r3 = &mut s_p; // mutable reference
    // println!("{}, {}, and {}", r1, r2, r3);
    // error[E0502]: cannot borrow `s_p` as mutable because it is also borrowed as immutable
    // it is allowed to create mutable and immutable references at the same time
    // BUT they should never be used at the same time
```
> Note that a reference’s scope starts from where it is introduced and continues through the last time that reference is used.
- ~~my understanding is reference goes out of scope after being used, that's why the following code can be compiled~~ 
```rust
    // the code below throws error
    let mut s_p: String = String::from("rust_king");
    let r1 = &s_p; // immutable reference
    let r2 = &s_p; // immutable reference
    let r3 = &mut s_p; // mutable reference
    println!("{}, {}", r1, r2);
    
    // reference's scope
    let mut s_p: String = String::from("rust_king");
    let r1 = &s_p;
    let r2 = &s_p;
    println!("{}, {}", r1, r2); // immutable reference
    let r3 = &mut s_p;
    println!("{}", r3); // mutable reference
```
> whether the scopes overlap with each other
- dangling references
	- *In Rust, by contrast, the compiler guarantees that references will never be dangling references: if you have a reference to some data, the compiler will ensure that the data will not go out of scope before the reference to the data does.*
```rust
// dangling reference
// local variable s goes out of scope and it is dropped
// the reference to the local variable is lost and it is a dangling reference
fn dangle_ref() -> &String {
    let s: String = String::from("dangle_ref");
    &s
}
```
- slice
![Z - assets/images/Pasted image 20241222133732.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241222133732.png)
- string literal is slice
- pass slice as function arguments
![Z - assets/images/Pasted image 20241222133832.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241222133832.png)
- struct
	- all fields are mutable if struct is mutable
	- field and parameter names are allowed to be the same
![Z - assets/images/Pasted image 20241222154338.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241222154338.png)