

#                                                                                    目录

[TOC]

**导读**

本书面向的读者，最好有C/C++基础，了解现代C++的各种概念。如果没有C/C++基础也没有关系，Rust远没有C/C++那么复杂。

重点章节：14，15，16 这个是Rust语言的特色部分，也是难点部分，必须要掌握，否则在阅读源码或者使用库的时候，会产生很多疑惑。

我尽力用最简单的语言将原作者要表达的语义表达清晰、准确。但我在注解的过程中，也发现，有些地方，我怎么解释也讲不清楚。这可能本身就是语言的复杂信。英文的表达比较简练，加上必须对Rust有深入的了解，才能有更好的注解。虽然作者对现代C++语言知识比较有自信，但是我也深知，对语言的真正理解来自工程实践，对Rust的了解可能存在盲点，有些注解不够完善，还请读者指正，我会虚心接受改善，为初学者提供更好的入门教材。

## **目标**

提供快速、准确、简单的入门方式，快速建立完整的Rust知识体系。

## **价值**

基础知识，但并不代表他不重要，高级的功能都依赖与我们对基础知识的准确把握和深入理解。

学习一门计算机语言，我认为不是在编写过程中，有了问题去百度、google，最好的方式是在下手coding前，花时间把官网的教程学习一遍，以便减少犯低级错误。

官网的文档有如下的优点：

1. 写这些文档的人都是大牛或者语言本身的创造者，他们对语言的特性的阐述和表达准确性是最高的。
2. 文档经过多次的修改和完善，去除了错误的、过时的信息，避免误导读者。
3. 知识体系完整，上下文知识关联，对于复杂的、需要静一步完善的知识，给出权威的参考链接。
4. 讲解与代码相结合，完整的表达了作者意图，通过错误例子编译，展示出错信息，加强了读者对知识点的理解。

缺点：

1. 文档大多数是英文，学习陡峭，初学者对语言的理解比较费脑。
2. 初学者即使理解字面意思了，缺乏了解Rust领域专业知识。
3. 学习挫败感强，效率低，容易放弃，或者三天打鱼，两天晒网，无法持续学习。

本注释版的价值：

1. 简单、准确，完全参考官网材料1:1编写；
2. 完整，全面覆盖Rust的重要特性和知识；
3. 及时迭代更新，对错误的、过时的内容和新增的内容及时更新发布新版本；
4. 初学者不需要搭建复杂的开发环境，可以直接在官网网页的playground上面运行，复杂的地方参考本书注释，通过代码和注解，快速掌握知识。
5. 降低挫败感，持续学习，提升学习效率和体验。

## 0.Why Rust？

​    在学Rust之前，需要搞清楚，为什么要用Rust?以下几点值得考虑：

1. 如果你做前端开发的，想了解后端服务器程序的开发；
2. 如果做Java开发很多年了，想了解系统底层架构，为架构师打基础；
3. 如果是学系统编程，如Linux系统编程和网络编程，Windows API，嵌入式编程
4. 涉及到多线程，大并发领域
5. 嵌入式

以上领域，Rust作为新兴的一门语言，是具有重要的学习价值的。

Rust类似于C++ ,是一门面向系统编程的语言，需要程序员对资源把控有一定的能力，其中语言特性主要参考了C++的新特性，比如Traits类似于C++ 20中的Concepts ，智能指针参考了RAII（Resource Acquisition Is Initialization) .

Rust在嵌入式，系统编程，跨平台方面有出色的表现，解决了C++的一些问题，尤其是在安全方面。微软，华为，google等大公司已经考虑将代码从C++移植到rust。

Rust解决了C++带来的一些问题，是C++的强有力的挑战者，尤其是在安全性方面。

其他重要的特性，读者可以网上搜索一下，这里不再废话。

## 1.Hello World

#### 1.1 Comments(注释)

Any program requires comments, and Rust supports a few different varieties:

- Regular comments

  which are ignored by the compiler:

  - `// Line comments which go to the end of the line.(行注释)`
  - `/* Block comments which go to the closing delimiter. */(块注释)`

- Doc comments which are parsed into HTML library documentation:

  - `/// Generate library docs for the following item.`（库文档）
  - `//! Generate library docs for the enclosing item.(库文档)`

  

#### 1.2 Formatted print(格式化输出)

- `format!`: write formatted text to [`String`](https://doc.rust-lang.org/rust-by-example/std/str.html)
- `print!`: same as `format!` but the text is printed to the console (io::stdout).(输出到控制台stdout)
- `println!`: same as `print!` but a newline is appended.(和print！一样，但是会换行)
- `eprint!`: same as `print!` but the text is printed to the standard error (io::stderr).(和print！一样，但是会输出到stderr)
- `eprintln!`: same as `eprint!` but a newline is appended.(和eprint!一样，但是会换行)

```rust
fn main() {
    // In general, the `{}` will be automatically replaced with any
    // arguments. These will be stringified.
    println!("{} days", 31);

    // Positional arguments can be used. Specifying an integer inside `{}`
    // determines which additional argument will be replaced. Arguments start
    // at 0 immediately after the format string
    println!("{0}, this is {1}. {1}, this is {0}", "Alice", "Bob");

    // As can named arguments.
    println!("{subject} {verb} {object}",
             object="the lazy dog",
             subject="the quick brown fox",
             verb="jumps over");

    // Different formatting can invoked by specified format character after a
    // `:`.
    println!("Base 10 repr:               {}",   69420);
    println!("Base 2 (binary) repr:       {:b}", 69420);
    println!("Base 8 (octal) repr:        {:o}", 69420);
    println!("Base 16 (hexadecimal) repr: {:x}", 69420);
    println!("Base 16 (hexadecimal) repr: {:X}", 69420);

    // You can right-align text with a specified width. This will output
    // "     1". 5 white spaces and a "1".
    println!("{number:>5}", number=1);

    // You can pad numbers with extra zeroes. This will output "000001".
    println!("{number:0>5}", number=1);

    // You can use named arguments in the format specifier by appending a `$`
    println!("{number:0>width$}", number=1, width=5);


    // Rust even checks to make sure the correct number of arguments are
    // used.
    // println!("My name is {0}, {1} {0}", "Bond");
    // FIXME ^ Add the missing argument: "James"

    // Only types that implement fmt::Display can be formatted with `{}`. User-
    // defined types to not implement fmt::Display by default

    #[allow(dead_code)]
    struct Structure(i32);

    // This will not compile because `Structure` does not implement
    // fmt::Display
    // println!("This struct `{}` won't print...", Structure(3));
    // FIXME ^ Comment out this line.

    // For Rust 1.58 and above, you can directly capture the argument from
    // surrounding variable. Just like the above, this will output
    // "     1". 5 white spaces and a "1".
    let number: f64 = 1.0;
    let width: usize = 6;
    println!("{number:>width$}");
}

运行结果：
31 days
Alice, this is Bob. Bob, this is Alice
the quick brown fox jumps over the lazy dog
Base 10 repr:               69420
Base 2 (binary) repr:       10000111100101100
Base 8 (octal) repr:        207454
Base 16 (hexadecimal) repr: 10f2c
Base 16 (hexadecimal) repr: 10F2C
    1
00001
00001
     1
```

##### 1.2.1 Debug

所有的类型需要实现std::fmt 格式化traits 才能打印，只有部分std库中自动实现了，其他的必须要手动实现

fmt::Debug trait 非常直接，所有的类型可以derive (自动创建)fmt::Debug的实现。但对fmt::Display来说还是要手动实现。

```rust

#![allow(unused)]
fn main() {
// This structure cannot be printed either with `fmt::Display` or
// with `fmt::Debug`.
struct UnPrintable(i32);

// The `derive` attribute automatically creates the implementation
// required to make this `struct` printable with `fmt::Debug`.
#[derive(Debug)]
struct DebugPrintable(i32);
}

```

所有标准库类型,可以通过{:?}自动打印:

```rust
// Derive the `fmt::Debug` implementation for `Structure`. `Structure`
// is a structure which contains a single `i32`.
#[derive(Debug)]
struct Structure(i32);

// Put a `Structure` inside of the structure `Deep`. Make it printable
// also.
#[derive(Debug)]
struct Deep(Structure); //嵌套structure

fn main() {
    // Printing with `{:?}` is similar to with `{}`.
    println!("{:?} months in a year.", 12);
    println!("{1:?} {0:?} is the {actor:?} name.",
             "Slater",
             "Christian",
             actor="actor's");

    // `Structure` is printable!
    println!("Now {:?} will print!", Structure(3));
    
    // The problem with `derive` is there is no control over how
    // the results look. What if I want this to just show a `7`?
    // 'derive'的问题是，我们没有办法直接显示 “7”？
    println!("Now {:?} will print!", Deep(Structure(7)));
}

```

所以fmt::Debug确实让对象具备可以打印性，但是牺牲一些优雅性。

Rust还通过”{:# ?}“提供了优雅打印（pretty printing）。

例如：

```rust
#[derive(Debug)]
struct Person<'a> {
    name: &'a str,
    age: u8
}

fn main() {
    let name = "Peter";
    let age = 27;
    let peter = Person { name, age };

    // Pretty print
    println!("{:#?}", peter);
}

运行结果：

Person {
    name: "Peter",
    age: 27,
}

```

另外一个方法是，可以手动实现fmt::Display来控制显示。

##### 1.2.2 Display

虽然使用fmt::Debug看起来已经非常简练了，但是未来更好的满足我们格式的需求，需要手动实现

fmt::Display，它通过使用{}来打印。

例如：

```rust

#![allow(unused)]
fn main() {
// Import (via `use`) the `fmt` module to make it available.
use std::fmt;

// Define a structure for which `fmt::Display` will be implemented. This is
// a tuple struct named `Structure` that contains an `i32`.
struct Structure(i32);

// To use the `{}` marker, the trait `fmt::Display` must be implemented
// manually for the type.
impl fmt::Display for Structure {
    // This trait requires `fmt` with this exact signature.
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        // Write strictly the first element into the supplied output
        // stream: `f`. Returns `fmt::Result` which indicates whether the
        // operation succeeded or failed. Note that `write!` uses syntax which
        // is very similar to `println!`.
        write!(f, "value = {value}", value=self.0)
    }
}
let s = Structure(10);
println!("{}",s);
}

运行结果：
value = 10

```

TODO:需要完善



##### 1.2.3 Formatting





## 2.Primitives 原语

Rust 提供了对各种原语(Primitives)的访问,包括以下：

#### Scalar Types（标量）

- signed integers: `i8`, `i16`, `i32`, `i64`, `i128` and `isize` (pointer size)
- unsigned integers: `u8`, `u16`, `u32`, `u64`, `u128` and `usize` (pointer size)
- floating point: `f32`, `f64`
- `char` Unicode scalar values like `'a'`, `'α'` and `'∞'` (4 bytes each)
- `bool` either `true` or `false`
- and the unit type `()`, whose only possible value is an empty tuple: `()`



#### Compound Types（复合类型）

- arrays like `[1, 2, 3]`
- tuples like `(1, true)`

```rust
fn main() {
    // Variables can be type annotated.
    let logical: bool = true;

    // 在变量后面：指定类型或者在值后面指定类型；
    let a_float: f64 = 1.0;  // Regular annotation
    let an_integer   = 5i32; // Suffix annotation

    
    // 或者 由编译器推导出 默认类型
    // Or a default will be used.
    let default_float   = 3.0; // `f64`
    let default_integer = 7;   // `i32`
    
    // 类型也可以从上下文中推导出来
    // A type can also be inferred from context 
    let mut inferred_type = 12; // Type i64 is inferred from another line
    inferred_type = 4294967296i64;
    
    //mutable变量的值可以修改，但是类型不能修改。
   
    let mut mutable = 12; // Mutable `i32`
    mutable = 21;
    
    
    // Error! The type of a variable can't be changed.
    // mutable = true;
    
    //变量可以覆写通过shadowning系统，及重复声明变量，会覆盖上一个变量；
    // Variables can be overwritten with shadowing.
    let mutable = true;
}

```

### 2.1 Literals and Operators

字面量和操作符

- 整数可以分别使用这些前缀:0 x, o或0 b来表示十六进制表示,八进制或二进制符号
- 下划线__可以插入数字字面值来改善可读性,例如_1_000等于1000,0.000001和0.000 _001是一样的。

```rust
fn main() {
    // Integer addition
    println!("1 + 2 = {}", 1u32 + 2);

    // Integer subtraction
    println!("1 - 2 = {}", 1i32 - 2);
    // TODO ^ Try changing `1i32` to `1u32` to see why the type is important
    // 1u32 -2 ,将会导致overflow
    

    // Short-circuiting boolean logic
    //逻辑运算符
    println!("true AND false is {}", true && false);
    println!("true OR false is {}", true || false);
    println!("NOT true is {}", !true);

    // Bitwise operations
    // 位操作符
    //使用后置字面量: 0x, 0o or 0b.可以修饰整形值；
    
    println!("0011 AND 0101 is {:04b}", 0b0011u32 & 0b0101);
    println!("0011 OR 0101 is {:04b}", 0b0011u32 | 0b0101);
    println!("0011 XOR 0101 is {:04b}", 0b0011u32 ^ 0b0101);
    println!("1 << 5 is {}", 1u32 << 5);
    println!("0x80 >> 2 is 0x{:x}", 0x80u32 >> 2);

    // Use underscores to improve readability!
    println!("One million is written as {}", 1_000_000u32);
}

```



### 2.2 Tuples

元组是**不同类型值**的集合。元组使用括号()构造，每个元组本身是一个具有类型签名(T1，T2，...)的值，其中 T1，T2是其成员的类型。函数可以使用元组返回多个值，因为元组可以保存**任意数量**的值。

```rust
// Tuples can be used as function arguments and as return values
//Tuples可以用作函数参数和返回值

fn reverse(pair: (i32, bool)) -> (bool, i32) {
    // `let` can be used to bind the members of a tuple to variables
    let (integer, boolean) = pair;

    (boolean, integer)
}

// The following struct is for the activity.
#[derive(Debug)]
struct Matrix(f32, f32, f32, f32);

fn main() {
    // A tuple with a bunch of different types
    //Tuple可以包含不同类型的值；
    let long_tuple = (1u8, 2u16, 3u32, 4u64,
                      -1i8, -2i16, -3i32, -4i64,
                      0.1f32, 0.2f64,
                      'a', true);

    // Values can be extracted from the tuple using tuple indexing
    // 值可以使用下表从Tuple从提取；
    println!("long tuple first value: {}", long_tuple.0);
    println!("long tuple second value: {}", long_tuple.1);

    // Tuples can be tuple members
    // Tuples本身也可以作为Tuple的成员
    let tuple_of_tuples = ((1u8, 2u16, 2u32), (4u64, -1i8), -2i16);

    // Tuples are printable
    // Tuples是可以打印的；
    println!("tuple of tuples: {:?}", tuple_of_tuples);
    
    //但是超过12元素的Tuples是无法打印的。
    // But long Tuples (more than 12 elements) cannot be printed
    // let too_long_tuple = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13);
    // println!("too long tuple: {:?}", too_long_tuple);
    // TODO ^ Uncomment the above 2 lines to see the compiler error

    let pair = (1, true);
    println!("pair is {:?}", pair);

    println!("the reversed pair is {:?}", reverse(pair));

    // To create one element tuples, the comma is required to tell them apart
    // from a literal surrounded by parentheses
    println!("one element tuple: {:?}", (5u32,));
    println!("just an integer: {:?}", (5u32));

    //tuples can be destructured to create bindings
    let tuple = (1, "hello", 4.5, true);

    let (a, b, c, d) = tuple;
    println!("{:?}, {:?}, {:?}, {:?}", a, b, c, d);

    let matrix = Matrix(1.1, 1.2, 2.1, 2.2);
    println!("{:?}", matrix);

}

```



### 2.3 Array and Slices

Array 是存储在**连续内存**中的**同一**类型 T 的对象的集合。数组是使用方括号[]创建的，它们的长度(在编译时已知)是它们的类型签名[ T; length ]的一部分。

Slices类似于数组,但是他们的长度是在编译时不知道。相反,Slices是由两个词对象,第一个词是一个指向数据的指针,第二个单词是切片的长度。这个词usize一样大小,由处理器体系结构如64位x86-64。Slices可以用来借一段数组,指定为& [T]类型。



```rust
use std::mem;

// This function borrows a slice
fn analyze_slice(slice: &[i32]) {
    println!("first element of the slice: {}", slice[0]);
    println!("the slice has {} elements", slice.len());
}

fn main() {
    // Fixed-size array (type signature is superfluous)
    //后面的类型声明是多余的，可以省略；
    let xs: [i32; 5] = [1, 2, 3, 4, 5];

    // All elements can be initialized to the same value
    // 所有的元素可以初始化为同一个值；
    let ys: [i32; 500] = [0; 500];

    // Indexing starts at 0
    //索引从0开始
    println!("first element of the array: {}", xs[0]);
    println!("second element of the array: {}", xs[1]);

    // `len` returns the count of elements in the array
    // len方法返回数组的长度；
    println!("number of elements in array: {}", xs.len());

    // Arrays are stack allocated
    // 数组是栈上分配的；
    println!("array occupies {} bytes", mem::size_of_val(&xs));

    //数组可以自动作为Slice借用；
    // Arrays can be automatically borrowed as slices
    println!("borrow the whole array as a slice");
    analyze_slice(&xs);

    // Slices can point to a section of an array
    // They are of the form [starting_index..ending_index]
    // starting_index is the first position in the slice
    // ending_index is one more than the last position in the slice
    //Slice可以指向数组的一块区域，它们从[starting_index..ending_index]
    // starting_index是第一个位置的元素，ending_index是最后一个元素下标+1，
    //例如：
    /*
    fn main(){
    let a = [1,2,3,4,8];
    let s = &a;
    let s1 = &s[0..5];//8的下标是4，下标是从0开始。
    println!("{:?}",s1);
	}
    
    运行结果：
    [1, 2, 3, 4, 8]
   
    */
    
    
    println!("borrow a section of the array as a slice");
    analyze_slice(&ys[1 .. 4]);

    // Example of empty slice `&[]`
    // 空Slice
    let empty_array: [u32; 0] = [];
    assert_eq!(&empty_array, &[]);
    assert_eq!(&empty_array, &[][..]); // same but more verbose

    // Arrays can be safely accessed using `.get`, which returns an
    // `Option`. This can be matched as shown below, or used with
    // `.expect()` if you would like the program to exit with a nice
    // message instead of happily continue.
    //Array（数组）可以通过使用.get方法安全的访问，get方法返回一个Option，它可以通过如下的match方式获取。
    //如果你希望程序给出一个提示信息后退出，可以使用.expect().
    //例如 ： println!("{}",a.get(5).expect("Arrar index out of range."));
    
    for i in 0..xs.len() + 1 { // OOPS, one element too far
        match xs.get(i) {
            Some(xval) => println!("{}: {}", i, xval),
            None => println!("Slow down! {} is too far!", i),
        }
    }

    //访问越界导致编译错误！
    // Out of bound indexing causes compile error
    //println!("{}", xs[5]);
}

```



## 3.自定义类型

rust主要是通过以下两个关键词实现自定义数据类型的:

- `struct`: define a structure
- `enum`: define an enumeration

常量也可以通过const和static关键字创建；

#### 3.1 Structures

可以使用 struct 关键字创建三种类型的结构(“ structs”) :

- Tuple structs
- The classic C structs C风格的struct

- Unit structs, which are field-less, are useful for generics.
- Unit struct 没有成员字段，对泛型编程很有用

```rust
// An attribute to hide warnings for unused code.
#![allow(dead_code)]

#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

// A unit struct
struct Unit;

// A tuple struct
struct Pair(i32, f32);

// A struct with two fields
struct Point {
    x: f32,
    y: f32,
}

// Structs can be reused as fields of another struct
// 结构体可以作为另外一个结构体的字段复用；

struct Rectangle {
    // A rectangle can be specified by where the top left and bottom right
    // corners are in space.
    top_left: Point,
    bottom_right: Point,
}

fn main() {
    // Create struct with field init shorthand
    let name = String::from("Peter");
    let age = 27;
    let peter = Person { name, age };

    // Print debug struct
    println!("{:?}", peter);

    // Instantiate a `Point`
    let point: Point = Point { x: 10.3, y: 0.4 };

    // Access the fields of the point
    println!("point coordinates: ({}, {})", point.x, point.y);

    // Make a new point by using struct update syntax to use the fields of our
    // other one
    let bottom_right = Point { x: 5.2, ..point };

    // `bottom_right.y` will be the same as `point.y` because we used that field
    // from `point`
    println!("second point: ({}, {})", bottom_right.x, bottom_right.y);

    // Destructure the point using a `let` binding
    // 使用let binding 解构,不会发生所有权转移。
    // 赋值会发生所有权转移 ，如 let p2 = point; 之后point将不可以再用了。
    
    
    let Point { x: left_edge, y: top_edge } = point;

    let _rectangle = Rectangle {
        // struct instantiation is an expression too
        top_left: Point { x: left_edge, y: top_edge },
        bottom_right: bottom_right,
    };

    // Instantiate a unit struct
    let _unit = Unit;

    // Instantiate a tuple struct
    let pair = Pair(1, 0.1);

    // Access the fields of a tuple struct
    println!("pair contains {:?} and {:?}", pair.0, pair.1);

    // Destructure a tuple struct
    let Pair(integer, decimal) = pair;

    println!("pair contains {:?} and {:?}", integer, decimal);
}

```



#### 3.2 Enums

The `enum` keyword allows the creation of a type which may be one of a few different variants. Any variant which is valid as a `struct` is also valid as an `enum`.

```rust
// Create an `enum` to classify a web event. Note how both
// names and type information together specify the variant:
// `PageLoad != PageUnload` and `KeyPress(char) != Paste(String)`.
// Each is different and independent.

enum WebEvent {
    // An `enum` may either be `unit-like`,
    PageLoad,
    PageUnload,
    // like tuple structs,
    KeyPress(char),
    Paste(String),
    // or c-like structures.
    Click { x: i64, y: i64 },
}

// A function which takes a `WebEvent` enum as an argument and
// returns nothing.

fn inspect(event: WebEvent) {
    match event {
        WebEvent::PageLoad => println!("page loaded"),
        WebEvent::PageUnload => println!("page unloaded"),
        // Destructure `c` from inside the `enum`.
        WebEvent::KeyPress(c) => println!("pressed '{}'.", c),
        WebEvent::Paste(s) => println!("pasted \"{}\".", s),
        // Destructure `Click` into `x` and `y`.
        WebEvent::Click { x, y } => {
            println!("clicked at x={}, y={}.", x, y);
        },
    }
}

fn main() {
    let pressed = WebEvent::KeyPress('x');
    // `to_owned()` creates an owned `String` from a string slice.
    let pasted  = WebEvent::Paste("my text".to_owned());
    let click   = WebEvent::Click { x: 20, y: 80 };
    let load    = WebEvent::PageLoad;
    let unload  = WebEvent::PageUnload;

    inspect(pressed);
    inspect(pasted);
    inspect(click);
    inspect(load);
    inspect(unload);
}

运行结果：
pressed 'x'.
pasted "my text".
clicked at x=20, y=80.
page loaded
page unloaded

```

Type aliases(类型别名)

如果你使用一个类型别名,您可以通过别名 引用 每个枚举变量 。

这可能是有用的,如果枚举的名字太长或太一般,你想重命名它。

例如：

```rust
enum VeryVerboseEnumOfThingsToDoWithNumbers {
    Add,
    Subtract,
}

// Creates a type alias
type Operations = VeryVerboseEnumOfThingsToDoWithNumbers;

fn main() {
    // We can refer to each variant via its alias, not its long and inconvenient
    // name.
    let x = Operations::Add;
}

```



```rust
enum VeryVerboseEnumOfThingsToDoWithNumbers {
    Add,
    Subtract,
}


impl VeryVerboseEnumOfThingsToDoWithNumbers {
    fn run(&self, x: i32, y: i32) -> i32 {
        match self {
            //Self 语法糖 简写，代表 VeryVerboseEnumOfThingsToDoWithNumbers
            Self::Add => x + y,
            Self::Subtract => x - y,
        }
    }
}

```



##### 3.2.1 use

使用 use 声明可以自动导入enum 定义的变量 [enum生命的究竟是变量，还是值？？]

```rust
// An attribute to hide warnings for unused code.
#![allow(dead_code)]

enum Status {
    Rich,
    Poor,
}

enum Work {
    Civilian,
    Soldier,
}

fn main() {
    // Explicitly `use` each name so they are available without
    // manual scoping.
    use crate::Status::{Poor, Rich};
    // Automatically `use` each name inside `Work`.
    use crate::Work::*;

    // Equivalent to `Status::Poor`.
    let status = Poor;
    // Equivalent to `Work::Civilian`.
    let work = Civilian;

    match status {
        // Note the lack of scoping because of the explicit `use` above.
        Rich => println!("The rich have lots of money!"),
        Poor => println!("The poor have no money..."),
    }

    match work {
        // Note again the lack of scoping.
        Civilian => println!("Civilians work!"),
        Soldier  => println!("Soldiers fight!"),
    }
}

```





##### 3.2.2 C-like

enum 可以像C语言中的enum使用。

例如：

```rust
// An attribute to hide warnings for unused code.
#![allow(dead_code)]

// enum with implicit discriminator (starts at 0)
enum Number {
    Zero,
    One,
    Two,
}

// enum with explicit discriminator
enum Color {
    Red = 0xff0000,
    Green = 0x00ff00,
    Blue = 0x0000ff,
}

fn main() {
    // `enums` can be cast as integers.
    println!("zero is {}", Number::Zero as i32);
    println!("one is {}", Number::One as i32);

    println!("roses are #{:06x}", Color::Red as i32);
    println!("violets are #{:06x}", Color::Blue as i32);
}

运行结果：
zero is 0
one is 1
roses are #ff0000
violets are #0000ff


```



##### 3.2.3 TestCase:linked-list 链表

A common way to implement a linked-list is via `enums`:

 最普通的通过enum方法实现一个链表的例子：

```rust
use crate::List::*;

enum List {
    // Cons: Tuple struct that wraps an element and a pointer to the next node
    Cons(u32, Box<List>),
    // Nil: A node that signifies the end of the linked list
    Nil,
}

// Methods can be attached to an enum
impl List {
    // Create an empty list
    fn new() -> List {
        // `Nil` has type `List`
        Nil
    }

    // Consume a list, and return the same list with a new element at its front
    fn prepend(self, elem: u32) -> List {
        // `Cons` also has type List
        Cons(elem, Box::new(self))
    }

    // Return the length of the list
    //TODO:???
    fn len(&self) -> u32 {
        // `self` has to be matched, because the behavior of this method
        // depends on the variant of `self`
        // `self` has type `&List`, and `*self` has type `List`, matching on a
        // concrete type `T` is preferred over a match on a reference `&T`
        // after Rust 2018 you can use self here and tail (with no ref) below as well,
        // rust will infer &s and ref tail. 
        // See https://doc.rust-lang.org/edition-guide/rust-2018/ownership-and-lifetimes/default-match-bindings.html
        match *self {
            // Can't take ownership of the tail, because `self` is borrowed;
            // instead take a reference to the tail
            Cons(_, ref tail) => 1 + tail.len(),
            // Base Case: An empty list has zero length
            Nil => 0
        }
    }

    // Return representation of the list as a (heap allocated) string
    fn stringify(&self) -> String {
        match *self {
            Cons(head, ref tail) => {
                // `format!` is similar to `print!`, but returns a heap
                // allocated string instead of printing to the console
                format!("{}, {}", head, tail.stringify())
            },
            Nil => {
                format!("Nil")
            },
        }
    }
}

fn main() {
    // Create an empty linked list
    let mut list = List::new();

    // Prepend some elements
    list = list.prepend(1);
    list = list.prepend(2);
    list = list.prepend(3);

    // Show the final state of the list
    println!("linked list has length: {}", list.len());
    println!("{}", list.stringify());
}

```



#### 3.3 constants

- `const`: 不可改变的值 (the common case).
- `static`: A possibly `mut`able variable with [`'static`](https://doc.rust-lang.org/rust-by-example/scope/lifetime/static_lifetime.html) lifetime. 访问static 修饰的变量是unsafe的，需要包含在unsafe语句块内.

```rust
// Globals are declared outside all other scopes.

const THRESHOLD: i32 = 10;
static mut LANGUAGE: &str = "Rust"; //这里是mut

fn is_big(n: i32) -> bool {
    // Access constant in some function
    n > THRESHOLD
}

fn main() {
    let n = 16;
    // 访问和修改static变量必须要使用unsafe语句块。
    unsafe {
        LANGUAGE = "Cpp";
        println!("This is {}", LANGUAGE);
    }

    
    println!("The threshold is {}", THRESHOLD);
    println!("{} is {}", n, if is_big(n) { "big" } else { "small" });

    // Error! Cannot modify a `const`.
    // THRESHOLD = 5;
    // FIXME ^ Comment out this line
}

```



## 4.变量绑定

Variable Bindings（变量绑定）

 　rust通过静态类型来提供类型安全。当声明变量时，可以绑定类型说明。

然而,在大多数情况下,编译器可以从上下文中推断出变量的类型,大大的降低了类型声明的负担。

值（类似字面量），可以通过let绑定到变量

例如：

```rust
fn main() {
    let an_integer = 1u32;
    let a_boolean = true;
    let unit = ();

    // copy `an_integer` into `copied_integer`
    let copied_integer = an_integer;

    println!("An integer: {:?}", copied_integer);
    println!("A boolean: {:?}", a_boolean);
    println!("Meet the unit value: {:?}", unit);

    // The compiler warns about unused variable bindings; these warnings can
    // be silenced by prefixing the variable name with an underscore
    //没有使用变量，可以在变量名前面加上下划线_;
    let _unused_variable = 3u32;

    let noisy_unused_variable = 2u32;
    // FIXME ^ Prefix with an underscore to suppress the warning
    // Please note that warnings may not be shown in a browser
}

```

### 4.1 Mutability(可变性)

变量的声明，默认是不可变的，但是可以使用mut 使其可变；

```rust
fn main() {
    let _immutable_binding = 1;
    let mut mutable_binding = 1;

    println!("Before mutation: {}", mutable_binding);

    // Ok
    mutable_binding += 1;

    println!("After mutation: {}", mutable_binding);

    // Error!
    //_immutable_binding += 1;
    // FIXME ^ Comment out this line
}

```



### 4.2 Scope and Shadowing

变量的有效性是有范围的,限制在{}语句块中。

```rust
fn main() {
    // This binding lives in the main function
    let long_lived_binding = 1;

    // 这个语句块的访问要比main函数小
    {
        //这个声明，只在这个语句块中有效
        //注意：变量超出语句块后，将自动销毁；
        let short_lived_binding = 2;

        println!("inner short: {}", short_lived_binding);
    }
    // 语句块结束

    // 错误！ `short_lived_binding` 不存在这个范围
    // println!("outer short: {}", short_lived_binding);
    // FIXME ^ Comment out this line

    println!("outer long: {}", long_lived_binding);
}

```

变量的shadowing（可以理解为太阳的影子）

参考以下例子，运行结果对比一下，很容易发现规律。

- shadowing的范围限制在语句块内，离开语句块后，对原始变量没有影响；
- 如果要影响原始变量，需要在同一个语句块范围重新shadowing.

```rust
fn main() {
    let shadowed_binding = 1;

    {
        println!("before being shadowed: {}", shadowed_binding);

        // This binding *shadows* the outer one
        let shadowed_binding = "abc";

        println!("shadowed in inner block: {}", shadowed_binding);
    }
    println!("outside inner block: {}", shadowed_binding);

    // This binding *shadows* the previous binding
    let shadowed_binding = 2;
    println!("shadowed in outer block: {}", shadowed_binding);
}

运行结果：

before being shadowed: 1
shadowed in inner block: abc
outside inner block: 1
shadowed in outer block: 2
```



### 4.3 Declare first（先声明，后bingding）

可以先声明变量,然后再初始化它们。

然而,这种形式是很少使用,因为它可能会导致使用未初始化的变量。

注意：最好像C++一样，声明即初始化。



```rust
fn main() {
    // Declare a variable binding
    let a_binding;

    {
        let x = 2;

        // Initialize the binding
        a_binding = x * x;
    }

    println!("a binding: {}", a_binding);

    let another_binding;

    // Error! Use of uninitialized binding
    // println!("another binding: {}", another_binding);
    // FIXME ^ Comment out this line

    another_binding = 1;

    println!("another binding: {}", another_binding);
}

```



### 4.4 Freezing

考虑如下情况：

```rust
fn main() {
    let mut _mutable_integer = 7i32;

    {
        // Shadowing by immutable `_mutable_integer`
        let _mutable_integer = _mutable_integer;

        // Error! `_mutable_integer` is frozen in this scope
        _mutable_integer = 50;
        // FIXME ^ Comment out this line

        // `_mutable_integer` goes out of scope
    }

    // Ok! `_mutable_integer` is not frozen in this scope
    _mutable_integer = 3;
}

   Compiling playground v0.0.1 (/playground)
error[E0384]: cannot assign twice to immutable variable `_mutable_integer`
 --> src/main.rs:9:9
  |
6 |         let _mutable_integer = _mutable_integer;
  |             ----------------
  |             |
  |             first assignment to `_mutable_integer`
  |             help: consider making this binding mutable: `mut _mutable_integer`
...
9 |         _mutable_integer = 50;
  |         ^^^^^^^^^^^^^^^^^^^^^ cannot assign twice to immutable variable


error[E0384]: cannot assign twice to immutable variable `_mutable_integer`

当变量声明为同一个名字的、不可以修改变量，会发生freeze现象。

frozen 的 data 需要等到离开作用域才能修改；


```



## 5.类型

Rust provides several mechanisms to change or define the type of primitive and user defined types. The following sections cover:

- Casting between primitive types
- Specifying the desired type of literals
- Using type inference
- Aliasing types

### 5.1 Casting(转换)

Rust provides no implicit type conversion (coercion) between primitive types. But, explicit type conversion (casting) can be performed using the `as` keyword.

Rust没有对简单（primitive）类型提供隐式转换功能，但是可以通过as关键字提供显示转换；

Rules for converting between integral types follow C conventions generally, except in cases where C has undefined behavior. The behavior of all casts between integral types is well defined in Rust.

整体类型之间的转换规则遵循C语言约定，除C有UB（undefined behavior.）之外。整体的转换行为在rust都有良好的定义；



注意转换规则：

有符号数和无符号数的转换，需要考虑溢出和符号的变化，

有符号数可能会有正数变为负数。



```rust
// Suppress all warnings from casts which overflow.
#![allow(overflowing_literals)]

fn main() {
    let decimal = 65.4321_f32;

    // Error! No implicit conversion
    let integer: u8 = decimal;
    // FIXME ^ Comment out this line

    // Explicit conversion
    let integer = decimal as u8;
    let character = integer as char;

    // Error! There are limitations in conversion rules. 
    // A float cannot be directly converted to a char.
    let character = decimal as char;
    // 错误信息：only `u8` can be cast as `char`, not `f32`
    // FIXME ^ Comment out this line

    println!("Casting: {} -> {} -> {}", decimal, integer, character);

    // when casting any value to an unsigned type, T,
    // T::MAX + 1 is added or subtracted until the value
    // fits into the new type

    // 1000 already fits in a u16
    println!("1000 as a u16 is: {}", 1000 as u16);

    // 1000 - 256 - 256 - 256 = 232
    // Under the hood, the first 8 least significant bits (LSB) are kept,
    // while the rest towards the most significant bit (MSB) get truncated.
    println!("1000 as a u8 is : {}", 1000 as u8);
    // -1 + 256 = 255
    println!("  -1 as a u8 is : {}", (-1i8) as u8);

    // For positive numbers, this is the same as the modulus
    println!("1000 mod 256 is : {}", 1000 % 256);

    // When casting to a signed type, the (bitwise) result is the same as
    // first casting to the corresponding unsigned type. If the most significant
    // bit of that value is 1, then the value is negative.

    // Unless it already fits, of course.
    println!(" 128 as a i16 is: {}", 128 as i16);
    
    // 128 as u8 -> -128, whose two's complement in eight bits is:
    println!(" 128 as a i8 is : {}", 128 as i8);

    // repeating the example above
    // 1000 as u8 -> 232
    println!("1000 as a u8 is : {}", 1000 as u8);
    // and the two's complement of 232 is -24
    println!(" 232 as a i8 is : {}", 232 as i8);
    
    // Since Rust 1.45, the `as` keyword performs a *saturating cast* 
    // when casting from float to int. If the floating point value exceeds 
    // the upper bound or is less than the lower bound, the returned value 
    // will be equal to the bound crossed.
    
    // 300.0 is 255
    println!("300.0 is {}", 300.0_f32 as u8);
    // -100.0 as u8 is 0
    println!("-100.0 as u8 is {}", -100.0_f32 as u8);
    // nan as u8 is 0
    println!("nan as u8 is {}", f32::NAN as u8);
    
    // This behavior incurs a small runtime cost and can be avoided 
    // with unsafe methods, however the results might overflow and 
    // return **unsound values**. Use these methods wisely:
    unsafe {
        // 300.0 is 44
        println!("300.0 is {}", 300.0_f32.to_int_unchecked::<u8>());
        // -100.0 as u8 is 156
        println!("-100.0 as u8 is {}", (-100.0_f32).to_int_unchecked::<u8>());
        // nan as u8 is 0
        println!("nan as u8 is {}", f32::NAN.to_int_unchecked::<u8>());
    }
}

```



### 5.2 Literals(字面量)

数字字面量类型，可以在后面添加注释。

例如,42应该有一个类型i32,写作42i32.



如果数字字面量类型，后面没有添加注释，那么它的类型将取决于如何使用它们。

如果没有约束存在,编译器将使用i32作为整数类型,f64作为浮点数类型。

例如

```rust
fn main() {
    // Suffixed literals, their types are known at initialization
    // 初始化即知道类型
    let x = 1u8;
    let y = 2u32;
    let z = 3f32;

    // Unsuffixed literals, their types depend on how they are used
    // 没有后缀的字面量，他们的类型由他们如何使用决定。
    let i = 1;
    let f = 1.0;

    // `size_of_val` returns the size of a variable in bytes
    println!("size of `x` in bytes: {}", std::mem::size_of_val(&x));
    println!("size of `y` in bytes: {}", std::mem::size_of_val(&y));
    println!("size of `z` in bytes: {}", std::mem::size_of_val(&z));
    println!("size of `i` in bytes: {}", std::mem::size_of_val(&i));
    println!("size of `f` in bytes: {}", std::mem::size_of_val(&f));
}

运行结果：

size of `x` in bytes: 1
size of `y` in bytes: 4
size of `z` in bytes: 4
size of `i` in bytes: 4
size of `f` in bytes: 8

(整型占用4个byte，浮点占用8个byte)

```

### 5.3 Inference(类型推导)

类型推断引擎很聪明。它不但看变量初始化期间值的类型，也看如何使用变量，来推断它的类型。

这是一个先进的类型推断的例子:

```rust
fn main() {
    // Because of the annotation, the compiler knows that `elem` has type u8.
    let elem = 5u8;

    // Create an empty vector (a growable array).
    let mut vec = Vec::new();
    // At this point the compiler doesn't know the exact type of `vec`, it
    // just knows that it's a vector of something (`Vec<_>`).
    
    //在这个时间点，编译器并不知道vec的确切类型，它只知道是一个`Vec<_>`的类型。

    // Insert `elem` in the vector.
    vec.push(elem);
    // Aha! Now the compiler knows that `vec` is a vector of `u8`s (`Vec<u8>`)
    // TODO ^ Try commenting out the `vec.push(elem)` line
    // 直到我们在vec中存放一个数值时，vec才知道准确的类型，是`Vec<u8>`

    println!("{:?}", vec);
}

```

No type annotation of variables was needed, the compiler is happy and so is the programmer!

所以编译器后台为程序员做了很多事，可以让我们不需要每一个变量都需要声明类型。

有点类似C++ 11 后的auto 关键字。



### 5.4 Aliasing(别名)

type 声明 可以用来给现有的类型一个新名字。

类型必须遵守UpperCamelCase规则,否则编译器会提高一个警告。

这个规则的例外是基本类型:usize, f32等等。

```rust
// `NanoSecond`, `Inch`, and `U64` are new names for `u64`.
type NanoSecond = u64;
type Inch = u64;
type U64 = u64;

fn main() {
    // `NanoSecond` = `Inch` = `U64` = `u64`.
    let nanoseconds: NanoSecond = 5 as U64;
    let inches: Inch = 2 as U64;

    // Note that type aliases *don't* provide any extra type safety, because
    // aliases are *not* new types
    // ???
    println!("{} nanoseconds + {} inches = {} unit?",
             nanoseconds,
             inches,
             nanoseconds + inches);
}

```

别名的主要用途是为了减少boilerplate（重复使用的代码），例如

IoResult<T> 是`Result<T, IoError>`的别名





## 6.Conversion 转换

简单类型可以互相转换。

Rust提供通过使用traits在自定义类型（struct,enum）间转换。一般的转换使用From 和Into traits。

### 6.1 From and Into

From和Into traits是内在关联的，如果可以从A转换为B，那么我们认为，也可以从B转换A。

#### From

From trait允许我们定义如何从其他的类型来创建自己，因此提供了一个非常简单的机制来做转换。在标准库里

有几种实现了该traits来做简单类型的转换。



例如我们可以很容易的从str 转为 String

```rust

#![allow(unused)]
fn main() {
    let my_str = "hello";
    let my_string = String::from(my_str);
}

```

我们也能为我们自定义的类型做同样的转换,例如：

```rust
use std::convert::From;

#[derive(Debug)]
struct Number {
    value: i32,
}

impl From<i32> for Number {
    fn from(item: i32) -> Self {
        Number { value: item }
    }
}

fn main() {
    let num = Number::from(30);
    println!("My number is {:?}", num);
}

```



#### Into



如果为类型实现了From trait,into()在需要的时候会自动调用From traits。

例如：

```rust
use std::convert::From;

#[derive(Debug)]
struct Number {
    value: i32,
}

impl From<i32> for Number {
    fn from(item: i32) -> Self {
        Number { value: item }
    }
}

fn main() {
    let a = 5;
    // Try removing the type declaration
    let num: Number = a.into();
    println!("My number is {:?}", num);
}
```



### 6.2 TryFrom and TryInto

和 From 、Into 一样, TryFrom 、TryInto 类型转型通用的trait。

和 From、Into, 不同的是， TryFrom、TryInto traits 被用来会产生转换错误的场景，比如, 还回 Result.

```rust
use std::convert::TryFrom;
use std::convert::TryInto;

#[derive(Debug, PartialEq)]
struct EvenNumber(i32);

impl TryFrom<i32> for EvenNumber {
    type Error = ();

    fn try_from(value: i32) -> Result<Self, Self::Error> {
        if value % 2 == 0 {
            Ok(EvenNumber(value))
        } else {
            Err(())
        }
    }
}

fn main() {
    // TryFrom

    assert_eq!(EvenNumber::try_from(8), Ok(EvenNumber(8)));
    assert_eq!(EvenNumber::try_from(5), Err(()));

    // TryInto

    let result: Result<EvenNumber, ()> = 8i32.try_into();
    assert_eq!(result, Ok(EvenNumber(8)));
    let result: Result<EvenNumber, ()> = 5i32.try_into();
    assert_eq!(result, Err(()));
}

```



### 6.3 To and from Strings

#### Converting to String

任何类型转换为字符串只需要简单实现ToString traits,

我们一般不这样做,应该实现fmt::Display traits ，它自动提供ToString trait.

例如：

```rust
use std::fmt;

struct Circle {
    radius: i32
}

impl fmt::Display for Circle {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Circle of radius {}", self.radius)
    }
}

fn main() {
    let circle = Circle { radius: 6 };
    println!("{}", circle.to_string());
}

```





#### Parsing a String

一种更加常见的需求是，将一个字符串转换为一个数字，惯用方法是使用解析函数parse()。



例如：

```rust
fn main() {
    let parsed: i32 = "5".parse().unwrap();
    let turbo_parsed = "10".parse::<i32>().unwrap();

    let sum = parsed + turbo_parsed;
    println!("Sum: {:?}", sum);
}

```

如要把字符串转换成指定的类型,只需要实现FromStr trait.

例如：

```rust
use std::{str::FromStr, fmt::Display};
#[derive(Debug)]
struct Money {value:i32}

impl Display for Money {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f,"output is {}",self.value)
    }
}

impl FromStr for Money  {
    type Err = i32;

    fn from_str(s: &str) -> Result<Self, Self::Err> {
        if !s.is_empty() {
            Ok(Money{value:100})
        }else {
            Err(-1)
        }
    }
}
#[allow(unused_variables)]
fn main(){
    let ss = Money{value:33};
    let ss = Money::from_str("10").unwrap();
    println!("{:?}",ss.to_string());
}
```



## 7.表达式

## 8.流控制



### 8.5 match

#### 8.5.1 解构

- Dereferencing uses `*`
- Destructuring uses `&`, `ref`, and `ref mut`

```rust
fn main() {
    // Try changing the values in the array, or make it a slice!
    let array = [1, -2, 6];

    match array {
        // Binds the second and the third elements to the respective variables
        [0, second, third] =>
            println!("array[0] = 0, array[1] = {}, array[2] = {}", second, third),

        // Single values can be ignored with _
        //_下划线表示忽略该元素；
        [1, _, third] => println!(
            "array[0] = 1, array[2] = {} and array[1] was ignored",
            third
        ),

        // You can also bind some and ignore the rest
        [-1, second, ..] => println!(
            "array[0] = -1, array[1] = {} and all the other ones were ignored",
            second
        ),
        // The code below would not compile
        // [-1, second] => ...

        // Or store them in another array/slice (the type depends on
        // that of the value that is being matched against)
        [3, second, tail @ ..] => println!(
            "array[0] = 3, array[1] = {} and the other elements were {:?}",
            second, tail
        ),

        // Combining these patterns, we can, for example, bind the first and
        // last values, and store the rest of them in a single array
        //@符合表示剩下的、除了first和last之外的所有元素。
        [first, middle @ .., last] => println!(
            "array[0] = {}, middle = {:?}, array[2] = {}",
            first, middle, last
        ),
    }
}

```

##### 8.5.1.4 Pointer/ref

```rust
fn main() {
    let value = 5;
    let mut mut_value = 6;

    // Use `ref` keyword to create a reference.
    // 这个地方有点难理解，不用ref关键字也是同样的运行结果；
    match value {
        ref r => println!("Got a reference to a value: {:?}", r),
    }

    // Use `ref mut` similarly.
    match mut_value {
        ref mut m => {
            // Got a reference. Gotta dereference it before we can
            // add anything to it.
            // 获得m的一个可变引用，以便我们可以修改他的值；
            *m += 10;
            println!("We added 10. `mut_value`: {:?}", m);
        },
    }
}
```

##### 8.8.1.5 structs 解构

```rust
fn main() {
    struct Foo {
        x: (u32, u32),
        y: u32,
    }

    // Try changing the values in the struct to see what happens
    let foo = Foo { x: (1, 2), y: 3 };

    match foo {
        Foo { x: (2, b), y } => println!("First of x is 1, b = {},  y = {} ", b, y),

        // you can destructure structs and rename the variables,
        // the order is not important
        Foo { y: 2, x: i } => println!("y is 2, i = {:?}", i),

        // and you can also ignore some variables:
        Foo { y, .. } => println!("y = {}, we don't care about x", y),
        // this will give an error: pattern does not mention field `x`
        //Foo { y } => println!("y = {}", y),
    }
}

运行结果：
//y = 3, we don't care about x
```

#### 8.5.2 Guards 守卫

A `match` *guard* can be added to filter the arm.

guard用来做进一步的过滤

```rust
enum Temperature {
    Celsius(i32),
    Farenheit(i32),
}

fn main() {
    let temperature = Temperature::Celsius(15);
    // ^ TODO try different values for `temperature`

    match temperature {
        Temperature::Celsius(t) if t > 30 => println!("{}C is above 30 Celsius", t),
        // The `if condition` part ^ is a guard
        // if 条件语句为guard 守卫
        Temperature::Celsius(t) => println!("{}C is below 30 Celsius", t),

        Temperature::Farenheit(t) if t > 86 => println!("{}F is above 86 Farenheit", t),
        Temperature::Farenheit(t) => println!("{}F is below 86 Farenheit", t),
    }
}

运行结果：
15C is below 30 Celsius
```



```rust
fn main() {
    let number: u8 = 40;

    match number {
        i if i == 0 => println!("Zero"),
        i if i > 0 => println!("Greater than zero"),
        _ => unreachable!("Should never happen."),
        // 取消这行会报错；
    }
}

运行结果：
Greater than zero

```

#### 8.5.3 Binding

 rust provides the `@` sigil for binding values to names:

rust提供了@符号，将值绑定到名字

代码：

```rust
// A function `age` which returns a `u32`.
fn age() -> u32 {
    100
}

fn main() {
    println!("Tell me what type of person you are");

    match age() {
        0             => println!("I haven't celebrated my first birthday yet"),
        // Could `match` 1 ..= 12 directly but then what age
        // would the child be? Instead, bind to `n` for the
        // sequence of 1 ..= 12. Now the age can be reported.
        n @ 1  ..= 12 => println!("I'm a child of age {:?}", n),
        n @ 13 ..= 19 => println!("I'm a teen of age {:?}", n),
        // Nothing bound. Return the result.
        n             => println!("I'm an old person of age {:?}", n),
    }
}
运行结果：
Tell me what type of person you are
I'm an old person of age 100


```

### 8.6 if let

不使用if let

```rust
// Make `optional` of type `Option<i32>`
let optional = Some(7);

match optional {
    Some(i) => {
        println!("This is a really long string and `{:?}`", i);
        // ^ Needed 2 indentations just so we could destructure
        // `i` from the option.
    },
    _ => {},
    // ^ Required because `match` is exhaustive. Doesn't it seem
    // like wasted space?
    //比较啰嗦，每个都要这样写，因为match需要穷尽；
};

```

改用if let后的代码：

```rust
fn main() {
    let number = Some(7);
    if let Some(i) = number {
        //注意：这里是赋值=，不是逻辑==
        println!("Matched {:?}!", i);
    }
}

```

将if let 融入逻辑分支语句：

```rust
fn main() {
    // All have type `Option<i32>`
    let number = Some(7);
    let letter: Option<i32> = None;
    let emoticon: Option<i32> = None;

    // The `if let` construct reads: "if `let` destructures `number` into
    // `Some(i)`, evaluate the block (`{}`).
    if let Some(i) = number {
        println!("Matched {:?}!", i);
    }

    // If you need to specify a failure, use an else:
    //如果需要指定匹配失败后执行的语句，可以使用else
    if let Some(i) = letter {
        println!("Matched {:?}!", i);
    } else {
        // Destructure failed. Change to the failure case.
        println!("Didn't match a number. Let's go with a letter!");
    }

    // Provide an altered failing condition.
    let i_like_letters = false;

    
    //if语句不符合，判断else if 也不符合，最后运行到else语句块；
    if let Some(i) = emoticon {
        println!("Matched {:?}!", i);
    // Destructure failed. Evaluate an `else if` condition to see if the
    // alternate failure branch should be taken:
    } else if i_like_letters {
        println!("Didn't match a number. Let's go with a letter!");
    } else {
        // The condition evaluated false. This branch is the default:
        println!("I don't like letters. Let's go with an emoticon :)!");
    }
}

```



注意：

```rust
// This enum purposely neither implements nor derives PartialEq.
// 这里不能使用 == ，因为没有 继承 PartialEq，
// That is why comparing Foo::Bar == a fails below.
enum Foo {Bar}

fn main() {
    let a = Foo::Bar;

    // Variable a matches Foo::Bar
    if Foo::Bar == a {
    
    // ^-- this causes a compile-time error. Use `if let` instead.
        println!("a is foobar");
    }
}

```

改为如下代码，也可以正常编译：

```Rust
#[derive(PartialEq)]
enum Foo {Bar}

fn main() {
    let a = Foo::Bar;

    // Variable a matches Foo::Bar
    if Foo::Bar == a {
    // ^-- this causes a compile-time error. Use `if let` instead.
        println!("a is foobar");
    }
}

```

注意：

​		if let 后面可以加else /else if 语句；

### 8.7 while let

```rust

#![allow(unused)]
fn main() {
// Make `optional` of type `Option<i32>`
let mut optional = Some(0);

// Repeatedly try this test.
loop {
    match optional {
        // If `optional` destructures, evaluate the block.
        Some(i) => {
            if i > 9 {
                println!("Greater than 9, quit!");
                optional = None;
            } else {
                println!("`i` is `{:?}`. Try again.", i);
                optional = Some(i + 1);
            }
            // ^ Requires 3 indentations!
        },
        // Quit the loop when the destructure fails:
        _ => { break; }
        // ^ Why should this be required? There must be a better way!
    }
}
}

```

如果注释掉 _ => { break; } 

会报如下错误：

 match optional {
    |           ^^^^^^^^ pattern `None` not covered



改为while let

```rust
fn main() {
    let mut optional = Some(0);
    while let Some(i) = optional {
        // = 等号为赋值，不要写出== 
        if i > 9 {
            println!("Greater than 9, quit!");
            optional = None;
        } else {
            println!("`i` is `{:?}`. Try again.", i);
            optional = Some(i + 1);
        }
    }
}
//注意：
. 不需要额外的None处理；
. 和if let不同，while后面不能加else/else if 语句；
```

## 9.函数

### 9.1方法

关联函数和方法

一些函数与特定的类型相关，有两种形式：关联函数和方法，关联函数通常定义在特定的类型上，方法是调用特定类型实例的关联函数。

*关联函数，相当于静态方法，或者C++中用statci定义的方法，不依赖于this指针；*

代码：

```rust
struct Point {
    x: f64,
    y: f64,
}

// Implementation block, all `Point` associated functions & methods go in here
// 所有Point相关的关联函数和方法都定义在这里；
impl Point {
    // This is an "associated function" because this function is associated with
    // a particular type, that is, Point.
    //这是一个关联函数，因为他与Point关联；
    // Associated functions don't need to be called with an instance.
    // These functions are generally used like constructors.
    // 关联函数调用不需要初始化一个实例，这些函数通常像构造函数一样；
    fn origin() -> Point {
        Point { x: 0.0, y: 0.0 }
    }

    // Another associated function, taking two arguments:
    fn new(x: f64, y: f64) -> Point {
        Point { x: x, y: y }
    }
    //例如可以这样调用：
    // let p = Point::new(0.0,0.0);
    // 它不需要初始化一个实例，就可以调用；

}

struct Rectangle {
    p1: Point,
    p2: Point,
}

impl Rectangle {
    // 这里是方法
    // &self 是self::&Self的语法糖，Self是调用者对象，这里Self=Rectange
    fn area(&self) -> f64 {
        // `self` gives access to the struct fields via the dot operator
        //self通过.点号，访问结构体的字段；
        let Point { x: x1, y: y1 } = self.p1;
        let Point { x: x2, y: y2 } = self.p2;

        // `abs` is a `f64` method that returns the absolute value of the
        // caller
        //这里的abs是调用方法，用来返回调用者的绝对值；
        ((x1 - x2) * (y1 - y2)).abs()
    }

    fn perimeter(&self) -> f64 {
        let Point { x: x1, y: y1 } = self.p1;
        let Point { x: x2, y: y2 } = self.p2;

        2.0 * ((x1 - x2).abs() + (y1 - y2).abs())
    }

    // This method requires the caller object to be mutable
    // `&mut self` desugars to `self: &mut Self`
    // &mut self 是 self::&mut Self 的语法糖形式；
    
    fn translate(&mut self, x: f64, y: f64) {
        self.p1.x += x;
        self.p2.x += x;

        self.p1.y += y;
        self.p2.y += y;
    }
}

// `Pair` owns resources: two heap allocated integers
struct Pair(Box<i32>, Box<i32>);

impl Pair {
    // This method "consumes" the resources of the caller object
    // `self` desugars to `self: Self`
    // self 是 self::Self的语法糖；
    
    fn destroy(self) {
        // Destructure `self`
        // 解构了self，结构体赋值，发生了生命周期转移操作，self move 给了Pair
        // self不存在了，从而实现了销毁操作语义；
        let Pair(first, second) = self;

        println!("Destroying Pair({}, {})", first, second);

        // `first` 和 `second` 超出作用范围，并且销毁掉了。
    }
}

fn main() {
    let rectangle = Rectangle {
        //关联函数调用采用::(2个冒号)符号
        p1: Point::origin(),
        p2: Point::new(3.0, 4.0),
    };


    // 方法的调用采用. (dot)（点号）
    // 注意第一个&self 为隐式传递；
    // `rectangle.perimeter()` === `Rectangle::perimeter(&rectangle)`
    
    println!("Rectangle perimeter: {}", rectangle.perimeter());
    println!("Rectangle area: {}", rectangle.area());

    let mut square = Rectangle {
        p1: Point::origin(),
        p2: Point::new(1.0, 1.0),
    };


    //translate调用需要一个Mutable 对象,所有以下调用会失败
    //rectangle.translate(1.0, 0.0);    

    // 现在square是一个 Mutable 对象，所有可以条用mutable方法
    square.translate(1.0, 1.0);

    let pair = Pair(Box::new(1), Box::new(2));

    pair.destroy();

    //destroy不能多次调用，调用一次已经销毁了，在调用一次会出错；
    // Error! Previous `destroy` call "consumed" `pair`
    //pair.destroy();
    // TODO ^ Try uncommenting this line
}

```



### 9.2闭包

Closures are functions that can capture the enclosing environment.

 For example, a closure that captures the `x` variable:

```rust
|val| val + x
```

闭包的特性：

- 输入、输出类型可以被推导出来，但是输入的变量名必须指定；
- 输入参数用||代替（）
- 如果是一个表达式，闭包体分隔符{}可以省略
- 可以捕获外面环境变量的值

例如：

```rust
fn main() {
    // Increment via closures and functions.
    fn function(i: i32) -> i32 { i + 1 }

    // Closures are anonymous, here we are binding them to references
    // Annotation is identical to function annotation but is optional
    // as are the `{}` wrapping the body. These nameless functions
    // are assigned to appropriately named variables.
    let closure_annotated = |i: i32| -> i32 { i + 1 };
    let closure_inferred  = |i     |          i + 1  ;

    let i = 1;
    // Call the function and closures.
    println!("function: {}", function(i));
    println!("closure_annotated: {}", closure_annotated(i));
    println!("closure_inferred: {}", closure_inferred(i));

    // A closure taking no arguments which returns an `i32`.
    // The return type is inferred.
    let one = || 1;
    println!("closure returning one: {}", one());

}

```

运行结果：

function: 2
closure_annotated: 2
closure_inferred: 2
closure returning one: 1





#### 9.2.1 Capturing （捕获）

闭包可以通过如下方式捕获变量：

- 通过引用：&T
- 通过可变引用：&mut T
- 通过值：T

他们优先通过引用捕获变量，只有在需要时下降（go lower）；

例如：

```rust
fn main() {
    use std::mem;
    let color = String::from("green");

    // A closure to print `color` which immediately borrows (`&`) `color` and
    // stores the borrow and closure in the `print` variable. It will remain
    // borrowed until `print` is used the last time. 
    //
    // `println!` only requires arguments by immutable reference so it doesn't
    // impose anything more restrictive.
    let print = || println!("`color`: {}", color);

    // Call the closure using the borrow.
    print();

    // `color` can be borrowed immutably again, because the closure only holds
    // an immutable reference to `color`. 
    let _reborrow = &color;
    print();

    // A move or reborrow is allowed after the final use of `print`
    
    let _color_moved = color;

    //注意：color的所有权已经转移，不能再使用color了。

    let mut count = 0;
    // A closure to increment `count` could take either `&mut count` or `count`
    // but `&mut count` is less restrictive so it takes that. Immediately
    // borrows `count`.
    //
    // A `mut` is required on `inc` because a `&mut` is stored inside. Thus,
    // calling the closure mutates the closure which requires a `mut`.
    let mut inc = || {
        count += 1;
        println!("`count`: {}", count);
    };

    // Call the closure using a mutable borrow.
    inc();

    // The closure still mutably borrows `count` because it is called later.
    // An attempt to reborrow will lead to an error.
    // let _reborrow = &count; 
    // ^ TODO: try uncommenting this line.
    inc();

    // The closure no longer needs to borrow `&mut count`. Therefore, it is
    // possible to reborrow without an error
    let _count_reborrowed = &mut count; 

    
    // A non-copy type.
    let movable = Box::new(3);

    // `mem::drop` requires `T` so this must take by value. A copy type
    // would copy into the closure leaving the original untouched.
    // A non-copy must move and so `movable` immediately moves into
    // the closure.
    let consume = || {
        println!("`movable`: {:?}", movable);
        mem::drop(movable); //drop掉moveable后，将不能再次调用consume()
    };

    // `consume` consumes the variable so this can only be called once.
    consume();
    // consume();
    // ^ TODO: Try uncommenting this line.
}

```



笔者添加：

**补充：**

Drop 对于实现 `Copy` 的类型实际上没有任何作用，例如整数。

此类值被复制然后移动到函数中，因此该值在此函数调用后仍然存在。

这个函数并不神奇；它的字面意思是

```
pub fn drop<T>(_x:T) { }
```

因为`_x` 被移动到函数中，所以在函数返回之前它会被自动删除。



在||前使用move强制获取捕获变量的所有权（ownship）

例如：

```rust
fn main() {
    // `Vec` has non-copy semantics.
    let haystack = vec![1, 2, 3];

    let contains =  move|needle| haystack.contains(needle);

    println!("{}", contains(&1));
    println!("{}", contains(&4));

    // println!("There're {} elements in vec", haystack.len());
    // ^ Uncommenting above line will result in compile-time error
    // because borrow checker doesn't allow re-using variable after it
    // has been moved.
    
    // Removing `move` from closure's signature will cause closure
    // to borrow _haystack_ variable immutably, hence _haystack_ is still
    // available and uncommenting above line will not cause an error.
}

```

#### 9.2.2 As input parameters（作为输入参数）



- FnOnce consumes the variables it captures from its enclosing scope, known as the closure’s environment. To consume the captured variables, the closure must take ownership of these variables and move them into the closure when it is defined. The Once part of the name represents the fact that the closure can’t take ownership of the same variables more than once, so it can be called only once.

- FnMut can change the environment because it mutably borrows values.

- Fn borrows values from the environment immutably.

  

- `Fn`: the closure uses the captured value by **reference** (`&T`)
- `FnMut`: the closure uses the captured value by **mutable referenc**e (`&mut T`)
- `FnOnce`: the closure uses the captured value by **value** (`T`)

下面的代码，在闭包中，有的需要Fn Trait,有的需要实现FnMut Trait,所以F：的限制，只有FnOnce满足，因为所有的参数都可以满足FnOnce。



```rust
// A function which takes a closure as an argument and calls it.
// <F> denotes that F is a "Generic type parameter"
fn apply<F>(f: F) where
    // The closure takes no input and returns nothing.
    F: FnOnce() {
    // ^ TODO: Try changing this to `Fn` or `FnMut`.
    // 这里改动就会报错.
    //expected a closure that implements the `Fn` trait,
    //but this closure only implements `FnOnce`

    f();
}

// A function which takes a closure and returns an `i32`.
fn apply_to_3<F>(f: F) -> i32 where
    // The closure takes an `i32` and returns an `i32`.
    F: Fn(i32) -> i32 {

    f(3)
}

fn main() {
    use std::mem;

    let greeting = "hello";
    // A non-copy type.
    // `to_owned` creates owned data from borrowed one
    let mut farewell = "goodbye".to_owned();

    // Capture 2 variables: `greeting` by reference and
    // `farewell` by value.
    let diary = || {
        // `greeting` is by reference: requires `Fn`.
        println!("I said {}.", greeting);

        // Mutation forces `farewell` to be captured by
        // mutable reference. Now requires `FnMut`.
        farewell.push_str("!!!");
        println!("Then I screamed {}.", farewell);
        println!("Now I can sleep. zzzzz");

        // Manually calling drop forces `farewell` to
        // be captured by value. Now requires `FnOnce`.
        mem::drop(farewell);
    };

    // Call the function which applies the closure.
    apply(diary);

    // `double` satisfies `apply_to_3`'s trait bound
    let double = |x| 2 * x;

    println!("3 doubled: {}", apply_to_3(double));
}

```



#### 9.2.3 Type anonymity???

```rust
// `F` must implement `Fn` for a closure which takes no
// inputs and returns nothing - exactly what is required
// for `print`.
fn apply<F>(f: F) where
    F: Fn() {
    f();
}

fn main() {
    let x = 7;

    // Capture `x` into an anonymous type and implement
    // `Fn` for it. Store it in `print`.
    let print = || println!("{}", x);

    apply(print);
}

```

当一个闭包定义时,编译器隐式地创建一个新的匿名结构，其中存储捕获的变量,同时通过这个未知的类型实现这种功能中的一个特征:Fn, FnMut或FnOnce。

这种类型是先分配给变量存储，直到调用才能确定是哪一个。???



参考：

[A thorough analysis](https://huonw.github.io/blog/2015/05/finding-closure-in-rust/), [`Fn`](https://doc.rust-lang.org/std/ops/trait.Fn.html), [`FnMut`](https://doc.rust-lang.org/std/ops/trait.FnMut.html), and [`FnOnce`](https://doc.rust-lang.org/std/ops/trait.FnOnce.html)



#### 9.2.4 Input functions

Fn, FnMut和FnOnce traits 决定一个闭包捕获的变量范围。

```rust
// Define a function which takes a generic `F` argument
// bounded by `Fn`, and calls it
fn call_me<F: Fn()>(f: F) {
    f();
}

// Define a wrapper function satisfying the `Fn` bound
fn function() {
    println!("I'm a function!");
}

fn main() {
    // Define a closure satisfying the `Fn` bound
    let closure = || println!("I'm a closure!");

    call_me(closure);
    call_me(function);
}

```

#### 9.2.5 As output parameters

闭包作为输入参数是可行的,所以返回输出闭包也应该是可能的。

然而,闭包匿名类型在定义的时候是未知的,

所以我们必须使用impl trait来return。

闭包返回的有效traits是:

- `Fn`
- `FnMut`
- `FnOnce`

除此之外,move关键字必须使用，用来指示，所有的捕获采用By Value.

这是必需的,因为任何通过引用的捕获,在函数退出时候将会立即调用Drop()，使其离开闭包时，保证其引用无效。？？？

```rust
fn create_fn() -> impl Fn() {
    let text = "Fn".to_owned();

    move || println!("This is a: {}", text)
}

fn create_fnmut() -> impl FnMut() {
    let text = "FnMut".to_owned();

    move || println!("This is a: {}", text)
}

fn create_fnonce() -> impl FnOnce() {
    let text = "FnOnce".to_owned();

    move || println!("This is a: {}", text)
}

fn main() {
    let fn_plain = create_fn();
    let mut fn_mut = create_fnmut();
    let fn_once = create_fnonce();

    fn_plain();
    fn_mut();
    fn_once(); //只能调用一次，为什么？
}

```

#### 9.2.6 Examples in std

##### 9.2.6.1 Iterator::any

##### 9.2.6.2 Searching through iterators



### 

## 14.泛型

泛型是采用统一的逻辑处理不同的参数类型。

泛型通常写作<T>.

例如，定义一个名字为foo的泛型函数，它有一个可以为任何类型的T参数：

```rust
fn foo<T>(arg: T) { ... }
```

下面的例子，解析了更多的语法说明：

```rust
// A concrete type `A`.
// A是实体类型，即不是泛型。实体类型(concrete type)和泛型(generic type)表达的意思相反；
struct A;

// In defining the type `Single`, the first use of `A` is not preceded by `<A>`.
// Therefore, `Single` is a concrete type, and `A` is defined as above.
struct Single(A);
//            ^ Here is `Single`s first use of the type `A`.

// Here, `<T>` precedes the first use of `T`, so `SingleGen` is a generic type.
// Because the type parameter `T` is generic, it could be anything, including
// the concrete type `A` defined at the top.
struct SingleGen<T>(T);

//注解：以上的注释可以理解为，区分是否为泛型，可以看使用的参数，是否之前在<>尖括号中使用。


fn main() {
    // `Single` is concrete and explicitly takes `A`.
    let _s = Single(A);
  	//根据注解，可以认定Single是实体类型，非泛型；
    
    // Create a variable `_char` of type `SingleGen<char>`
    // and give it the value `SingleGen('a')`.
    // Here, `SingleGen` has a type parameter explicitly specified.
  	// SingleGen显式的指定了参数的类型，即char；
    let _char: SingleGen<char> = SingleGen('a');
  	//这里还可以写成，和C++不同的是，前面多了两个冒号。
    let _char = SingleGen::<char>('a');

  

    // `SingleGen` can also have a type parameter implicitly specified:
  	// SingleGen也可以隐式的指定参数，由编译器推导；
    let _t    = SingleGen(A); // Uses `A` defined at the top.
    let _i32  = SingleGen(6); // Uses `i32`.
    let _char = SingleGen('a'); // Uses `char`.
}

```



### 14.1 函数

使用泛型函数有时需要显示的指定参数类型，例如，如果函数返回的类型是泛型或者编译器没有足够的信息来推导出参数的类型。

函数调用，可以像这样：fun::<A,B,...> 显示的指定类型参数。

例如：

```rust
struct A;          // Concrete type `A`.
struct S(A);       // Concrete type `S`.
struct SGen<T>(T); // Generic type `SGen`.

//注意这里：concrete和generic表达的意思；

// The following functions all take ownership of the variable passed into
// them and immediately go out of scope, freeing the variable.
// 下面的函数都是拿走传递给他的参数的对象的所有权，并且在超出作用域后立即释放。

// 定义一个reg_fn函数，它有一个S类型的_S参数
// 因为它没有<T>,所以它不是一个泛型函数。
// Define a function `reg_fn` that takes an argument `_s` of type `S`.
// This has no `<T>` so this is not a generic function.
fn reg_fn(_s: S) {}

// 定义一个gen_spec_t函数，它有一个SGen<T>类型的_S参数。
// 它显示的给定了一个参数类型A，因为A没有被显示的制定为gen_spec_t的泛型类型参数，所以 // 它不是泛型。

// Define a function `gen_spec_t` that takes an argument `_s` of type `SGen<T>`.
// It has been explicitly given the type parameter `A`, but because `A` has not 
// been specified as a generic type parameter for `gen_spec_t`, it is not generic.
fn gen_spec_t(_s: SGen<A>) {}

// 同上
// Define a function `gen_spec_i32` that takes an argument `_s` of type `SGen<i32>`.
// It has been explicitly given the type parameter `i32`, which is a specific type.
// Because `i32` is not a generic type, this function is also not generic.
fn gen_spec_i32(_s: SGen<i32>) {}

// 因为SGen<T>类型，在之前被<T>定义，所以这个函数是关于T的泛型函数。

// Define a function `generic` that takes an argument `_s` of type `SGen<T>`.
// Because `SGen<T>` is preceded by `<T>`, this function is generic over `T`.
fn generic<T>(_s: SGen<T>) {}

fn main() {
    // Using the non-generic functions
    reg_fn(S(A));          // Concrete type.
    gen_spec_t(SGen(A));   // Implicitly specified type parameter `A`.
    gen_spec_i32(SGen(6)); // Implicitly specified type parameter `i32`.

    // Explicitly specified type parameter `char` to `generic()`.
  	// 为generic()显示指定char类型参数.
  
    generic::<char>(SGen('a'));

  
    // Implicitly specified type parameter `char` to `generic()`.
    // 为generic()隐示指定char类型参数.
    generic(SGen('c'));
}

```





### 14.2 实现（implementation)

主要是为struct或者enum添加方法时，需要考虑泛型参数。

例如：

```rust

#![allow(unused)]
fn main() {
struct S; // Concrete type `S`
struct GenericVal<T>(T); // Generic type `GenericVal`

// impl of GenericVal where we explicitly specify type parameters:
impl GenericVal<f32> {} // Specify `f32`
impl GenericVal<S> {} // Specify `S` as defined above

// `<T>` Must precede the type to remain generic
impl<T> GenericVal<T> {}
}

```



```rust
struct Val {
    val: f64,
}

struct GenVal<T> {
    gen_val: T,
}

// impl of Val
impl Val {
    fn value(&self) -> &f64 {
        &self.val
    }
}

// impl of GenVal for a generic type `T`
// 为泛型类Genval实现value方法，impl携带了T参数，所以这是一个泛型函数。
impl<T> GenVal<T> {
    fn value(&self) -> &T {
        &self.gen_val
    }
}

fn main() {
    let x = Val { val: 3.0 };
    let y = GenVal { gen_val: 3i32 };

    println!("{}, {}", x.value(), y.value());
}

```



### 14.3 Traits

trait也可以泛型化，这里我们定义了一个，重新实现了Drop trait，作为一个泛型方法drop它自己和输入。

```rust
// Non-copyable types.
// 非拷贝类型，即类型只能move，不能copy
struct Empty;
struct Null;


// A trait generic over `T`.
trait DoubleDrop<T> {
    // Define a method on the caller type which takes an
    // additional single parameter `T` and does nothing with it.
    fn double_drop(self, _: T);
}

// Implement `DoubleDrop<T>` for any generic parameter `T` and
// caller `U`.
impl<T, U> DoubleDrop<T> for U {
    // This method takes ownership of both passed arguments,
    // deallocating both.
    // 这个方法获取了传入的两个参数的生命周期，并且在离开作用域时，将其释放；
    fn double_drop(self, _: T) {}
}

fn main() {
    let empty = Empty;
    let null  = Null;

    // Deallocate `empty` and `null`.
    // 释放empty和null，生命周期结束，后面无法使用；
    empty.double_drop(null);
  

    //empty;
    //null;
    // ^ TODO: Try uncommenting these lines.
}

```



### 14.4 Bounds

当使用泛型的时候，类型参数经常必须使用traits作为bounds来规定，类型必须要实现什么功能。下面的例子说明了，使用Display traits来打印，T被bounds为Display，T需要实现Display。

```rust
// Define a function `printer` that takes a generic type `T` which
// must implement trait `Display`.

fn printer<T: Display>(t: T) {
    println!("{}", t);
}

```



```rust
struct S<T: Display>(T);

// Error! `Vec<T>` does not implement `Display`. This
// specialization will fail.
// 因为Vec<T>没有实现Display，所以这个特化会失败；
let s = S(vec![1]);

```



bounding的另外一个作用是，泛型实例可以被允许访问bounds里，traits定义的方法。

例如：

```rust
// A trait which implements the print marker: `{:?}`.
use std::fmt::Debug;

trait HasArea {
    fn area(&self) -> f64;
}

impl HasArea for Rectangle {
    fn area(&self) -> f64 { self.length * self.height }
}

#[derive(Debug)]
struct Rectangle { length: f64, height: f64 }
#[allow(dead_code)]
struct Triangle  { length: f64, height: f64 }

// The generic `T` must implement `Debug`. Regardless
// of the type, this will work properly.
fn print_debug<T: Debug>(t: &T) {
    println!("{:?}", t);
}

// `T` must implement `HasArea`. Any type which meets
// the bound can access `HasArea`'s function `area`.
fn area<T: HasArea>(t: &T) -> f64 { t.area() }

fn main() {
    let rectangle = Rectangle { length: 3.0, height: 4.0 };
    let _triangle = Triangle  { length: 3.0, height: 4.0 };

    print_debug(&rectangle);
    println!("Area: {}", rectangle.area());

    //print_debug(&_triangle);
    //println!("Area: {}", _triangle.area());
    // ^ TODO: Try uncommenting these.
    // | Error: Does not implement either `Debug` or `HasArea`. 
    // 这里出错是因为Rectangle没有实现HasArea traits.
}

```

另外多说一句，在一些场景下，where子句也可以被用于使用bounds，它更加具有表达力。

#### 14.4.1 Testcase: empty bounds

即使traits没有包含任何方法，你也可以使用它作为bounds，在std库中的Eq和Copy就是例子。

例如：

```rust
struct Cardinal;
struct BlueJay;
struct Turkey;

trait Red {}
trait Blue {}

impl Red for Cardinal {}
impl Blue for BlueJay {}

// These functions are only valid for types which implement these
// traits. The fact that the traits are empty is irrelevant.
fn red<T: Red>(_: &T)   -> &'static str { "red" }
fn blue<T: Blue>(_: &T) -> &'static str { "blue" }

fn main() {
    let cardinal = Cardinal;
    let blue_jay = BlueJay;
    let _turkey   = Turkey;

    // `red()` won't work on a blue jay nor vice versa
    // because of the bounds.
    // red（）和blue（）有明确的Bounds限定，不能随意调用；
    println!("A cardinal is {}", red(&cardinal));
    println!("A blue jay is {}", blue(&blue_jay));
    //println!("A turkey is {}", red(&_turkey));
    // Turkey 没有实现 Red traits，所以会出错；
    // ^ TODO: Try uncommenting this line.
}

```



### 14.5 Multiple bounds

为单个类型应用多个bounds，可以使用+加号，不同的类型以，逗号分隔开。

例如：

```rust
use std::fmt::{Debug, Display};
// 类型T被要求实现Debug和Display traits
fn compare_prints<T: Debug + Display>(t: &T) {
    println!("Debug: `{:?}`", t);
    println!("Display: `{}`", t);
}

//类型T和类型U被要求实现Debug traits
fn compare_types<T: Debug, U: Debug>(t: &T, u: &U) {
    println!("t: `{:?}`", t);
    println!("u: `{:?}`", u);
}

fn main() {
    let string = "words";
    let array = [1, 2, 3];
    let vec = vec![1, 2, 3];

    compare_prints(&string);
    //compare_prints(&array);
    // TODO ^ Try uncommenting this.
    // array没有实现Display traits，所以会报错。

    compare_types(&array, &vec);
}

```



### 14.6 Where子句

bound也可以在 { 前、通过使用where子句表达，它可以应用到任何类型参数上。

在以下场景下，使用where更加有用：

- 当分别指定不同的泛型类型和bounds时：

```rust
impl <A: TraitB + TraitC, D: TraitE + TraitF> MyTrait<A, D> for YourType {}

// Expressing bounds with a `where` clause
impl <A, D> MyTrait<A, D> for YourType where
    A: TraitB + TraitC,
    D: TraitE + TraitF {}

```

- 当使用where比一般语法更加有表现力时。以下这个例子在不使用where时，将无法直接表达。

```rust
use std::fmt::Debug;

trait PrintInOption {
    fn print_in_option(self);
}

// Because we would otherwise have to express this as `T: Debug` or 
// use another method of indirect approach, this requires a `where` clause:

impl<T> PrintInOption for T where
    Option<T>: Debug {
    // We want `Option<T>: Debug` as our bound because that is what's
    // being printed. Doing otherwise would be using the wrong bound.
    // 关键点在于，因为我们需要将Option<T>: Debug作为bound，所以通过一般的方法是无法实现，
    // 只能通过where才能实现，这里可以看到where比一般语法的长处。
      
    fn print_in_option(self) {
        println!("{:?}", Some(self));
    }
}

fn main() {
    let vec = vec![1, 2, 3];

    vec.print_in_option();
}

```



### 14.7 New Type Idiom

Idiom可以理解为习惯用法，即我们口语中的成语、方言。

New type习惯用法，在编译时，确保给程序的值，是正确的类型。

例如：

```rust
struct Years(i64);

struct Days(i64);

impl Years {
    pub fn to_days(&self) -> Days {
        Days(self.0 * 365)
    }
}

impl Days {
    /// truncates partial years
    pub fn to_years(&self) -> Years {
        Years(self.0 / 365)
    }
}

fn old_enough(age: &Years) -> bool {
    age.0 >= 18
}

fn main() {
    let age = Years(5);
    let age_days = age.to_days();
    println!("Old enough {}", old_enough(&age));
    println!("Old enough {}", old_enough(&age_days.to_years()));
    // println!("Old enough {}", old_enough(&age_days));
    // 错误信息：
    // ^^^^^^^^^ expected struct `Years`, found struct `Days`
}

```



### 14.8 Associated items

#### 14.8.1 The Problem

问题的提出，参看如下例子：

```rust
struct Container(i32, i32);

// A trait which checks if 2 items are stored inside of container.
// Also retrieves first or last value.
trait Contains<A, B> {
    fn contains(&self, _: &A, _: &B) -> bool; // Explicitly requires `A` and `B`.
    fn first(&self) -> i32; // Doesn't explicitly require `A` or `B`.
    fn last(&self) -> i32;  // Doesn't explicitly require `A` or `B`.
}

impl Contains<i32, i32> for Container {
    // True if the numbers stored are equal.
    fn contains(&self, number_1: &i32, number_2: &i32) -> bool {
        (&self.0 == number_1) && (&self.1 == number_2)
    }

    // Grab the first number.
    fn first(&self) -> i32 { self.0 }

    // Grab the last number.
    fn last(&self) -> i32 { self.1 }
}

// `C` contains `A` and `B`. In light of that, having to express `A` and
// `B` again is a nuisance.

// 问题：这个函数定义的特别麻烦，而且阅读起来，也不清晰。
fn difference<A, B, C>(container: &C) -> i32 where
    C: Contains<A, B> {
    container.last() - container.first()
}

fn main() {
    let number_1 = 3;
    let number_2 = 10;

    let container = Container(number_1, number_2);

    println!("Does container contain {} and {}: {}",
        &number_1, &number_2,
        container.contains(&number_1, &number_2));
    println!("First number: {}", container.first());
    println!("Last number: {}", container.last());

    println!("The difference is: {}", difference(&container));
}

```



#### 14.8.2 Associated types

使用关联类型（Associated types），将内置的类作为traits的输出类型，移动到traits的定义内部， 从而提升了代码的整体可读性，traits的定义语法如下：

```rust

#![allow(unused)]
fn main() {
// `A` and `B` are defined in the trait via the `type` keyword.
// (Note: `type` in this context is different from `type` when used for
// aliases).
// 注意：这里的type不同于使用type定义的别名；
trait Contains {
    type A;
    type B;

    // Updated syntax to refer to these new types generically.
    fn contains(&self, _: &Self::A, _: &Self::B) -> bool;
}
}

```

注意，函数的定义不在需要使用A 或者B：

例如：

```rust
//不使用关联类型
// Without using associated types
fn difference<A, B, C>(container: &C) -> i32 where
    C: Contains<A, B> { ... }

// 使用关联类型简化版
// Using associated types
fn difference<C: Contains>(container: &C) -> i32 { ... }

```

现在，让我们使用关联类型，重新写之前的例子：

```rust
struct Container(i32, i32);

// A trait which checks if 2 items are stored inside of container.
// Also retrieves first or last value.
trait Contains {
    // Define generic types here which methods will be able to utilize.
    type A;
    type B;
    // 关键点：使用输出类型A和B，定义了关联类型；
    fn contains(&self, _: &Self::A, _: &Self::B) -> bool;
  	
    fn first(&self) -> i32;
    fn last(&self) -> i32;
}

impl Contains for Container {
    // Specify what types `A` and `B` are. If the `input` type
    // is `Container(i32, i32)`, the `output` types are determined
    // as `i32` and `i32`.
    // 初始化关联类型
    type A = i32;
    type B = i32;

    // `&Self::A` and `&Self::B` are also valid here.
    // 函数中，使用关联类型定义输入参数.
    fn contains(&self, number_1: &i32, number_2: &i32) -> bool {
        (&self.0 == number_1) && (&self.1 == number_2)
    }
    // Grab the first number.
    fn first(&self) -> i32 { self.0 }

    // Grab the last number.
    fn last(&self) -> i32 { self.1 }
}

fn difference<C: Contains>(container: &C) -> i32 {
    container.last() - container.first()
}

fn main() {
    let number_1 = 3;
    let number_2 = 10;

    let container = Container(number_1, number_2);

    println!("Does container contain {} and {}: {}",
        &number_1, &number_2,
        container.contains(&number_1, &number_2));
    println!("First number: {}", container.first());
    println!("Last number: {}", container.last());
    
    println!("The difference is: {}", difference(&container));
}

```



### 14.9 Phantom type parameters

Phantom可以理解为幻影，幽灵。

笔者认为可以是语法层面，用来使编译器编译不在报错的一种技巧，具备一定的语义。

有点类似占位符。

“A phantom type parameter is one that doesn't show up at runtime, but is checked statically (and only) at compile time.“

-- phantom类型是一种不会出现在运行时（runtime），但是（只会）在编译时（compile time）会被静态检查。

数据类型在编译时，可以使用额外的泛型类型参数作为标志，对类型参数进行检查。

这些额外的参数没有hold存储值，并且没有runtime开销。

下面这个例子，我们组合使用std::marker::PhantomData作为类型参数，来创建包含不同数据类型的tuple。

```rust
use std::marker::PhantomData;

// A phantom tuple struct which is generic over `A` with hidden parameter `B`.
#[derive(PartialEq)] // Allow equality test for this type.
struct PhantomTuple<A, B>(A, PhantomData<B>);
// 这里的PhantomTuple中的<B>类型,标记为PhantomData,需要用PhantomData来初始化.
// 相当于占位符.




// A phantom type struct which is generic over `A` with hidden parameter `B`.
#[derive(PartialEq)] // Allow equality test for this type.
struct PhantomStruct<A, B> { first: A, phantom: PhantomData<B> }
// 这里的PhantomTuple中的<B>类型,标记为PhantomData,需要用PhantomData来初始化.
// 相当于占位符.

// Note: Storage is allocated for generic type `A`, but not for `B`.
//       Therefore, `B` cannot be used in computations.

fn main() {
    // Here, `f32` and `f64` are the hidden parameters.
    // PhantomTuple type specified as `<char, f32>`.
    let _tuple1: PhantomTuple<char, f32> = PhantomTuple('Q', PhantomData);
    // PhantomTuple type specified as `<char, f64>`.
    let _tuple2: PhantomTuple<char, f64> = PhantomTuple('Q', PhantomData);

    // Type specified as `<char, f32>`.
    let _struct1: PhantomStruct<char, f32> = PhantomStruct {
        first: 'Q',
        phantom: PhantomData,
    };
  
    // Type specified as `<char, f64>`.
    let _struct2: PhantomStruct<char, f64> = PhantomStruct {
        first: 'Q',
        phantom: PhantomData,
    };

    // Compile-time Error! Type mismatch so these cannot be compared:
    // println!("_tuple1 == _tuple2 yields: {}",
    //           _tuple1 == _tuple2);

    // Compile-time Error! Type mismatch so these cannot be compared:
    // println!("_struct1 == _struct2 yields: {}",
    //           _struct1 == _struct2);
}

```



  	/*编译出错信息：
  	   |
  	34 |               _tuple1 == _tuple2);
  	   |                          ^^^^^^^ expected `f32`, found `f64`
  	   |
  	   = note: expected struct `PhantomTuple<_, f32>`
  	              found struct `PhantomTuple<_, f64>`	
  	
  	*/



#### 14.9.1 Testcase:unit clarification





## 15.Scopeing rules

在所有权，借用，和生命周期中，Scopes扮演着重要的部分。这是因为，它指示编译器什么时候借用是有效的，什么时候
资源可以被释放掉，什么时候变量被创建或者销毁。

### 15.1 RAII

变量在rust中不仅仅是栈中存储数据，他们也是资源的owner ，也拥有资源。

例如Box<T>在堆中拥有内存数据。rust强制RAII(Resource Acquisition Is Initialization 资源获取即初始化),所以当一个对象离开作用域，它的析构器将被调用，并且它拥有的资源将被释放掉。这种行为防止了资源泄露的Bug。所以你从来不需要手动的释放内存或者担心内存泄露。下面是一个快速的展示：

```rust
// raii.rs
fn create_box() {
    // Allocate an integer on the heap
    let _box1 = Box::new(3i32);

    // `_box1` is destroyed here, and memory gets freed
}

fn main() {
    // Allocate an integer on the heap
    let _box2 = Box::new(5i32);

    // A nested scope:
    {
        // Allocate an integer on the heap
        let _box3 = Box::new(4i32);

        // `_box3` is destroyed here, and memory gets freed
    }

    // Creating lots of boxes just for fun
    // There's no need to manually free memory!
    for _ in 0u32..1_000 {
        create_box();
    }

    // `_box2` is destroyed here, and memory gets freed
}

```

当然，我们可以使用valgrind来再次确认内存错误：

```shell
$ rustc raii.rs && valgrind ./raii
==26873== Memcheck, a memory error detector
==26873== Copyright (C) 2002-2013, and GNU GPL'd, by Julian Seward et al.
==26873== Using Valgrind-3.9.0 and LibVEX; rerun with -h for copyright info
==26873== Command: ./raii
==26873==
==26873==
==26873== HEAP SUMMARY:
==26873==     in use at exit: 0 bytes in 0 blocks
==26873==   total heap usage: 1,013 allocs, 1,013 frees, 8,696 bytes allocated
==26873==
==26873== All heap blocks were freed -- no leaks are possible
==26873==
==26873== For counts of detected and suppressed errors, rerun with: -v
==26873== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 2 from 2)

```



**Destructor**

在rust中析构器destructor的概念是通过Drop trait来提供的。当资源离开作用域，析构器将被调用这个trait不是每个类型都需要实现，只需要为自定义类型并且需要自定义的析构逻辑。

下面的例子展示了Drop trait是如何工作的。但变量离开main函数的时候，自定义的析构器将会被调用。

```rust
struct ToDrop;

impl Drop for ToDrop {
    fn drop(&mut self) {
        println!("ToDrop is being dropped");
    }
}

fn main() {
    let x = ToDrop;
    println!("Made a ToDrop!");
}

```

### 15.2 所有权和转移（Ownership and moves）

因为变量负责释放她们拥有的资源，所以资源只能有一个owner。这也阻止了资源被释放的次数多于一次。注意，不是所有的变量都拥有资源。

当做赋值(let x=y)或者通过值传递函数参数(foo(x))，资源的ownership被转移，在rust中叫做move。

在moving资源之后，先前的owner不能再使用。这样就避免了创建悬挂指针。

```rust
// This function takes ownership of the heap allocated memory
fn destroy_box(c: Box<i32>) {
    println!("Destroying a box that contains {}", c);

    // `c` is destroyed and the memory freed
}

fn main() {
    // _Stack_ allocated integer
    let x = 5u32;

    // *Copy* `x` into `y` - no resources are moved
    let y = x;

    // Both values can be independently used
    println!("x is {}, and y is {}", x, y);

    // `a` is a pointer to a _heap_ allocated integer
    let a = Box::new(5i32);

    println!("a contains: {}", a);

    // *Move* `a` into `b`
    let b = a;
    // The pointer address of `a` is copied (not the data) into `b`.
    // Both are now pointers to the same heap allocated data, but
    // `b` now owns it.
    
    // Error! `a` can no longer access the data, because it no longer owns the
    // heap memory
    //println!("a contains: {}", a);
    // TODO ^ Try uncommenting this line

    // This function takes ownership of the heap allocated memory from `b`
    destroy_box(b);

    // Since the heap memory has been freed at this point, this action would
    // result in dereferencing freed memory, but it's forbidden by the compiler
    // Error! Same reason as the previous Error
    //println!("b contains: {}", b);
    // TODO ^ Try uncommenting this line
}

```

#### 15.2.1 Mutability(可变性)

数据的可变性是指，数据的ownership被转移后，数据可以被修改。

```rust
fn main() {
    let immutable_box = Box::new(5u32);

    println!("immutable_box contains {}", immutable_box);

    // Mutability error
    //*immutable_box = 4;

    // *Move* the box, changing the ownership (and mutability)
    let mut mutable_box = immutable_box;

    println!("mutable_box contains {}", mutable_box);

    // Modify the contents of the box
    *mutable_box = 4;

    println!("mutable_box now contains {}", mutable_box);
}

```

#### 15.2.2 Partial moves

在解构单一变量时，同时，有两种模式binding都可以使用，by-move和by-reference.
这样做会导致变量 partial move ，这意味着，变量的一部分会被move，而其他部分将会保留不动。
在这种情况下，父变量不能被作为一个整体被使用，尽管这样，被引用的部分仍然能够被使用。

```rust
fn main() {
    #[derive(Debug)]
    struct Person {
        name: String,
        age: Box<u8>,
    }

    let person = Person {
        name: String::from("Alice"),
        age: Box::new(20),
    };

    // `name` is moved out of person, but `age` is referenced
    let Person { name, ref age } = person;

    println!("The person's age is {}", age);

    println!("The person's name is {}", name);

    // Error! borrow of partially moved value: `person` partial move occurs
    //println!("The person struct is {:?}", person);

    // `person` cannot be used but `person.age` can be used as it is not moved
    println!("The person's age from person struct is {}", person.age);
}

```

(在这个例子中，我们将age变量存储在堆(heap)中来展示partial move，删除ref后将会导致编译错误“ the ownership of person.age would be moved to the variable age.”，如果Person.age 存储在(栈)stack ，ref关键字将不再需要，，因为age只是从person.age 拷贝数据，而不是移动它。)

### 15.3 Borrowing(借用)

很多时候，我们想访问数据，但是不想拿走它的owner，为了达到此目的，Rust采用了借用（borrowing）机制。对象的传递使用引用（&T）替代传值（T）。

编译器静态的保证（通过借用检查），引用总是指向有效对象，这就是说，当一个一个对象被引用时，它将不能被销毁。

```rust
// This function takes ownership of a box and destroys it
fn eat_box_i32(boxed_i32: Box<i32>) {
    println!("Destroying box that contains {}", boxed_i32);
}

// This function borrows an i32
fn borrow_i32(borrowed_i32: &i32) {
    println!("This int is: {}", borrowed_i32);
}

fn main() {
    // Create a boxed i32, and a stacked i32
    let boxed_i32 = Box::new(5_i32);
    let stacked_i32 = 6_i32;

    // Borrow the contents of the box. Ownership is not taken,
    // so the contents can be borrowed again.
    
    borrow_i32(&boxed_i32);
    borrow_i32(&stacked_i32);

    {
        // Take a reference to the data contained inside the box
        let _ref_to_i32: &i32 = &boxed_i32;

        // Error!
        // Can't destroy `boxed_i32` while the inner value is borrowed later in scope.
        // eat_box_i32(boxed_i32);
      
        //这个地方调用出错的原因是，后面_ref_to_i32还引用了boxed_i32,所以         
        // boxed_i32 不能被销毁。
      
        // FIXME ^ Comment out this line

        // Attempt to borrow `_ref_to_i32` after inner value is destroyed
        borrow_i32(_ref_to_i32);
        // `_ref_to_i32` goes out of scope and is no longer borrowed.
    }

    // `boxed_i32` can now give up ownership to `eat_box` and be destroyed
    eat_box_i32(boxed_i32);
}

```

#### 15.3.1 Mutability（可变）

可变数据可以通过使用&mut T来可变的借用，这称为可变引用，给予借用者以读/写权限,

相反，&T通过一个非可变的引用来借用数据。借用者可以读取数据，但是不能修改它。

```rust
#[allow(dead_code)]
#[derive(Clone, Copy)]
struct Book {
    // `&'static str` is a reference to a string allocated in read only memory
    author: &'static str,
    title: &'static str,
    year: u32,
}

// This function takes a reference to a book
fn borrow_book(book: &Book) {
    println!("I immutably borrowed {} - {} edition", book.title, book.year);
}

// This function takes a reference to a mutable book and changes `year` to 2014
fn new_edition(book: &mut Book) {
    book.year = 2014;
    println!("I mutably borrowed {} - {} edition", book.title, book.year);
}

fn main() {
    // Create an immutable Book named `immutabook`
    let immutabook = Book {
        // string literals have type `&'static str`
        author: "Douglas Hofstadter",
        title: "Gödel, Escher, Bach",
        year: 1979,
    };

    // Create a mutable copy of `immutabook` and call it `mutabook`
    let mut mutabook = immutabook;
    
    // Immutably borrow an immutable object
    // 以不可变的方式借用不可变对象。有点绕口，仔细想想～
    borrow_book(&immutabook);

    // Immutably borrow a mutable object
    // 以不可变的方式借用可变对象。
    borrow_book(&mutabook);
    
    // Borrow a mutable object as mutable
    // 以可变的方式借用可变对象
    new_edition(&mut mutabook);
    
    // Error! Cannot borrow an immutable object as mutable
    // 以可变的方式借用不可变对象，报错了～～～
    new_edition(&mut immutabook);
    // FIXME ^ Comment out this line
}
```

#### 15.3.2 Aliasing

数据可以以不可变的方式借用任意次数，但在不可变数据借用期间，原始数据不能被可变的借用。另一方面，在任一时间，只允许有一个可变借用存在，原始数据只可以在可变引用最后使用完后，再次被借用。

```rust
struct Point { x: i32, y: i32, z: i32 }

fn main() {
    let mut point = Point { x: 0, y: 0, z: 0 };

    let borrowed_point = &point;
    let another_borrow = &point;

    // Data can be accessed via the references and the original owner
    println!("Point has coordinates: ({}, {}, {})",
                borrowed_point.x, another_borrow.y, point.z);

    // Error! Can't borrow `point` as mutable because it's currently
    // borrowed as immutable.
    // let mutable_borrow = &mut point;
    // TODO ^ Try uncommenting this line

    // The borrowed values are used again here
    println!("Point has coordinates: ({}, {}, {})",
                borrowed_point.x, another_borrow.y, point.z);

    // The immutable references are no longer used for the rest of the code so
    // it is possible to reborrow with a mutable reference.
    let mutable_borrow = &mut point;

    // Change data via mutable reference
    mutable_borrow.x = 5;
    mutable_borrow.y = 2;
    mutable_borrow.z = 1;

    // Error! Can't borrow `point` as immutable because it's currently
    // borrowed as mutable.
    // let y = &point.y;
    // TODO ^ Try uncommenting this line

    // Error! Can't print because `println!` takes an immutable reference.
    // println!("Point Z coordinate is {}", point.z);
    // TODO ^ Try uncommenting this line

    // Ok! Mutable references can be passed as immutable to `println!`
    println!("Point has coordinates: ({}, {}, {})",
                mutable_borrow.x, mutable_borrow.y, mutable_borrow.z);

    // The mutable reference is no longer used for the rest of the code so it
    // is possible to reborrow
    let new_borrowed_point = &point;
    println!("Point now has coordinates: ({}, {}, {})",
             new_borrowed_point.x, new_borrowed_point.y, new_borrowed_point.z);
}

```



#### 15.3.3 The ref pattern

当通过let绑定进行模式匹配或解构时,ref关键字可以被用来引用结构或者元组的字段。

下面的例子显示了一个有用的实例。

```rust
#[derive(Clone, Copy)]
struct Point { x: i32, y: i32 }

fn main() {
    let c = 'Q';

    // A `ref` borrow on the left side of an assignment is equivalent to
    // an `&` borrow on the right side.
    let ref ref_c1 = c;
    let ref_c2 = &c;

    println!("ref_c1 equals ref_c2: {}", *ref_c1 == *ref_c2);

    let point = Point { x: 0, y: 0 };

    // `ref` is also valid when destructuring a struct.
    let _copy_of_x = {
        // `ref_to_x` is a reference to the `x` field of `point`.
        let Point { x: ref ref_to_x, y: _ } = point;

        // Return a copy of the `x` field of `point`.
        *ref_to_x
    };

    // A mutable copy of `point`
    let mut mutable_point = point;

    {
        // `ref` can be paired with `mut` to take mutable references.
        // ref mut 这样用怪怪的～～
        let Point { x: _, y: ref mut mut_ref_to_y } = mutable_point;

        // Mutate the `y` field of `mutable_point` via a mutable reference.
        *mut_ref_to_y = 1;
    }

    println!("point is ({}, {})", point.x, point.y);
    println!("mutable_point is ({}, {})", mutable_point.x, mutable_point.y);

    // A mutable tuple that includes a pointer
    let mut mutable_tuple = (Box::new(5u32), 3u32);
    
    {
        // Destructure `mutable_tuple` to change the value of `last`.
        let (_, ref mut last) = mutable_tuple;
        *last = 2u32;
    }
    
    println!("tuple is {:?}", mutable_tuple);
}

```

笔者注：这种使用方式，我觉得很奇葩，但是要能看懂

### 15.4 Lifetimes(生命周期)

生命周期是编译器的一种构想（更确切的说，是一种借用检查器）用来确保所有的借用是有效的，特别是变量的生命周期什么时候开始，什么时候结束。生命周期和作用域经常在一起被引用，但是，它们不是同一个事物。

例如，当我们通过&借用一个变量，借用的生命周期是由它在什么地方声明决定。因此，借用的有效性和借用者被销毁之前一样长？？。尽管这样，借用的作用域由引用被使用的地方决定。

接下来的例子和剩下章节，我们将看到lifetimes和scope的关系已经它们的差异。

```rust
// Lifetimes are annotated below with lines denoting the creation
// and destruction of each variable.
// `i` has the longest lifetime because its scope entirely encloses 
// both `borrow1` and `borrow2`. The duration of `borrow1` compared 
// to `borrow2` is irrelevant since they are disjoint.
// i 的生命周期最长。borrow1和borrow2不相关。
fn main() {
    let i = 3; // Lifetime for `i` starts. ────────────────┐
    //                                                     │
    { //                                                   │
        let borrow1 = &i; // `borrow1` lifetime starts. ──┐│
        //                                                ││
        println!("borrow1: {}", borrow1); //              ││
    } // `borrow1 ends. ──────────────────────────────────┘│
    //                                                     │
    //                                                     │
    { //                                                   │
        let borrow2 = &i; // `borrow2` lifetime starts. ──┐│
        //                                                ││
        println!("borrow2: {}", borrow2); //              ││
    } // `borrow2` ends. ─────────────────────────────────┘│
    //                                                     │
}   // Lifetime ends. ─────────────────────────────────────┘

```

#### 15.4.1 Explicit annotation

借用检查，使用显示生命周期注释来决定，多长的引用时间才应该是有效的。当生命周期没有被省略，Rust需要显示的注释来决定，引用的生命周期应该是多久。显示标注生命周期，可以像下面一样，使用撇号。

```rust
foo<'a>
// `foo` has a lifetime parameter `'a`

```

和闭包相同，使用生命周期，需要使用泛型。另外，这个生命周期的语法说明，foo的生命周期不会超过‘a。显示的注释一个类型使用&’a T。

多个生命周期的语法为：

```rust
foo<'a, 'b>
// `foo` has lifetime parameters `'a` and `'b`
// foo 既不能超过‘a或者’b
```

以下是显示生命周期注释的例子：

```rust
// `print_refs` takes two references to `i32` which have different
// lifetimes `'a` and `'b`. These two lifetimes must both be at
// least as long as the function `print_refs`.
// 这个函数有两个引用参数，这两个参数有不同的生命周期
// 这个两个生命周期，必须至少和函数print_refs的生命周期一样长。
fn print_refs<'a, 'b>(x: &'a i32, y: &'b i32) {
    println!("x is {} and y is {}", x, y);
}

// A function which takes no arguments, but has a lifetime parameter `'a`.
fn failed_borrow<'a>() {
    let _x = 12;
    // 换成 const  _x:i32 = 12 或者 static _x:i32 = 12 试一下～～

    // ERROR: `_x` does not live long enough
    let y: &'a i32 = &_x;
    // Attempting to use the lifetime `'a` as an explicit type annotation 
    // inside the function will fail because the lifetime of `&_x` is shorter
    // than that of `y`. A short lifetime cannot be coerced into a longer one.
    // &_x的生命周期比y短。一个短的生命周期，不能被强制转为长的生命周期.
}

fn main() {
    // Create variables to be borrowed below.
    let (four, nine) = (4, 9);
    
    // Borrows (`&`) of both variables are passed into the function.
    print_refs(&four, &nine);
    // Any input which is borrowed must outlive the borrower. 
    // In other words, the lifetime of `four` and `nine` must 
    // be longer than that of `print_refs`.
    // four和nine的生命周期必须比print_refs长。
  
    
    failed_borrow();
    // `failed_borrow` contains no references to force `'a` to be 
    // longer than the lifetime of the function, but `'a` is longer.
    // Because the lifetime is never constrained, it defaults to `'static`.
}

```

#### 15.4.2 Functions

函数生命周期签名有如下限制：

- 任何引用必须有一个带注解的生命周期。
- 被返回的任何引用，必须有和输入参数相同的生命周期，或者是static

```rust
// One input reference with lifetime `'a` which must live
// at least as long as the function.
// 参数x必须至少和函数有一样的生命周期。
fn print_one<'a>(x: &'a i32) {
    println!("`print_one`: x is {}", x);
}

// Mutable references are possible with lifetimes as well.
fn add_one<'a>(x: &'a mut i32) {
    *x += 1;
}

// Multiple elements with different lifetimes. In this case, it
// would be fine for both to have the same lifetime `'a`, but
// in more complex cases, different lifetimes may be required.
fn print_multi<'a, 'b>(x: &'a i32, y: &'b i32) {
    println!("`print_multi`: x is {}, y is {}", x, y);
}

// Returning references that have been passed in is acceptable.
// However, the correct lifetime must be returned.
// 返回参数的生命周期必须正确。
fn pass_x<'a, 'b>(x: &'a i32, _: &'b i32) -> &'a i32 { x }

//fn invalid_output<'a>() -> &'a String { &String::from("foo") }
// The above is invalid: `'a` must live longer than the function.
// Here, `&String::from("foo")` would create a `String`, followed by a
// reference. Then the data is dropped upon exiting the scope, leaving
// a reference to invalid data to be returned.
// 这个是一个错误的例子，函数要求返回String，但是实际上离开作用域后，String被销毁，所以会编译出错。

fn main() {
    let x = 7;
    let y = 9;
    
    print_one(&x);
    print_multi(&x, &y);
    
    let z = pass_x(&x, &y);
    print_one(z);

    let mut t = 3;
    add_one(&mut t);
    print_one(&t);
}

```

#### 15.4.3 Mechods

方法的注释和函数类似。

```rust
// One input reference with lifetime `'a` which must live
// at least as long as the function.
fn print_one<'a>(x: &'a i32) {
    println!("`print_one`: x is {}", x);
}

// Mutable references are possible with lifetimes as well.
fn add_one<'a>(x: &'a mut i32) {
    *x += 1;
}

// Multiple elements with different lifetimes. In this case, it
// would be fine for both to have the same lifetime `'a`, but
// in more complex cases, different lifetimes may be required.
fn print_multi<'a, 'b>(x: &'a i32, y: &'b i32) {
    println!("`print_multi`: x is {}, y is {}", x, y);
}

// Returning references that have been passed in is acceptable.
// However, the correct lifetime must be returned.
fn pass_x<'a, 'b>(x: &'a i32, _: &'b i32) -> &'a i32 { x }

//fn invalid_output<'a>() -> &'a String { &String::from("foo") }
// The above is invalid: `'a` must live longer than the function.
// Here, `&String::from("foo")` would create a `String`, followed by a
// reference. Then the data is dropped upon exiting the scope, leaving
// a reference to invalid data to be returned.

fn main() {
    let x = 7;
    let y = 9;
    
    print_one(&x);
    print_multi(&x, &y);
    
    let z = pass_x(&x, &y);
    print_one(z);

    let mut t = 3;
    add_one(&mut t);
    print_one(&t);
}

```



#### 15.4.4 Structs

结构体中，生命周期的注解和函数是类似。

```rust
// A type `Borrowed` which houses a reference to an
// `i32`. The reference to `i32` must outlive `Borrowed`.
// i32 必须活的比Borrowed长.

#[derive(Debug)]
struct Borrowed<'a>(&'a i32);

// Similarly, both references here must outlive this structure.
#[derive(Debug)]
struct NamedBorrowed<'a> {
    x: &'a i32,
    y: &'a i32,
}

// An enum which is either an `i32` or a reference to one.
// 一个要么是i32，要么是i32引用的枚举
#[derive(Debug)]
enum Either<'a> {
    Num(i32),
    Ref(&'a i32),
}

fn main() {
    let x = 18;
    let y = 15;

    let single = Borrowed(&x);
    let double = NamedBorrowed { x: &x, y: &y };
    let reference = Either::Ref(&x);
    let number    = Either::Num(y);

    println!("x is borrowed in {:?}", single);
    println!("x and y are borrowed in {:?}", double);
    println!("x is borrowed in {:?}", reference);
    println!("y is *not* borrowed in {:?}", number);
}

```



#### 15.4.5 Traits

trait中生命周期的注释基本上和函数是相同的。注意，impl也可以有生命周期注释。

```rust
// A struct with annotation of lifetimes.
#[derive(Debug)]
struct Borrowed<'a> {
    x: &'a i32,
}

// Annotate lifetimes to impl.
// 如果省略掉<'a'>,编译器会报错：use of undeclared lifetime name `'a`。
impl<'a> Default for Borrowed<'a> {
    fn default() -> Self {
        Self {
            x: &10,
        }
    }
}

fn main() {
    let b: Borrowed = Default::default();
    println!("b is {:?}", b);
}

```

#### 15.4.6 Bounds（界定）

和泛型类型可以被界定一样，生命周期也可以被（bound）界定，：冒号在这里，有着稍微不同的意义，但是+加号是相同的。留意下面的表达式读法：

1. T:'a  所有的在T中的引用，必须活的比‘a长。
2. T:Trait + 'a  类型T必须实现Trait ，所有在T中的引用，必须活的比‘a长。

下面的例子展示了关键字where的使用：

```rust
use std::fmt::Debug; // Trait to bound with.

#[derive(Debug)]
struct Ref<'a, T: 'a>(&'a T);
// `Ref` contains a reference to a generic type `T` that has
// an unknown lifetime `'a`. `T` is bounded such that any
// *references* in `T` must outlive `'a`. Additionally, the lifetime
// of `Ref` may not exceed `'a`.
// Ref包含了一个泛型类型T的引用，这个引用有一个未知的生命周期‘a。T被（bound）界定，因此任何在T中的引
// 用必须活着的时间比‘a长。

// A generic function which prints using the `Debug` trait.
fn print<T>(t: T) where
    T: Debug {
    println!("`print`: t is {:?}", t);
}

// Here a reference to `T` is taken where `T` implements
// `Debug` and all *references* in `T` outlive `'a`. In
// addition, `'a` must outlive the function.
fn print_ref<'a, T>(t: &'a T) where
    T: Debug + 'a {
    println!("`print_ref`: t is {:?}", t);
}

// 参数中的‘a和T bound的'a,有什么不同呢？
// 'a 不但注解（annotation）T，而且界定(Bounds)T

fn main() {
    let x = 7;
    let ref_x = Ref(&x);

    print_ref(&ref_x);
    print(ref_x);
}

```



#### 15.4.7 Coercion

A longer lifetime can be coerced into a shorter one so that it works inside a scope it normally wouldn't work in. This comes in the form of inferred coercion by the Rust compiler, and also in the form of declaring a lifetime difference:

```rust
// Here, Rust infers a lifetime that is as short as possible.
// The two references are then coerced to that lifetime.
fn multiply<'a>(first: &'a i32, second: &'a i32) -> i32 {
    first * second
}

// `<'a: 'b, 'b>` reads as lifetime `'a` is at least as long as `'b`.
// Here, we take in an `&'a i32` and return a `&'b i32` as a result of coercion.
fn choose_first<'a: 'b, 'b>(first: &'a i32, _: &'b i32) -> &'b i32 {
    first
}

fn main() {
    let first = 2; // Longer lifetime
    
    {
        let second = 3; // Shorter lifetime
        
        println!("The product is {}", multiply(&first, &second));
        println!("{} is the first", choose_first(&first, &second));
    };
}

```

#### 15.4.8 Static

Rust有一些预留的生命周期名字，其中一个是'static.你可能在以下两种场景遇到：

1. ‘static 生命周期 修饰 引用
2. ’static作为trait界定的一部分

```rust
// A reference with 'static lifetime:
let s: &'static str = "hello world";

// 'static as part of a trait bound:
fn generic<T>(x: T) where T: 'static {}

```

它们之间相关但是有着精妙的不同点，这通常会在学习Rust时造成困惑，下面这两种场景使用的一些例子：



**Reference lifetime**

一个被‘static修饰的生命周期的意思是，被引用的数据存活时间为整个程序的运行时间。它可以被强制压塑为较短的生命周期。

有两种方式可以将变量的生命周期标记为‘static,它们都存储在二进制程序的只读内存区：

- 使用static声明一个常量
- 使用&’static str声明一个string字面量

下面的例子演示了这两方式：



```rust
// Make a constant with `'static` lifetime.
static NUM: i32 = 18;

// Returns a reference to `NUM` where its `'static`
// lifetime is coerced to that of the input argument.
fn coerce_static<'a>(_: &'a i32) -> &'a i32 {
    &NUM
}

fn main() {
    {
        // Make a `string` literal and print it:
        let static_string = "I'm in read-only memory";
        println!("static_string: {}", static_string);

        // When `static_string` goes out of scope, the reference
        // can no longer be used, but the data remains in the binary.
    }

    {
        // Make an integer to use for `coerce_static`:
        let lifetime_num = 9;

        // Coerce `NUM` to lifetime of `lifetime_num`:
        let coerced_static = coerce_static(&lifetime_num);

        println!("coerced_static: {}", coerced_static);
    }

    println!("NUM: {} stays accessible!", NUM);
}

```

**Trait bound**

作为trait的界定，它意识是，类型不会包含任何非static的引用。接收者可以想多久就多久的控制类型数据，它永远不会变得无效直到它们drop他。

需要明白的最要一点是，被‘static声明周期界定的输入，**拥有数据**，但是引用通常不会。

```rust
use std::fmt::Debug;

fn print_it( input: impl Debug + 'static ) {
    println!( "'static value passed in is: {:?}", input );
}

fn main() {
    // i is owned and contains no references, thus it's 'static:
    let i = 5;
    print_it(i);

    // oops, &i only has the lifetime defined by the scope of
    // main(), so it's not 'static:
    // &i的生命周只限定在main()中，离开main()后即Drop掉，变得无效。
    // 但是print_it需要的是’static,所以编译器会报错。
    print_it(&i);
}

```

#### 15.4.9 Elision（省略）

一些生命周期模式非常常见，所以借用检查器允许你忽略他们以减少打字频率和提高阅读性。

下面的例子演示了elision的用法，更加复杂的用法可以参考[lifetime elision](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-elision) 

```rust
// `elided_input` and `annotated_input` essentially have identical signatures
// because the lifetime of `elided_input` is inferred by the compiler:
fn elided_input(x: &i32) {
    println!("`elided_input`: {}", x);
}

fn annotated_input<'a>(x: &'a i32) {
    println!("`annotated_input`: {}", x);
}

// Similarly, `elided_pass` and `annotated_pass` have identical signatures
// because the lifetime is added implicitly to `elided_pass`:
fn elided_pass(x: &i32) -> &i32 { x }

fn annotated_pass<'a>(x: &'a i32) -> &'a i32 { x }

fn main() {
    let x = 3;

    elided_input(&x);
    annotated_input(&x);

    println!("`elided_pass`: {}", elided_pass(&x));
    println!("`annotated_pass`: {}", annotated_pass(&x));
}

```

生命周期省略的三个规则

- 编译器使用3个规则在没有显示标志生命周期的情况下，来确定引用的生命周期
  - 规则一 应用于输入生命周期
  - 规则二、三应用于输出生命周期
  - 如果编译器应用完3个规则之后，仍然无法确定生命周期的引用，则报错
  - 这些规则适用于fn定义和impl块
- 规则一 每个引用类型的参数都有自己的生命周期
- 规则二 如果只有一个输入生命周期参数，那么该生命周期被赋给所有的输出生命周期参数
- 规则三 如果有多个输入生命周期参数，但其中一个是&self或者&mut self,那么self的生命周期会被赋给所有的输出生命周期参数，本规则适用于方法。



## 16.Traits

一个trait是为未知类型Self定义的一堆方法的集合。它们可以访问同一个trait的其他方法。

trait可以为任意类型实现其中定义的方法。下面的例子，我们定义了Animal,一组方法，然后Sheep数据类型实现了Animal trait.sheep实例允许使用Animal中定义的方法。

```rust
struct Sheep { naked: bool, name: &'static str }

trait Animal {
    // Associated function signature; `Self` refers to the implementor type.
    // 这里的Self指被实现的类型。
    fn new(name: &'static str) -> Self;

    // Method signatures; these will return a string.
    fn name(&self) -> &'static str;
    fn noise(&self) -> &'static str;

    // Traits can provide default method definitions.
    fn talk(&self) {
        println!("{} says {}", self.name(), self.noise());
    }
}

impl Sheep {
    fn is_naked(&self) -> bool {
        self.naked
    }

    fn shear(&mut self) {
        if self.is_naked() {
            // Implementor methods can use the implementor's trait methods.
            println!("{} is already naked...", self.name());
        } else {
            println!("{} gets a haircut!", self.name);

            self.naked = true;
        }
    }
}

// Implement the `Animal` trait for `Sheep`.
impl Animal for Sheep {
    // `Self` is the implementor type: `Sheep`.
    fn new(name: &'static str) -> Sheep {
        Sheep { name: name, naked: false }
    }

    fn name(&self) -> &'static str {
        self.name
    }

    fn noise(&self) -> &'static str {
        if self.is_naked() {
            "baaaaah?"
        } else {
            "baaaaah!"
        }
    }
    
    // Default trait methods can be overridden.
    fn talk(&self) {
        // For example, we can add some quiet contemplation.
        println!("{} pauses briefly... {}", self.name, self.noise());
    }
}

fn main() {
    // Type annotation is necessary in this case.
    let mut dolly: Sheep = Animal::new("Dolly");
    // TODO ^ Try removing the type annotations.

    dolly.talk();
    dolly.shear();
    dolly.talk();
}

```



### 16.1 Derive

编译器有能力通过#[derive]属性，为一些trait提供简单的实现。这些traits如有更加复杂的行为，仍然需要手动实现。

下面是可推导的traits列表：

- 比较traits：Eq,PartialEq,Ord,PartialOrd.
- Clone,通过拷贝 ，从&T创建T
- Copy，相比move语义，提供Copy语义
- Hash，从&T计算hash
- Default，创建一个空的数据实例
- Debug，使用{:?}格式化器来格式化一个值

```rust
// `Centimeters`, a tuple struct that can be compared
#[derive(PartialEq, PartialOrd)]
struct Centimeters(f64);

// `Inches`, a tuple struct that can be printed
#[derive(Debug)]
struct Inches(i32);

impl Inches {
    fn to_centimeters(&self) -> Centimeters {
        let &Inches(inches) = self;

        Centimeters(inches as f64 * 2.54)
    }
}

// `Seconds`, a tuple struct with no additional attributes
struct Seconds(i32);

fn main() {
    let _one_second = Seconds(1);

    // Error: `Seconds` can't be printed; it doesn't implement the `Debug` trait
    //println!("One second looks like: {:?}", _one_second);
    // TODO ^ Try uncommenting this line

  	// Seconds不能被打印，因为它没有实现Debug trait
    // Seconds不能被比较，因为它没有实现PartialEq
    // Error: `Seconds` can't be compared; it doesn't implement the `PartialEq` trait
    //let _this_is_true = (_one_second == _one_second);
    // TODO ^ Try uncommenting this line

    let foot = Inches(12);

    println!("One foot equals {:?}", foot);

    let meter = Centimeters(100.0);

    let cmp =
        if foot.to_centimeters() < meter {
            "smaller"
        } else {
            "bigger"
        };

    println!("One foot is {} than one meter.", cmp);
}

```





### 16.2 Returing Traits with dyn

Rust编译器需要知道每个函数返回的类型的占用空间。这意味着你的函数必须返回一个具体类型(concrete type)，不像其他语言，如果你有一个trait，比如Animal，你不能写一个函数返回Animal，因为不同的实现需要不同的内存空间。

不过，有一个简单的替代办法，我们的函数可以返回一个Box，这个Box包含了Animal，用来替代直接返回一个trait对象。Box是堆对象的引用。因为引用是静态、已知大小，编译器可以保证它指向一个已经堆上分配的Animal，我们可以从我们的函数返回trait

在堆上分配内存的时候，Rust会尝试尽量显示（Rust tries to be as explicit as possible whenever it allocates memory on the heap.）所以如果你的函数通过这种方法，返回一个指向在堆上分配的trait时，你需要使用dyn关键字写在返回类型前面，比如Box<dyn Animal>

```rust
struct Sheep {}
struct Cow {}

trait Animal {
    // Instance method signature
    fn noise(&self) -> &'static str;
}

// Implement the `Animal` trait for `Sheep`.
impl Animal for Sheep {
    fn noise(&self) -> &'static str {
        "baaaaah!"
    }
}

// Implement the `Animal` trait for `Cow`.
impl Animal for Cow {
    fn noise(&self) -> &'static str {
        "moooooo!"
    }
}

// Returns some struct that implements Animal, but we don't know which one at compile time.
// 返回已经实现Animal的结构体，但在编译的时候，我们不知道具体是哪个.
fn random_animal(random_number: f64) -> Box<dyn Animal> {
    if random_number < 0.5 {
        Box::new(Sheep {})
    } else {
        Box::new(Cow {})
    }
}

fn main() {
    let random_number = 0.234;
    let animal = random_animal(random_number);
    println!("You've randomly chosen an animal, and it says {}", animal.noise());
}



```



### 16.3 Operator Overloading(操作符重载)

再Rust中，很多操作符可以被重写。这是因为，一些操作符可以根据它们的输入参数完成不同的任务。有这种可能，是因为操作符是方法调用的语法糖。例如，在a+b中，+操作符调用add方法。（像是a.add(b）).这个add方法是Add trait的一部分，因此，+操作符能够被实现了Add trait的类型使用。

一些重载了操作符，可以在core::ops中找到。

```rust
use std::ops;

struct Foo;
struct Bar;

#[derive(Debug)]
struct FooBar;

#[derive(Debug)]
struct BarFoo;

// The `std::ops::Add` trait is used to specify the functionality of `+`.
// Here, we make `Add<Bar>` - the trait for addition with a RHS of type `Bar`.
// The following block implements the operation: Foo + Bar = FooBar
// 为FOO实现Add trait，Add trait声明了一个Bar类型，在add函数中被使用。
impl ops::Add<Bar> for Foo {
    type Output = FooBar;
    // 声明了返回类型为FooBar

    fn add(self, _rhs: Bar) -> FooBar {
        println!("> Foo.add(Bar) was called");
        FooBar
    }
}

// By reversing the types, we end up implementing non-commutative addition.
// Here, we make `Add<Foo>` - the trait for addition with a RHS of type `Foo`.
// This block implements the operation: Bar + Foo = BarFoo
impl ops::Add<Foo> for Bar {
    type Output = BarFoo;

    fn add(self, _rhs: Foo) -> BarFoo {
        println!("> Bar.add(Foo) was called");

        BarFoo
    }
}

fn main() {
    println!("Foo + Bar = {:?}", Foo + Bar);
    println!("Bar + Foo = {:?}", Bar + Foo);
}

```



### 16.4 Drop

Drop trait只有一个方法：drop，当一个对象离开作用域时会被自动调用。

Drop trait的主要目的是，释放对象实例拥有的资源。

下面的例子是当调用drop时，会在屏幕上打印一条信息。

```rust
struct Droppable {
    name: &'static str,
}

// This trivial implementation of `drop` adds a print to console.
impl Drop for Droppable {
    fn drop(&mut self) {
        println!("> Dropping {}", self.name);
    }
}

fn main() {
    let _a = Droppable { name: "a" };

    // block A
    {
        let _b = Droppable { name: "b" };

        // block B
        {
            let _c = Droppable { name: "c" };
            let _d = Droppable { name: "d" };

            println!("Exiting block B");
        }
        println!("Just exited block B");

        println!("Exiting block A");
    }
    println!("Just exited block A");

    // Variable can be manually dropped using the `drop` function
    drop(_a);
    // TODO ^ Try commenting this line

    println!("end of the main function");

    // `_a` *won't* be `drop`ed again here, because it already has been
    // (manually) `drop`ed
}

```

有点类似C++的析构函数。

### 16.5 Iterators(迭代器)

Iterator trait用于对集合，实现iterator，例如数组。

这个trait只需要为next 元素定义一个方法，它可以在impl 块中手动定义，或者自动定义（像arrays和ranges）

一般情况下，为了方便，for 语句通过.into_iter()方法将集合转到迭代器。

```rust
struct Fibonacci {
    curr: u32,
    next: u32,
}

// Implement `Iterator` for `Fibonacci`.
// The `Iterator` trait only requires a method to be defined for the `next` element.
impl Iterator for Fibonacci {
    // We can refer to this type using Self::Item
    type Item = u32;
    
    // Here, we define the sequence using `.curr` and `.next`.
    // The return type is `Option<T>`:
    //     * When the `Iterator` is finished, `None` is returned. 迭代完成，返回None
    //     * Otherwise, the next value is wrapped in `Some` and returned. 否则返回Some
    // We use Self::Item in the return type, so we can change
    // the type without having to update the function signatures.
    fn next(&mut self) -> Option<Self::Item> {
        let new_next = self.curr + self.next;

        self.curr = self.next;
        self.next = new_next;

        // Since there's no endpoint to a Fibonacci sequence, the `Iterator` 
        // will never return `None`, and `Some` is always returned.
        Some(self.curr)
    }
}

// Returns a Fibonacci sequence generator
fn fibonacci() -> Fibonacci {
    Fibonacci { curr: 0, next: 1 }
}

fn main() {
    // `0..3` is an `Iterator` that generates: 0, 1, and 2.
    let mut sequence = 0..3;

    println!("Four consecutive `next` calls on 0..3");
    println!("> {:?}", sequence.next());
    println!("> {:?}", sequence.next());
    println!("> {:?}", sequence.next());
    println!("> {:?}", sequence.next());

    // `for` works through an `Iterator` until it returns `None`.
    // Each `Some` value is unwrapped and bound to a variable (here, `i`).
    println!("Iterate through 0..3 using `for`");
    for i in 0..3 {
        println!("> {}", i);
    }

    // The `take(n)` method reduces an `Iterator` to its first `n` terms.
    println!("The first four terms of the Fibonacci sequence are: ");
    // 调用Iterator trait的take方法
    for i in fibonacci().take(4) {
        println!("> {}", i);
    }

    // The `skip(n)` method shortens an `Iterator` by dropping its first `n` terms.
    println!("The next four terms of the Fibonacci sequence are: ");
  // 调用Iterator trait的skip和take方法
    for i in fibonacci().skip(4).take(4) {
        println!("> {}", i);
    }

    let array = [1u32, 3, 3, 7];

    // The `iter` method produces an `Iterator` over an array/slice.
    println!("Iterate the following array {:?}", &array);
    for i in array.iter() {
        println!("> {}", i);
    }
}

```



### 16.6 impl Trait

impl Trait 可以被用在两个地方：

- 作为参数类型
- 作为返回类型

**作为参数类型**

如果你的函数是泛型的并且依赖于trait，但是你并不在意具体的类型，你可以简单的使用impl trait声明指定函数的参数。

例如，考虑如下的代码：

```rust
fn parse_csv_document<R: std::io::BufRead>(src: R) -> std::io::Result<Vec<Vec<String>>> {
    src.lines()
        .map(|line| {
            // For each line in the source
            line.map(|line| {
                // If the line was read successfully, process it, if not, return the error
                line.split(',') // Split the line separated by commas
                    .map(|entry| String::from(entry.trim())) // Remove leading and trailing whitespace
                    .collect() // Collect all strings in a row into a Vec<String>
            })
        })
        .collect() // Collect all lines into a Vec<Vec<String>>
}

```

Parse_csv_document 是一个泛型函数，允许传入任何实现了BufRead trait的类型参数，比如BufferRead<File>或[u8],R是什么类型并不重要，R只用来声明src的类型，所以函数也可以写成这样：

```rust
fn parse_csv_document(src: impl std::io::BufRead) -> std::io::Result<Vec<Vec<String>>> {
    src.lines()
        .map(|line| {
            // For each line in the source
            line.map(|line| {
                // If the line was read successfully, process it, if not, return the error
                line.split(',') // Split the line separated by commas
                    .map(|entry| String::from(entry.trim())) // Remove leading and trailing whitespace
                    .collect() // Collect all strings in a row into a Vec<String>
            })
        })
        .collect() // Collect all lines into a Vec<Vec<String>>
}

```

注意：使用impl Trait作为参数类型，你不能显示声明，你可以使用何种形式的函数，例如，parse_csv_document::<std::io::Empty>(std::io::empty)在第二个例子中无法正常使用。



**作为一个返回类型**

如果你的函数返回一个实现了MyTrait的类型，你可以写作-> impl MyTrait.

这会帮你，将函数的签名简化很多！

```rust
use std::iter;
use std::vec::IntoIter;

// This function combines two `Vec<i32>` and returns an iterator over it.
// Look how complicated its return type is!
// 这个函数组合了两个Vec<i32>,并返回一个关于它的迭代器。
// 看看这个返回类型有多么复杂！
fn combine_vecs_explicit_return_type(
    v: Vec<i32>,
    u: Vec<i32>,
) -> iter::Cycle<iter::Chain<IntoIter<i32>, IntoIter<i32>>> {
    v.into_iter().chain(u.into_iter()).cycle()
}

// This is the exact same function, but its return type uses `impl Trait`.
// Look how much simpler it is!
// 这个函数有相同的功能，但返回类型使用了 impl Trait,看看有多么简单。
fn combine_vecs(
    v: Vec<i32>,
    u: Vec<i32>,
) -> impl Iterator<Item=i32> {
    v.into_iter().chain(u.into_iter()).cycle()
}

fn main() {
    let v1 = vec![1, 2, 3];
    let v2 = vec![4, 5];
    let mut v3 = combine_vecs(v1, v2);
    assert_eq!(Some(1), v3.next());
    assert_eq!(Some(2), v3.next());
    assert_eq!(Some(3), v3.next());
    assert_eq!(Some(4), v3.next());
    assert_eq!(Some(5), v3.next());
    println!("all done");
}

```

更重要的是，一些Rust类型不能写到外面。例如，每一个闭包有它自己未命名的实体类型，在impl Trait语法之前，你必须在堆上分配内存，以便返回一个闭包。但是现在你可以像这样做：

```rust
// Returns a function that adds `y` to its input
// 返回一个闭包，y作为函数的输入
fn make_adder_function(y: i32) -> impl Fn(i32) -> i32 {
    let closure = move |x: i32| { x + y };
    closure
}

fn main() {
    let plus_one = make_adder_function(1);
    assert_eq!(plus_one(2), 3);
}

```

你也可以使用impl Trait返回一个迭代器，这将会使map和filter函数变得简单。因为闭包类型没有名字，你不能显式的返回一个迭代器的闭包。但是使用impl Trait ，你可以轻易的做到：

```rust
fn double_positives<'a>(numbers: &'a Vec<i32>) -> impl Iterator<Item = i32> + 'a {
    numbers
        .iter()
        .filter(|x| x > &&0)
        .map(|x| x * 2)
}

fn main() {
    let singles = vec![-3, -2, 2, 3];
    let doubles = double_positives(&singles);
    assert_eq!(doubles.collect::<Vec<i32>>(), vec![4, 6]);
}

```



### 16.7 Clone

当我们处理资源时，默认行为是，在赋值或函数调用时，会转移资源。然而，有时候，我们也需要拷贝资源。

Clone trait帮助我们正确的做这件事。绝大数情况下，我们可以定义Clone trait来使用.clone()方法。

```rust
// A unit struct without resources
// 没有资源的结构体
#[derive(Debug, Clone, Copy)]
struct Unit;

// A tuple struct with resources that implements the `Clone` trait
// 有资源的tuple结构体
#[derive(Clone, Debug)]
struct Pair(Box<i32>, Box<i32>);

fn main() {
    // Instantiate `Unit`
    let unit = Unit;
    // Copy `Unit`, there are no resources to move
    // 没有资源被move
    let copied_unit = unit;

    // Both `Unit`s can be used independently
    // 两个Unit可以独立使用
    println!("original: {:?}", unit);
    println!("copy: {:?}", copied_unit);

    // Instantiate `Pair`
    let pair = Pair(Box::new(1), Box::new(2));
    println!("original: {:?}", pair);

    // Move `pair` into `moved_pair`, moves resources
    // 资源发生了move
    let moved_pair = pair;
    println!("moved: {:?}", moved_pair);

    // Error! `pair` has lost its resources
    // pair被move后，丢失了它的资源
    //println!("original: {:?}", pair);
    // TODO ^ Try uncommenting this line

    // Clone `moved_pair` into `cloned_pair` (resources are included)
    // 将move_pair克隆到cloned_pair，也包括资源
    let cloned_pair = moved_pair.clone();
    // Drop the original pair using std::mem::drop
    // moved_pair被销毁
    drop(moved_pair);

    // Error! `moved_pair` has been dropped
    //println!("copy: {:?}", moved_pair);
    // TODO ^ Try uncommenting this line
    // 这里不能再使用moved_pair

    // The result from .clone() can still be used!
    // 但 cloned_pair仍然能够使用，因为它们都有自己的资源。
    println!("clone: {:?}", cloned_pair);
}

```

### 16.8 Supertraits

Rust没有继承机制，但是你可以定义一个超集trait作为其他trait。例如：

```rust
trait Person {
    fn name(&self) -> String;
}

// Person is a supertrait of Student.
// Person是Student的超集，实现Student同样需要实现Person trait
// Implementing Student requires you to also impl Person.
trait Student: Person {
    fn university(&self) -> String;
}

trait Programmer {
    fn fav_language(&self) -> String;
}

// CompSciStudent (computer science student) is a subtrait of both Programmer 
// and Student. Implementing CompSciStudent requires you to impl both supertraits.
// CompSciStudent是Programer和Student的子集。实现CompSciStuden需要同时实现超集trait
trait CompSciStudent: Programmer + Student {
    fn git_username(&self) -> String;
}

fn comp_sci_student_greeting(student: &dyn CompSciStudent) -> String {
    format!(
        "My name is {} and I attend {}. My favorite language is {}. My Git username is {}",
        student.name(),
        student.university(),
        student.fav_language(),
        student.git_username()
    )
}

fn main() {}

```

### 16.9 Disambiguating overlapping traits

一个类型可能实现了许多不同的traits,如果两个trait都需要有相同的名字会怎么样呢？例如，许多trait可能都有一个名字为get()的方法，它们甚至可能有不同返回类型。

好消息是，因为每个trait的实现有它们自己的impl 块，很清楚是哪个trait的get方法。

如果要同时调用这些方法怎么办呢？为了消除歧义，我们必须使用全名语法（Fully Qualified Syntax）

```rust
trait UsernameWidget {
    // Get the selected username out of this widget
    fn get(&self) -> String;
}

trait AgeWidget {
    // Get the selected age out of this widget
    fn get(&self) -> u8;
}

// A form with both a UsernameWidget and an AgeWidget
struct Form {
    username: String,
    age: u8,
}

impl UsernameWidget for Form {
    fn get(&self) -> String {
        self.username.clone()
    }
}

impl AgeWidget for Form {
    fn get(&self) -> u8 {
        self.age
    }
}

fn main() {
    let form = Form {
        username: "rustacean".to_owned(),
        age: 28,
    };

    // If you uncomment this line, you'll get an error saying
    // "multiple `get` found". Because, after all, there are multiple methods
    // named `get`.
    // println!("{}", form.get());
    // form.get()调用会有歧义，rust不知道你到底是想调用哪个trait中的get方法。

  	// 将Form转型为UsernameWidget
    let username = <Form as UsernameWidget>::get(&form);
    assert_eq!("rustacean".to_owned(), username);
    // 将Form转型为AgeWidget
    let age = <Form as AgeWidget>::get(&form);
    assert_eq!(28, age);
}

```







