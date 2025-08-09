# 使用 C++20 概念来限制对 noexcept 支持的代码

Constraining code on noexcept support using C++20 concepts.

实现通用 C++ 代码可能比较棘手，因为任何操作都可能引发错误。

值得注意的是，当需要强大的异常保证时，这会大大复杂化代码并导致运行时开销（甚至改变大 O 复杂性）。

幸运的是，C++20 概念可用于在编译时强制执行*noexcept保证。*



```C++
#include <utility>

struct UnsafeType {
    UnsafeType() = default;
    UnsafeType(UnsafeType&&) {}
    UnsafeType& operator=(UnsafeType&&) { return *this; }
};

template <typename T>
void unsafe_swap(T& left, T& right) {
    auto tmp = std::move(left);
    left = std::move(right); // What happens if this move throws?
    // left was moved from, and moving it back might throw again
    right = std::move(tmp);
}

struct SafeType {
    SafeType() = default;
    SafeType(SafeType&&) noexcept {}
    SafeType& operator=(SafeType&&) noexcept { return *this; }
};

template <typename T>
requires requires (T& a, T& b) {
 	// move assignment is valid and doesn't throw
	{ a = std::move(b) } noexcept;
}
void safe_swap(T& left, T& right) {
    auto tmp = std::move(left);
    left = std::move(right);
    right = std::move(tmp);
}

int main() {
    SafeType a, b;
    safe_swap(a, b); // OK

    UnsafeType x, y;
    // Wouldn't compile:
    // safe_swap(x, y); // UnsafeType doesn't satisfy the noexcept requirement
}
```



这段代码展示了 C++ 中移动语义（move semantics）和异常安全性（exception safety）在交换操作中的应用，同时使用了 C++20 的 requires 约束来限制模板函数的使用。以下是对代码的逐步解释：

------

1. UnsafeType 结构体

cpp

```cpp
struct UnsafeType {
    UnsafeType() = default;
    UnsafeType(UnsafeType&&) {}
    UnsafeType& operator=(UnsafeType&&) { return *this; }
};
```

- **用途**：定义一个类型，其移动构造函数和移动赋值操作符未标记为 noexcept。
- **细节**：
  - UnsafeType(UnsafeType&&) 是移动构造函数，默认情况下不保证不抛异常（即不是 noexcept）。
  - UnsafeType& operator=(UnsafeType&&) 是移动赋值操作符，同样未标记 noexcept。
- **含义**：编译器无法假设这些操作是异常安全的，可能在移动过程中抛出异常。

------

2. unsafe_swap 函数

cpp

```cpp
template <typename T>
void unsafe_swap(T& left, T& right) {
    auto tmp = std::move(left);
    left = std::move(right); // What happens if this move throws?
    // left was moved from, and moving it back might throw again
    right = std::move(tmp);
}
```

- **用途**：实现一个通用的交换函数，使用移动语义将 left 和 right 的内容互换。
- **执行步骤**：
  1. 将 left 移动到临时变量 tmp（left 变为“已移动”状态）。
  2. 将 right 移动赋值给 left。
  3. 将 tmp 移动赋值给 right。
- **问题**：
  - 如果第 2 步（left = std::move(right)）抛出异常：
    - left 已经处于“已移动”状态（可能不可用）。
    - right 尚未被修改。
    - tmp 持有原始的 left 值，但无法恢复状态。
  - 如果第 3 步（right = std::move(tmp)）也抛异常，则情况更糟，数据可能完全丢失。
- **异常安全性**：这个函数不提供强异常保证（strong exception guarantee），即如果抛出异常，状态可能不一致。

------

3. SafeType 结构体

cpp

```cpp
struct SafeType {
    SafeType() = default;
    SafeType(SafeType&&) noexcept {}
    SafeType& operator=(SafeType&&) noexcept { return *this; }
};
```

- **用途**：定义一个类型，其移动构造函数和移动赋值操作符都标记为 noexcept。
- **细节**：
  - SafeType(SafeType&&) noexcept：移动构造函数保证不抛异常。
  - SafeType& operator=(SafeType&&) noexcept：移动赋值操作符保证不抛异常。
- **含义**：编译器可以安全地假设这些操作不会中断，适合需要异常安全性的场景。

------

4. safe_swap 函数

cpp

```cpp
template <typename T>
requires requires (T& a, T& b) {
    { a = std::move(b) } noexcept;
}
void safe_swap(T& left, T& right) {
    auto tmp = std::move(left);
    left = std::move(right);
    right = std::move(tmp);
}
```

- **用途**：实现一个异常安全的交换函数，仅接受满足特定条件的类型。
- **约束**：
  - 使用 C++20 的 requires 子句，要求类型 T 的移动赋值操作 (a = std::move(b)) 是合法的且标记为 noexcept。
  - 这确保交换操作在任何步骤抛出异常的可能性为零。
- **执行步骤**：与 unsafe_swap 相同，但因为移动操作是 noexcept，整个函数提供强异常保证：
  - 如果没有异常，交换成功完成。
  - 如果有异常（理论上不会发生），状态保持一致。
- **优势**：异常安全性得到保证，适用于需要可靠性的场景。

------

5. main 函数

cpp

```cpp
int main() {
    SafeType a, b;
    safe_swap(a, b); // OK

    UnsafeType x, y;
    // Wouldn't compile:
    // safe_swap(x, y); // UnsafeType doesn't satisfy the noexcept requirement
}
```

- **用途**：测试 safe_swap 和 unsafe_swap 的行为。
- **细节**：
  - SafeType a, b：创建两个 SafeType 对象。
  - safe_swap(a, b)：编译通过，因为 SafeType 的移动赋值操作是 noexcept 的，满足 safe_swap 的约束。
  - UnsafeType x, y：创建两个 UnsafeType 对象。
  - safe_swap(x, y)：被注释掉，因为如果取消注释，编译会失败。原因是 UnsafeType 的移动赋值操作不是 noexcept，不满足 requires 约束。

------

总结

1. **UnsafeType 和 unsafe_swap**：
   - 展示了移动语义中异常安全性的潜在问题。
   - 如果移动操作可能抛异常，交换操作可能导致数据丢失或不一致状态。
2. **SafeType 和 safe_swap**：
   - 通过将移动操作标记为 noexcept 并使用 requires 约束，确保交换操作的异常安全性。
   - 只有满足 noexcept 条件的类型才能使用 safe_swap。
3. **关键概念**：
   - **移动语义**：使用 std::move 优化性能，但需要考虑异常情况。
   - **异常安全性**：强异常保证要求操作要么成功，要么不改变状态。
   - **C++20 约束**：requires 提供了编译期类型检查，增强了代码的安全性。

这段代码通过对比，清晰地说明了异常安全设计的重要性，以及如何利用现代 C++ 特性（如 noexcept 和 requires）来避免潜在问题。