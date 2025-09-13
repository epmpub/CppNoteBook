## cloned() 和 clone()有什么差异

在 Rust 中，cloned() 和 clone() 都是用于复制数据的，但它们的用途、上下文和行为有显著差异。以下是对 cloned() 和 clone() 的详细对比，特别是在你的代码背景（Rayon 和迭代器）中，结合简洁的解释。1. 定义和来源

- cloned():
  - 属于 Rust 标准库的 Iterator trait，定义在 std::iter 中。
  - 是一个迭代器方法，专门用于将迭代器中的引用类型（&T 或 &&T）转换为值类型（T），通过调用 T 的 clone() 方法。
  - 要求 T 实现 Clone trait。
  - 常用于迭代器链式调用，如 numbers.par_iter().cloned()。
- clone():
  - 属于 Clone trait，定义在 std::clone 中。
  - 是一个通用的实例方法，直接在某个值上调用（如 value.clone()），用于生成该值的深拷贝。
  - 适用于任何实现了 Clone trait 的类型，不限于迭代器上下文。
- 使用场景

- cloned():

  - 用于迭代器链中，批量处理引用类型的元素，生成值类型的迭代器。

  - 典型场景：将 Iterator<Item = &T> 转换为 Iterator<Item = T>。

  - 示例：

    rust

    

    ```rust
    let numbers = vec![1, 2, 3];
    let cloned: Vec<i32> = numbers.iter().cloned().collect(); // 转换为 Vec<i32>
    ```

    这里 iter() 产生 &i32 的迭代器，cloned() 调用每个 &i32 的 clone() 方法，生成 i32。

- clone():

  - 用于单个值的复制，适用于任何需要复制的具体对象。

  - 示例：

    rust

    

    ```rust
    let s = String::from("hello");
    let s_copy = s.clone(); // 复制整个 String
    ```

    这里 clone() 直接作用于 s，生成一个新的 String。

- 与 copied() 的关系在你的代码背景中（numbers.par_iter().filter(|&&x| x % 2 == 0).copied().collect()），你提到 copied() 用于将 &i32 转为 i32。cloned() 可以作为替代，但有以下差异：

- copied():

  - 要求类型实现 Copy trait（i32 满足），执行高效的位复制（bitwise copy）。

  - 更轻量，适合基本类型（如 i32、f64），因为 Copy 不涉及复杂内存分配。

  - 示例：

    rust

    

    ```rust
    let evens: Vec<i32> = numbers.par_iter().filter(|&&x| x % 2 == 0).copied().collect();
    ```

- cloned():

  - 要求类型实现 Clone trait，可能涉及深拷贝（如为 String 分配新内存）。

  - 对于 i32 这样同时实现 Copy 和 Clone 的类型，cloned() 效果等同于 copied()，但语义上不如 copied() 明确（因为 cloned() 暗示可能有更高开销）。

  - 示例：

    rust

    

    ```rust
    let evens: Vec<i32> = numbers.par_iter().filter(|&&x| x % 2 == 0).cloned().collect();
    ```

- 性能差异

- copied():

  - 更高效，专为 Copy 类型设计，避免不必要的开销。
  - 对于 i32 这样的简单类型，copied() 只是简单的内存复制。

- cloned():

  - 更通用，可能涉及深拷贝（如 String 或 Vec 的复制），开销更大。
  - 对于 Copy 类型（如 i32），cloned() 等效于 copied()，但编译器优化后性能无明显差异。

- clone():

  - 针对单个值，可能涉及深拷贝，性能取决于类型的 Clone 实现。

  - 在迭代器中，clone() 需要显式调用（如通过 map），不如 cloned() 简洁：

    rust

    

    ```rust
    let evens: Vec<i32> = numbers.par_iter().filter(|&&x| x % 2 == 0).map(|&x| x.clone()).collect();
    ```

- 适用类型

- cloned():
  - 适用于任何实现了 Clone 的类型（如 i32、String、Vec<T>）。
  - 但在迭代器中，通常处理 &T 到 T 的转换。
- clone():
  - 适用于任何实现了 Clone 的类型，上下文更广（如结构体、枚举等）。
- 注意：copied() 比 cloned() 更严格，因为它要求 Copy trait，而 cloned() 只要求 Clone trait。
- 代码示例对比

rust



```rust
let numbers = vec![1, 2, 3, 4, 5];

// 使用 copied()（推荐，i32 实现 Copy）
let evens: Vec<i32> = numbers.par_iter().filter(|&&x| x % 2 == 0).copied().collect();
// 输出：[2, 4]

// 使用 cloned()（等效但语义稍弱）
let evens: Vec<i32> = numbers.par_iter().filter(|&&x| x % 2 == 0).cloned().collect();
// 输出：[2, 4]

// 使用 clone()（需要显式 map，稍繁琐）
let evens: Vec<i32> = numbers.par_iter().filter(|&&x| x % 2 == 0).map(|&x| x.clone()).collect();
// 输出：[2, 4]
```

对于非 Copy 类型（如 String）：

rust



```rust
let strings = vec![String::from("a"), String::from("b")];
// cloned() 适用，copied() 会报错（String 不实现 Copy）
let cloned: Vec<String> = strings.iter().cloned().collect();
// 直接 clone() 示例
let s = strings[0].clone(); // 复制单个 String
```

7. 总结

- cloned():
  - 迭代器方法，批量将 &T 转为 T，调用 T::clone()。
  - 适合迭代器链，通用但可能有深拷贝开销。
- clone():
  - 实例方法，复制单个值，适用于任何 Clone 类型。
  - 在迭代器中需配合 map，不如 cloned() 简洁。
- 与 copied() 的选择：
  - 对于 i32 等 Copy 类型，优先用 copied()，语义更明确，性能等效。
  - 对于 String 等非 Copy 类型，只能用 cloned() 或 map(|x| x.clone())。

在你的代码中，copied() 是最佳选择，因为 i32 实现 Copy，效率高且代码清晰。如果处理复杂类型（如 String），则需用 cloned()。如果有进一步问题或需要更多示例，随时告诉我！