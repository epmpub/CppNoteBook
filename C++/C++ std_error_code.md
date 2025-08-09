# std::error_code

这段代码展示了 C++ 中 <system_error> 和 <future> 库的使用，特别是如何创建和使用错误码（std::error_code）来表示特定错误条件。

代码中还涉及 std::errc 和 std::future_errc 枚举，用于表示通用错误和 std::future 相关的错误。以下是逐步解释。

代码:

```c++
#include <system_error>
#include <future>
#include <iostream>
#include <iomanip>
#include <cassert>
#include <utility>

int main() {
    // Helper for creating error codes that automatically asociates
    // the code with the corresponding category.
    auto err = std::make_error_code(std::errc::not_enough_memory);
    // err.category().name() == "generic"
    // err.message() == "Cannot allocate memory"
    // err.value() == ENOMEM

    // Same as:
    auto err_e = std::error_code(std::to_underlying(std::errc::not_enough_memory), std::generic_category());
    // err == err_e

    assert(err == err_e);

    // Note that the text is implementation specific, 
    // however, specifically for std::errc the values map to errno.

    std::cout << err.category().name() << " error (" << err.value() << ") " << std::quoted(err.message()) << "\n";

    auto future = std::make_error_code(std::future_errc::promise_already_satisfied);
    // future.category().name() == "future"
    // err.message() == "Promise already satisfied"
    
    std::cout << future.category().name() << " error (" << future.value() << ") " << std::quoted(future.message()) << "\n";
}
```



------

代码概览

- 代码展示了两种创建 std::error_code 的方式：
  1. 使用 std::make_error_code 直接从枚举值构造。
  2. 使用 std::error_code 构造函数手动指定值和类别。
- 涉及两种错误类别：
  - std::generic_category()：通用错误类别，与 POSIX errno 对应。
  - std::future_category()：与 std::future 相关的错误类别。

------

关键组件

1. **头文件**

cpp

```cpp
#include <system_error>
#include <future>
#include <utility>
```

- <system_error>：提供 std::error_code、std::errc 等，用于错误处理。
- <future>：提供 std::future_errc，与异步操作相关的错误。
- <utility>：提供 std::to_underlying，将枚举转换为底层整数值。
- **创建通用错误码**

cpp

```cpp
auto err = std::make_error_code(std::errc::not_enough_memory);
// err.category().name() == "generic"
// err.message() == "Cannot allocate memory"
// err.value() == ENOMEM
```

- **std::make_error_code**：
  - 是一个辅助函数，用于从枚举值（如 std::errc）创建 std::error_code。
  - 参数 std::errc::not_enough_memory 是 std::errc 枚举值，表示“内存不足”。
- **结果**：
  - err 是一个 std::error_code 对象。
  - err.category().name() 返回 "generic"，表示这是通用错误类别（与 POSIX errno 相关）。
  - err.message() 返回错误描述，如 "Cannot allocate memory"（具体文本依赖实现）。
  - err.value() 返回底层整数值，对应 POSIX 的 ENOMEM（通常是 12）。
- **细节**：
  - std::errc 的值直接映射到 errno 的值，std::make_error_code 自动关联到 std::generic_category()。
- **手动构造等价错误码**

cpp

```cpp
auto err_e = std::error_code(
  std::to_underlying(std::errc::not_enough_memory),
  std::generic_category());
// err == err_e
```

- **std::error_code**：
  - 构造函数接受两个参数：
    1. 错误值（整数）。
    2. 错误类别（std::error_category 的引用）。
- **std::to_underlying**：
  - C++23 引入的工具，将枚举值（如 std::errc::not_enough_memory）转换为其底层整数值（ENOMEM）。
- **std::generic_category()**：
  - 返回通用错误类别的单例，表示 POSIX 风格的错误。
- **结果**：
  - err_e 与 err 等价，因为它们具有相同的值和类别。
- **目的**：
  - 展示 std::make_error_code 是手动构造的简便替代。
- **注释说明**

cpp

```cpp
// Note that the text is implementation specific, 
// however, specifically for std::errc the values map to errno.
```

- **message() 的文本**：
  - 由实现定义，可能因平台或编译器不同而异（如 "Cannot allocate memory" 或 "Not enough space"）。
- **std::errc 的映射**：
  - std::errc 枚举值（如 not_enough_memory）与 POSIX errno 值（如 ENOMEM）一一对应，保证一致性。
- **创建 future 相关的错误码**

cpp

```cpp
auto future = std::make_error_code(
  std::future_errc::promise_already_satisfied);
// future.category().name() == "future"
// err.message() == "Promise already satisfied"
```

- **std::future_errc**：
  - 定义在 <future> 中，表示与 std::future 和 std::promise 相关的错误。
  - promise_already_satisfied 表示一个 std::promise 已经被设置过值（重复设置是错误）。
- **std::make_error_code**：
  - 从 std::future_errc 创建 std::error_code，自动关联到 std::future_category()。
- **结果**：
  - future 是一个 std::error_code 对象。
  - future.category().name() 返回 "future"，表示这是 std::future 相关的错误类别。
  - future.message() 返回 "Promise already satisfied"（文本依赖实现）。
- **细节**：
  - 与 std::errc 不同，std::future_errc 的值不直接映射到 errno，而是专为异步操作设计。

------

为什么这样工作？

1. **std::error_code 的设计**：
   - 由两部分组成：错误值（整数）和错误类别（std::error_category）。
   - 类别决定如何解释错误值（例如 "generic" 或 "future"）。
2. **std::make_error_code**：
   - 简化了构造过程，自动选择正确的类别（std::generic_category() 或 std::future_category()）。
3. **枚举与类别**：
   - std::errc 对应 POSIX 错误，与 std::generic_category() 关联。
   - std::future_errc 对应异步错误，与 std::future_category() 关联。

------

使用场景

- **std::errc**：
  - 用于表示系统级错误（如文件操作、内存分配失败）。
  - 示例：检查 malloc 失败时返回 std::make_error_code(std::errc::not_enough_memory)。
- **std::future_errc**：
  - 用于异步编程中（如 std::future 或 std::promise 的错误）。
  - 示例：尝试对已设置的 std::promise 再次调用 set_value() 会抛出此错误。

------

输出（假设实现）：

- err.category().name() → "generic"
- err.message() → "Cannot allocate memory"
- err.value() → 12（ENOMEM 的典型值）
- future.category().name() → "future"
- future.message() → "Promise already satisfied"

------

总结

- 这段代码展示了如何使用 std::make_error_code 创建错误码，并将其与适当的类别关联。
- std::errc 表示通用 POSIX 错误，映射到 errno。
- std::future_errc 表示异步操作的特定错误。
- std::error_code 提供了一种轻量、类型安全的方式来处理错误，广泛用于现代 C++ 编程中。