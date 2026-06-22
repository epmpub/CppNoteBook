**`constexpr std::atomic<T>` 和 `std::atomic_ref<T>`** 是 C++26 通过提案 **P3309R3** 引入的重要特性。

### 背景与动机
在 C++26 之前，`std::atomic` 和 `std::atomic_ref` **不能**用于 `constexpr` / `consteval` 上下文。这导致需要在编译时和运行时维护两份几乎相同的代码（一份普通版，一份纯 `constexpr` 版本），非常不便。

**P3309R3** 的目标是让大部分原子操作在**常量求值**（constant evaluation）中可用，从而实现代码复用，尤其适合：
- 编译时初始化复杂数据结构（带原子计数的表、共享指针等）。
- `consteval` 函数中模拟单线程原子操作。
- 复用运行时多线程代码到编译时。

### 主要变更（C++26）

- **大多数成员函数和自由函数** 标记为 `constexpr`（**排除** `volatile` 重载）。
- `is_lock_free()` / `is_always_lock_free` **不**标记为 `constexpr`（因为运行时环境可能不同）。
- `wait()` / `notify_one()` / `notify_all()`：
  - 在常量求值中，`notify` 是 no-op。
  - `wait` 如果等待不同值，会被视为无限循环（deadlock），导致常量求值失败（符合常量表达式限制）。
- 内存顺序（`memory_order`）参数在常量求值中被忽略（单线程环境）。
- 同步原语（如 `atomic_thread_fence`、`kill_dependency`）实现为 no-op。

**Feature Test Macro**：
```cpp
#if __cpp_lib_constexpr_atomic >= 202411L
// 支持 constexpr atomic / atomic_ref
#endif
```

### 支持范围
- `std::atomic<T>`（`T` 为 `TriviallyCopyable` 类型）。
- `std::atomic_ref<T>`。
- 指针、整数、浮点等特化均受支持。
- `std::atomic<std::shared_ptr<T>>` / `weak_ptr` 的 constexpr 支持取决于另一个提案（可能稍后落地）。

### 示例（来自提案）

```cpp
constexpr bool process_first_unprocessed(std::atomic<size_t>& counter, 
                                         std::span<cell> subject) {
    const size_t current = counter.fetch_add(1);  // C++26 前：constexpr 错误
    if (current >= subject.size()) {
        return false;
    }
    process(subject[current]);
    return true;
}

constexpr void process_all(std::span<cell> subject, unsigned thread_count = 1) {
    std::atomic<size_t> counter{0};   // OK in constexpr
    // ... 单线程路径在 constexpr 中正常工作
}
```

在常量求值中，多个 `fetch_add` 等操作会顺序执行，就像单线程环境一样。

### 注意事项
1. **常量求值语义**：常量求值是**单线程**的，没有真正的并发。原子操作退化为普通读写 + 必要的检查。
2. **volatile 重载**：仍非 `constexpr`（与整体设计一致）。
3. **实现方式**：标准库通常用 `if consteval { /* 普通操作 */ } else { /* 运行时原子指令 */ }` 或允许原子内置在常量求值器中工作。
4. **限制**：`constexpr std::atomic` 不能真正解决多线程问题，但极大提升了**代码复用性**。

### 实际影响
- 极大方便了**编译时计算**中使用原子风格的计数、状态机等。
- 为未来更多并发原语的 `constexpr` 化铺路。
- 编译器/标准库（如 libc++、libstdc++）已在跟进支持。

更多细节可参考：
- 提案 [P3309R3](https://wg21.link/P3309R3)
- cppreference：[`std::atomic`](https://en.cppreference.com/w/cpp/atomic/atomic) 和 [`std::atomic_ref`](https://en.cppreference.com/w/cpp/atomic/atomic_ref)（已标记 C++26 constexpr 部分）

这个特性是 C++26 “constexpr 化” 大潮中的重要一环，让原子操作在编译时也更加实用。