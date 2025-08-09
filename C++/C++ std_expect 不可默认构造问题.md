# std::expect 不可默认构造问题

这段代码展示了 C++23 中引入的 std::expected 类型，结合一个**不可默认构造的类型** NoDefault，说明其构造行为。

以下是对代码的逐步解释。

------

完整代码（假设）

cpp

```cpp
#include <expected>
#include <iostream>

struct NoDefault {
    NoDefault(int) {}
};

int main() {
    // std::expected<NoDefault, int> n; // 编译失败
    std::expected<NoDefault, int> n{20};

    std::cout << "Has value: " << std::boolalpha << n.has_value() << "\n";
    if (n.has_value()) {
        std::cout << "Value: " << n.value() << "\n"; // 注意：需定义 operator<<
    }

    return 0;
}
```

------

逐步解释

**std::expected 简介**

- **std::expected<T, E>**：
  - C++23 引入，定义在 <expected> 中。
  - 表示一个可能成功（持有 T）或失败（持有 E）的结果。
  - T 是预期值类型，E 是错误类型。
- **状态**：
  - **has_value() == true：持有 T。**
  - **has_value() == false：持有 E。**

------

**定义 NoDefault**

cpp

```cpp
struct NoDefault {
    NoDefault(int) {}
};
```

- **NoDefault**：
  - 自定义类型，仅提供带参数构造函数（NoDefault(int)）。
  - 无默认构造函数（隐式默认构造被禁用）。
- **影响**：
  - 不能默认构造 NoDefault 对象（如 NoDefault n; 会编译失败）。

------

**默认构造问题**

cpp

```cpp
// std::expected<NoDefault, int> n; // 编译失败
```

- **std::expected 的默认构造**：
  - 默认构造时，std::expected<T, E> 会尝试默认构造 T（即 NoDefault）。
  - 因为 NoDefault 不可默认构造，std::expected<NoDefault, int> 的默认构造函数不可用。
- **注释**：
  - “If the result type cannot be default constructed, the resulting std::expected cannot be default constructed either.”
  - 说明 T 的构造限制直接影响 std::expected。

------

## **显式构造**

cpp

```cpp
std::expected<NoDefault, int> n{20};
// n.has_value() == true
```

- **n{20}**：
  - 使用初始化列表构造 std::expected<NoDefault, int>。
  - 20 被隐式转换为 NoDefault（通过 NoDefault(int) 构造函数）。
  - std::expected 存储 T（NoDefault），而不是 E（int）。
- **状态**：
  - n.has_value() == true：表示 n 持有 NoDefault 值。
  - n.value()：返回 NoDefault{20}。
- **验证**：
  - n.error() 不可访问（未持有错误）。

------

**输出（假设扩展）**

若添加打印：

```text
Has value: true
```

- **说明**：
  - n 成功构造为持有 NoDefault{20}。
  - 注意：直接打印 n.value() 需要 NoDefault 定义 operator<<，否则需访问成员或扩展代码。

------

关键点分析

1. **std::expected 的构造**：
   - 默认构造依赖 T 的默认构造。
   - 显式构造可通过 T 的构造函数初始化。
2. **NoDefault**：
   - 无默认构造函数，限制了 std::expected 的默认构造。
3. **初始化**：
   - n{20} 构造 NoDefault{20}，存储为预期值。

------

时间复杂度

- **构造**：O(1)，依赖 NoDefault 构造函数。
- **has_value()**：O(1)。

------

使用场景

1. **错误处理**：

   - 用 std::expected 表示成功（T）或失败（E）。

   cpp

   ```cpp
   std::expected<int, std::string> result = 42; // 成功
   std::expected<int, std::string> error{std::unexpect, "fail"}; // 失败
   ```

2. **非默认构造类型**：

   - 处理像 NoDefault 这样的类型。

------

注意事项

1. **C++23 要求**：
   - 需要 -std=c++23 和支持 <expected> 的编译器。
2. **类型限制**：
   - T 和 E 需满足移动构造要求。
3. **访问**：
   - value() 在无值时抛出 std::bad_expected_access。

------

总结

- **std::expected<NoDefault, int>**：
  - 不可默认构造，因 NoDefault 无默认构造函数。
  - 可通过 n{20} 初始化，持有 NoDefault{20}。
- **n.has_value() == true**：
  - 表示成功构造为预期值。 C++23 的 std::expected 提供了一种现代的错误处理方式，灵活适应各种类型。如果你有具体问题或想扩展代码，请告诉我！