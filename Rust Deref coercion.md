Deref coercion

在 Rust 里，解引用强制转换（Deref coercion）是一项很有用的特性，它能自动把引用类型转换为其他引用类型。这一特性让代码更具灵活性和可读性。以下是对 Rust 解引用强制转换的详细介绍：

### 工作原理

当函数或方法期望的是某种引用类型，而传入的却是另一种引用类型时，解引用强制转换就会发挥作用。Rust 会自动调用`Deref`或`DerefMut`特性的`deref`方法，从而将传入的引用类型转换为所需的类型。

### 示例

```rust
use std::ops::Deref;

// 定义一个自定义智能指针
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

// 为MyBox实现Deref特性
impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

// 一个接受字符串切片的函数
fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    // 这里发生了解引用强制转换
    hello(&m);
}
```

### 代码解释

1. **自定义智能指针`MyBox`**：`MyBox`是一个简单的自定义智能指针，它封装了一个类型为`T`的值。
2. **实现`Deref`特性**：为`MyBox`实现`Deref`特性，需要定义`Target`类型并实现`deref`方法。`deref`方法返回一个指向内部值的引用。
3. **`hello`函数**：`hello`函数接受一个字符串切片`&str`作为参数。
4. **解引用强制转换**：在`main`函数中，`hello(&m)`调用时，`&m`是`&MyBox<String>`类型。由于`MyBox`实现了`Deref`特性，Rust 会自动调用`deref`方法，将`&MyBox<String>`转换为`&String`，再进一步转换为`&str`，这样就可以匹配`hello`函数的参数类型。

### 注意事项

- 解引用强制转换仅适用于引用类型。
- 若涉及可变性，Rust 也会进行相应的转换，如从`&mut T`转换为`&T`。
- 解引用强制转换会在编译时完成，不会产生运行时开销。

