# std::move_only_function

std::move_only_function 是 C++23 引入的一个功能，定义在 <functional> 头文件中。它是 std::function 的一个变体，专门用于存储仅支持移动的可调用对象（move-only callables）。与 std::function 不同，std::move_only_function 不要求可调用对象是可复制的（copyable），因此更适合处理现代 C++ 中常见的移动语义场景，例如 lambda 表达式捕获的 unique_ptr 或其他不可复制的资源。

以下是对 std::move_only_function 的详细解释：

------

定义

cpp

```cpp
#include <functional>

namespace std {
    template<class R, class... Args>
    class move_only_function<R(Args...) noexcept(/* see below */)>;
}
```

- **R**: 返回类型。
- **Args...**: 参数类型。
- **noexcept**: 可选的异常规格（见下文）。
- 类似 std::function，它是一个类型擦除（type-erased）的包装器，但仅支持移动语义。

语法

cpp

```cpp
std::move_only_function<int(int)> f = /* move-only callable */;
```

- 支持的调用签名：R(Args...)。

------

行为

- **std::move_only_function**：
  - 存储一个可调用对象（如函数指针、lambda、仿函数）。
  - 只能通过移动构造或移动赋值操作，不能复制。
  - 调用时通过 operator() 执行存储的可调用对象。
- **类型擦除**：
  - 隐藏底层可调用对象的具体类型，只保留调用签名。
- **异常规格**：
  - 可指定 noexcept，要求存储的可调用对象满足该规格。

------

前提条件

- **可调用对象**：
  - 必须支持调用签名 R(Args...)。
  - 可以是不可复制的（如含 std::unique_ptr 的 lambda）。
- **移动性**：
  - 必须支持移动构造或移动赋值。
- **存储**：
  - 小对象优化（SBO）：若可调用对象足够小，可能存储在对象内部，否则动态分配。

------

示例代码

示例 1：基本用法（移动 lambda）

cpp

```cpp
#include <functional>
#include <iostream>
#include <memory>

int main() {
    auto lambda = [p = std::make_unique<int>(42)](int x) {
        return *p + x;
    };

    std::move_only_function<int(int)> f = std::move(lambda);

    std::cout << "Result: " << f(10) << "\n"; // 调用

    // std::move_only_function<int(int)> f2 = f; // 错误：不可复制
    std::move_only_function<int(int)> f3 = std::move(f); // 正确：移动

    std::cout << "Moved result: " << f3(5) << "\n";

    return 0;
}
```

输出

```text
Result: 52
Moved result: 47
```

- **解释**：
  - Lambda 捕获 unique_ptr，不可复制。
  - f 通过移动存储 lambda，f3 通过移动从 f 接管。

示例 2：与 noexcept

cpp

```cpp
#include <functional>
#include <iostream>

int main() {
    auto lambda = [](int x) noexcept { return x * 2; };

    std::move_only_function<int(int) noexcept> f = std::move(lambda);

    std::cout << "Result: " << f(5) << "\n";

    return 0;
}
```

输出

```text
Result: 10
```

- **解释**：
  - 指定 noexcept，要求 lambda 不抛异常。

示例 3：空状态

cpp

```cpp
#include <functional>
#include <iostream>

int main() {
    std::move_only_function<int(int)> f;

    if (!f) {
        std::cout << "Function is empty\n";
    }

    f = [](int x) { return x + 1; };
    std::cout << "Result: " << f(10) << "\n";

    return 0;
}
```

输出

```text
Function is empty
Result: 11
```

- **解释**：
  - 默认构造为空，可通过移动赋值填充。

------

返回类型与存储

- **存储**：
  - 使用类型擦除，类似 std::function，但不要求复制。
  - 小对象优化（SBO）：避免动态分配小型可调用对象。
- **调用**：
  - R operator()(Args...)。
  - 若为空，调用未定义行为（需检查）。

------

时间复杂度

- **构造/移动**：通常 O(1)，若动态分配则 O(1) 加内存分配成本。
- **调用**：O(1)，与底层可调用对象的复杂度无关。

------

与 std::function 的对比

| 特性     | std::function    | std::move_only_function |
| -------- | ---------------- | ----------------------- |
| 引入版本 | C++11            | C++23                   |
| 复制支持 | 是               | 否（仅移动）            |
| 移动支持 | 是               | 是                      |
| 适用场景 | 可复制可调用对象 | 仅移动可调用对象        |
| 异常规格 | 无显式支持       | 支持 noexcept           |
| 空状态   | 支持             | 支持                    |

------

使用场景

1. **移动语义**：

   - 存储含 unique_ptr 或其他不可复制资源的 lambda。

   cpp

   ```cpp
   auto f = std::move_only_function<void()>([p = std::make_unique<int>(42)]{});
   ```

2. **性能优化**：

   - 避免不必要的复制，提升效率。

3. **现代 C++**：

   - 与移动语义和 RAII 集成。

------

注意事项

1. **C++23 要求**：
   - 需要 -std=c++23 和支持 C++23 的编译器。
2. **不可复制**：
   - 尝试复制会导致编译错误。
3. **空状态检查**：
   - 使用 if (f) 检查是否可调用，避免未定义行为。
4. **异常规格**：
   - 若指定 noexcept，可调用对象必须符合。

------

替代方案（C++20 前）

在 C++20 或更早版本中，可以用 std::function（需复制）或手动实现：

cpp

```cpp
#include <memory>
#include <iostream>

struct MoveOnlyFunc {
    std::unique_ptr<int> p;
    int operator()(int x) const { return *p + x; }
    MoveOnlyFunc() : p(std::make_unique<int>(42)) {}
    MoveOnlyFunc(MoveOnlyFunc&&) = default;
    MoveOnlyFunc(const MoveOnlyFunc&) = delete;
};

int main() {
    MoveOnlyFunc f;
    std::cout << f(10) << "\n"; // 52
    return 0;
}
```

- **缺点**：无类型擦除，需手动管理。

------

总结

std::move_only_function 是 C++23 中一个现代化的可调用对象包装器，专为移动语义设计。它弥补了 std::function 对不可复制对象的限制，支持 unique_ptr 等场景，适合现代 C++ 编程。如果你有具体问题或想探讨用法，请告诉我！