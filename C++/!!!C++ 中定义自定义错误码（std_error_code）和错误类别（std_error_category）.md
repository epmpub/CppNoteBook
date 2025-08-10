# C++ 中定义自定义错误码（std::error_code）和错误类别（std::error_category）

```c++
#include <system_error>
#include <string>
#include <iostream>
#include <utility>
#include <expected>
#include <cassert>

// Custom error enum
enum class TransactionError {
    OK = 0,
    TemporaryError = 1,
    PermanentError = 2,
};

// Custom category that provides text description
struct TransactionErrorCategory : std::error_category {
    const char* name() const noexcept override {
        return "transaction";
    }
    std::string message( int condition ) const override {
        using namespace std::string_literals;
        switch(condition) {
            case 0: return "ok"s;
            case 1: return "temporary error, please retry"s;
            case 2: return "permanent error"s;
        }
        std::abort(); // unreacheable
    }
};

// Register the enum as an error code enum
template<> struct std::is_error_code_enum<TransactionError> 
    : public std::true_type{};

// Mapping from error code enum to category
std::error_code make_error_code(TransactionError e) {
    static auto category = TransactionErrorCategory{};
    return std::error_code(std::to_underlying(e), category);
}

// And now we can use it:
std::error_code my_function() noexcept {
    return TransactionError::PermanentError;
}
// And obviously, this also works with std::expected
std::expected<void,std::error_code> my_other_function() noexcept {
    return std::unexpected{TransactionError::TemporaryError};
}

int main() {
    if (auto err = my_function(); err) {
        std::cout << err.category().name() << " : " << err.message() << "\n";
        // prints: "transaction : permanent error"
    }
    if (auto res = my_other_function(); !res) {
        std::cout << res.error().category().name() << " : " << res.error().message() << "\n";
        // prints: "transaction : temporary error, please retry"
    }
}
```



这段代码展示了如何在 C++ 中定义自定义错误码（std::error_code）和错误类别（std::error_category），并将其与现代 C++23 的 std::expected 类型结合使用。代码通过自定义错误枚举 TransactionError 和对应的错误类别 TransactionErrorCategory，实现了标准化的错误处理机制。以下是逐步解释。

------

代码概览

- 定义了一个自定义错误枚举 TransactionError，表示事务相关的错误。
- 创建了一个自定义错误类别 TransactionErrorCategory，提供错误码的名称和描述。
- 将 TransactionError 注册为错误码枚举，并实现了 make_error_code 函数。
- 在函数中使用自定义错误，并结合 std::error_code 和 std::expected 展示其用法。

------

关键组件

1. **头文件**

cpp

```cpp
#include <system_error>
#include <string>
#include <iostream>
#include <utility>
#include <expected>
#include <cassert>
```

- <system_error>：提供 std::error_code 和 std::error_category。
- <string>：用于错误消息的 std::string。
- <iostream>：用于输出。
- <utility>：提供 std::to_underlying。
- <expected>：C++23 的 std::expected，用于返回成功或错误。
- <cassert>：提供 std::abort。
- **自定义错误枚举**

cpp

```cpp
enum class TransactionError {
    OK = 0,
    TemporaryError = 1,
    PermanentError = 2,
};
```

- **TransactionError**：
  - 一个强类型枚举（enum class），表示事务操作的错误状态。
  - 值：
    - OK = 0：无错误。
    - TemporaryError = 1：临时错误，可重试。
    - PermanentError = 2：永久错误，无法恢复。
- **自定义错误类别**

cpp

```cpp
struct TransactionErrorCategory : std::error_category {
    const char* name() const noexcept override {
        return "transaction";
    }
    std::string message(int condition) const override {
        using namespace std::string_literals;
        switch(condition) {
            case 0: return "ok"s;
            case 1: return "temporary error, please retry"s;
            case 2: return "permanent error"s;
        }
        std::abort(); // unreacheable
    }
};
```

- **TransactionErrorCategory**：
  - 继承自 std::error_category，用于定义自定义错误类别。
- **name()**：
  - 返回类别名称 "transaction"，标识这是一个事务相关的错误类别。
  - noexcept 表示此函数不会抛出异常。
- **message(int condition)**：
  - 根据错误值（condition）返回对应的描述字符串。
  - 使用 std::string_literals 的 s 后缀创建 std::string。
  - switch 语句映射：
    - 0 → "ok"。
    - 1 → "temporary error, please retry"。
    - 2 → "permanent error"。
  - 如果传入未知值，调用 std::abort()（理论上不可达，因为枚举值有限）。
- **注册为错误码枚举**

cpp

```cpp
template<> struct std::is_error_code_enum<TransactionError> 
    : public std::true_type{};
```

- **std::is_error_code_enum**：
  - 一个 traits 类，用于告诉标准库某个类型是错误码枚举。
  - 通过特化，声明 TransactionError 是错误码枚举。
  - 继承 std::true_type 表示 true，启用 std::make_error_code 支持。
- **映射到 std::error_code**

cpp

```cpp
std::error_code make_error_code(TransactionError e) {
    static auto category = TransactionErrorCategory{};
    return std::error_code(std::to_underlying(e), category);
}
```

- **make_error_code**：
  - 将 TransactionError 转换为 std::error_code。
  - static auto category：定义一个静态的 TransactionErrorCategory 实例，避免重复构造。
  - std::to_underlying(e)：将枚举值转换为底层整数（C++23 特性）。
  - 返回 std::error_code，包含错误值和类别。
- **隐式转换**：
  - 因为注册了 std::is_error_code_enum，TransactionError 可以直接赋值给 std::error_code，编译器会调用此函数。
- **使用自定义错误**

cpp

```cpp
std::error_code my_function() noexcept {
    return TransactionError::PermanentError;
}
```

- **my_function**：
  - 返回一个 std::error_code，表示永久错误。
  - TransactionError::PermanentError 被隐式转换为 std::error_code（通过 make_error_code）。
  - noexcept 表示不抛出异常。
- **结合 std::expected**

cpp

```cpp
std::expected<void,std::error_code> my_other_function() noexcept {
    return std::unexpected{TransactionError::TemporaryError};
}
```

- **std::expected**：
  - C++23 引入的类型，表示可能成功（void）或失败（std::error_code）的结果。
- **std::unexpected**：
  - 表示错误情况，将 TransactionError::TemporaryError 包装为错误结果。
  - 同样通过 make_error_code 转换为 std::error_code。
- **main 函数**

cpp

```cpp
int main() {
    if (auto err = my_function(); err) {
        std::cout << err.category().name() << " : "
          << err.message() << "\n";
        // prints: "transaction : permanent error"
    }
    if (auto res = my_other_function(); !res) {
        std::cout << res.error().category().name() << " : "
          << res.error().message() << "\n";
        // prints: "transaction : temporary error, please retry"
    }
}
```

- **第一部分：检查 my_function**：
  - err 是 std::error_code，值为 PermanentError（2）。
  - if (err) 检查错误码是否非零（err.value() != 0）。
  - 输出：
    - err.category().name() → "transaction"。
    - err.message() → "permanent error"。
- **第二部分：检查 my_other_function**：
  - res 是 std::expected<void, std::error_code>。
  - !res 检查是否有错误（即 res.has_value() == false）。
  - res.error() 返回错误码（TemporaryError，值为 1）。
  - 输出：
    - res.error().category().name() → "transaction"。
    - res.error().message() → "temporary error, please retry"。

------

为什么这样工作？

1. **自定义错误集成**：
   - 通过继承 std::error_category 和特化 std::is_error_code_enum，自定义错误无缝融入标准库。
2. **make_error_code**：
   - 桥接 TransactionError 和 std::error_code，提供隐式转换。
3. **std::expected**：
   - 利用 std::error_code 表示错误，符合现代 C++ 的错误处理模式（避免异常）。
4. **类型安全**：
   - enum class 和 std::error_code 确保类型安全和清晰的错误表示。

------

输出

```text
transaction : permanent error
transaction : temporary error, please retry
```

------

使用场景

- **事务处理**：如数据库操作，TemporaryError 表示可重试的失败，PermanentError 表示不可恢复的错误。
- **现代错误处理**：结合 std::expected，替代传统的返回值或异常。

------

总结

- 代码定义了自定义错误 TransactionError 和类别 TransactionErrorCategory。
- 通过 make_error_code 和 std::is_error_code_enum，将其集成到 std::error_code。
- 展示了在函数返回和 std::expected 中使用自定义错误。
- 这种方法提供了一种轻量、可扩展的错误处理方式，适用于现代 C++ 编程。