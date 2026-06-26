**C++26 `constexpr` Exception Types**（例如 `std::bad_alloc`、`std::bad_cast` 等）是通过提案 **P3068R6**（允许常量求值中抛出异常）和 **P3378R2**（`constexpr` 异常类型）引入的特性。

### 核心变化

1. **语言层面**（P3068）：允许在 `constexpr` / `consteval` 上下文中使用 `throw` 和 `catch`。
   - 如果异常**未被捕获**，常量求值失败（产生编译错误），但错误信息更友好（可显示 `what()` 消息）。
   - 如果异常被**正确捕获**，则常量求值可以继续（类似于 `constexpr` 动态内存分配）。

2. **库层面**：大量标准异常类型现在**构造函数、析构函数、拷贝操作和 `what()`** 都是 `constexpr`。

### 支持的异常类型（C++26）

- **`std::exception`**（基类）：所有成员函数均为 `constexpr`。
- **`std::bad_alloc`**、`std::bad_cast`**（用户特别提到的）。
- **`std::bad_array_new_length`**、`std::bad_typeid`。
- `<stdexcept>` 中的：
  - `std::logic_error` 及其派生：`domain_error`、`invalid_argument`、`length_error`、`out_of_range`。
  - `std::runtime_error` 及其派生：`range_error`、`overflow_error`、`underflow_error`。
- 其他：
  - `std::bad_optional_access`、`std::bad_variant_access`、`std::bad_expected_access`。
  - `std::format_error` 等。

**Feature Test Macro**：
```cpp
#if __cpp_lib_constexpr_exceptions >= 202411L
// 支持 constexpr 异常类型
#endif
```

### 示例

```cpp
#include <stdexcept>
#include <new>
#include <optional>

constexpr unsigned safe_divide(unsigned n, unsigned d) {
    if (d == 0) {
        throw std::invalid_argument{"division by zero"};  // C++26 前错误
    }
    return n / d;
}

constexpr std::optional<unsigned> checked_divide(unsigned n, unsigned d) {
    try {
        return safe_divide(n, d);
    } catch (...) {          // 可以捕获
        return std::nullopt;
    }
}

constexpr auto ok = checked_divide(10, 2);     // OK → 5
// constexpr auto bad = checked_divide(10, 0); // 编译错误（未捕获异常）
```

**`bad_alloc` 示例**（在 `constexpr new` 失败时）：
```cpp
constexpr int* allocate() {
    return new int[1000000000];  // 可能在常量求值中抛出 bad_alloc
}
```

### 重要语义

- `constexpr` 异常**不能逃出**常量求值上下文（和 `constexpr` 分配内存类似）。
- `what()` 返回的字符串在常量求值中使用普通字面编码。
- 这极大提升了编译时错误报告能力，尤其配合静态反射、编译时容器等特性。

### 实际影响

- 编译时编程更强大：可以写出带错误处理的 `constexpr` 函数，而不需要返回错误码或 `std::optional`。
- 标准库实现（如 `dynamic_cast`、`typeid`、`new` 等）在常量求值中现在能抛出合适的异常。
- GCC、Clang 等编译器正在快速跟进支持。

更多细节参考：
- [cppreference: std::exception](https://en.cppreference.com/w/cpp/error/exception)（C++26 标记）
- 提案 P3068R6 和 P3378R2

这是 C++26 “constexpr 大扩张”中非常实用的一部分，让编译时代码更接近运行时代码的表达力。需要具体某个异常类型的示例或与之前 constexpr 特性的组合用法吗？