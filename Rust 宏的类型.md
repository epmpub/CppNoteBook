Rust 的宏（Macro）是一种强大的 **元编程工具**，允许在编译时生成或变换代码。与函数不同，宏可以操作代码结构本身，适用于模式化代码生成、语法扩展和减少重复。以下是 Rust 宏的全面解析：

---

### **一、宏的类型**
Rust 支持两类宏，分别适用于不同场景：

| **宏类型**   | **定义方式**         | **典型用途**                 | **示例**           |
| ------------ | -------------------- | ---------------------------- | ------------------ |
| **声明式宏** | `macro_rules!` 语法  | 模式匹配生成代码             | `vec!`, `println!` |
| **过程宏**   | 通过 `proc-macro` 库 | 生成复杂代码、自定义语法扩展 | `#[derive(Debug)]` |

---

### **二、声明式宏（Declarative Macros）**
通过 `macro_rules!` 定义，基于模式匹配生成代码。

#### **1. 基本语法**
```rust
macro_rules! my_macro {
    // 模式匹配 => 代码模板
    ($arg:expr) => {
        println!("Value: {}", $arg);
    };
    ($a:expr, $b:expr) => {
        println!("Sum: {}", $a + $b);
    };
}

fn main() {
    my_macro!(42);       // 输出: Value: 42
    my_macro!(3, 5);     // 输出: Sum: 8
}
```

#### **2. 模式匹配符**
| **标识符**     | **含义**             | **示例**      |
| -------------- | -------------------- | ------------- |
| `$var:expr`    | 匹配表达式           | `$x:expr`     |
| `$var:ident`   | 匹配标识符（变量名） | `$func:ident` |
| `$var:ty`      | 匹配类型             | `$T:ty`       |
| `$var:literal` | 匹配字面量           | `"hello"`     |

#### **3. 重复模式**
使用 `$(...)*` 或 `$(...)+` 匹配重复部分：
```rust
macro_rules! vector {
    ($($elem:expr),*) => {
        {
            let mut v = Vec::new();
            $(v.push($elem);)*
            v
        }
    };
}

fn main() {
    let v = vector![1, 2, 3]; // 等价于 vec![1, 2, 3]
}
```

---

### **三、过程宏（Procedural Macros）**
通过 Rust 代码动态生成代码，分为三类：

#### **1. 派生宏（Derive Macros）**
自动为结构体或枚举实现 trait。
```rust
// 自定义派生宏实现
use proc_macro::TokenStream;

#[proc_macro_derive(Hello)]
pub fn hello_derive(input: TokenStream) -> TokenStream {
    // 解析输入并生成代码
    // ...
}

// 使用
#[derive(Hello)]
struct MyStruct;
```

#### **2. 属性宏（Attribute Macros）**
为代码块添加自定义属性逻辑。
```rust
#[route(GET, "/")]
fn index() { /* ... */ }

// 宏定义
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
    // 解析 attr 和 item 并生成代码
    // ...
}
```

#### **3. 函数式宏（Function-like Macros）**
类似声明式宏，但更灵活。
```rust
my_macro!(some code here);

// 宏定义
#[proc_macro]
pub fn my_macro(input: TokenStream) -> TokenStream {
    // 解析 input 并生成代码
    // ...
}
```

---

### **四、宏的典型应用场景**
#### **1. 消除重复代码**
```rust
// 生成多个相似函数
macro_rules! create_functions {
    ($($name:ident),*) => {
        $(fn $name() { println!("Function: {}", stringify!($name)); })*
    };
}

create_functions!(foo, bar);
fn main() {
    foo(); // 输出: Function: foo
    bar(); // 输出: Function: bar
}
```

#### **2. 领域特定语言（DSL）**
```rust
// 自定义 HTML DSL
macro_rules! html {
    ($tag:ident { $($body:tt)* }) => {
        format!("<{}>{}</{}>", stringify!($tag), html!($($body)*), stringify!($tag))
    };
    ($text:expr) => { $text };
}

fn main() {
    let page = html! {
        div {
            h1 { "Hello World" }
            p { "This is a paragraph." }
        }
    };
    println!("{}", page);
}
```

#### **3. 编译时计算**
```rust
// 计算数组长度（编译时）
macro_rules! const_len {
    ($($elem:expr),*) => {
        <[()]>::len(&[$(const_len!(@unit $elem)),*])
    };
    (@unit $e:expr) => { () };
}

const ARR_LEN: usize = const_len!(1, 2, 3); // 编译时值为 3
```

---

### **五、宏的注意事项**
1. **卫生性（Hygiene）**  
   Rust 宏自动避免变量名冲突，无需手动处理（不同于 C 宏）。
2. **调试困难**  
   使用 `cargo expand` 查看宏展开后的代码：
   ```bash
   cargo install cargo-expand
   cargo expand
   ```
3. **性能影响**  
   宏在编译时展开，不影响运行时性能。

---

### **六、总结**
| **场景**       | **推荐宏类型**            |
| -------------- | ------------------------- |
| 简单模式替换   | 声明式宏 (`macro_rules!`) |
| 复杂代码生成   | 过程宏                    |
| 自定义语法扩展 | 属性宏/函数式宏           |

合理使用宏可以显著提升代码的灵活性和简洁性，但需注意 **可读性** 和 **维护成本**。对于常见任务（如 `println!`），优先使用标准库宏；对于领域特定需求，可自定义宏实现高效开发。