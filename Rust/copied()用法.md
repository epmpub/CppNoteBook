## copied()用法

```rust
let evens: Vec<i32> = numbers.par_iter().filter(|&&x| x % 2 == 0).copied().collect();
```

copied() 方法的作用是将迭代器中的引用类型（&T）转换为值类型（T），通过复制（copy）引用的值。它是 Rust 标准库中 Iterator trait 提供的一个方法，适用于实现了 Copy trait 的类型（如 i32、f64 等基本类型）。具体分析

1. 代码上下文：

   - numbers 是一个 Vec<i32>，调用 par_iter() 返回一个并行迭代器，迭代的元素类型是 &i32（对 Vec 中元素的引用）。
   - filter(|&&x| x % 2 == 0) 过滤出偶数。由于 par_iter() 返回的迭代器元素是 &i32，在 filter 闭包中，x 是 &&i32（因为闭包捕获的是引用类型）。通过两次解引用（&&x），x 变成 i32，以便进行 % 运算。

2. copied() 的作用：

   - 经过 filter，迭代器的元素仍然是 &i32（引用类型）。
   - copied() 将每个 &i32 复制为 i32 值。i32 实现了 Copy trait，因此可以安全地复制，而不会涉及复杂的克隆（clone）或所有权转移。
   - 这样，迭代器的元素从 &i32 转换为 i32，以便 collect 收集到一个 Vec<i32> 中。

3. 为什么需要 copied()：

   - 如果没有 copied()，迭代器的元素是 &i32，collect 会尝试生成 Vec<&i32>（引用类型的向量）。但你的目标是 Vec<i32>（值类型的向量）。
   - copied() 确保迭代器输出的是 i32，这样 collect 就能生成符合预期的 Vec<i32>。

4. 替代方法：

   - 如果不使用 copied()，你可以用 cloned()，它对实现了 Clone trait 的类型进行深拷贝。但对于 i32 这样的简单类型，copied() 更高效，因为它利用了 Copy trait 的零成本复制。

   - 或者，可以在 map 中手动解引用并复制：

     rust

     

     ```rust
     let evens: Vec<i32> = numbers.par_iter().filter(|&&x| x % 2 == 0).map(|&x| x).collect();
     ```

     但 copied() 更简洁且语义明确。

代码效果

rust



```rust
let numbers = vec![1, 2, 3, 4, 5];
let evens: Vec<i32> = numbers.par_iter().filter(|&&x| x % 2 == 0).copied().collect();
// 输出：[2, 4]
```

- par_iter()：生成 &i32 的并行迭代器。
- filter(|&&x| x % 2 == 0)：保留偶数的引用（&i32）。
- copied()：将 &i32 转为 i32。
- collect()：收集为 Vec<i32>。

总结copied() 是一个便捷方法，用于将迭代器中的引用类型（&T）转换为值类型（T），前提是 T 实现了 Copy trait。它在你的代码中确保了 collect 能生成 Vec<i32> 而不是 Vec<&i32>，语法简洁且性能高效。如果还有其他疑问或需要更深入的解释，随时告诉我！