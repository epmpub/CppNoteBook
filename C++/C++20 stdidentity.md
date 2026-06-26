#  C++20 std::identity

在 C++ 中，std::identity 是 C++20 引入的一个函数对象（functor），定义在 <functional> 头文件中。

它是一个简单的工具，用于表示“恒等函数”（identity function），即**直接返回输入值而不做任何修改**。

std::identity 主要用于 Ranges 库中，作为默认的投影函数（projection），但也可以在其他需要函数对象的场景中使用。

以下是对 std::identity 的详细解释。

------

定义

cpp

```cpp
namespace std {
    struct identity {
        template<typename T>
        constexpr T&& operator()(T&& t) const noexcept {
            return std::forward<T>(t);
        }

        using is_transparent = void; // 表示支持透明比较
    };
}
```

- **operator()**：
  - 接受任意类型的参数 T&&（通用引用），返回相同的参数。
  - 使用 std::forward 保持值类别（lvalue 或 rvalue）。
- **constexpr 和 noexcept**：
  - 可在编译期执行，无异常抛出。
- **is_transparent**：
  - 表示该函数对象支持透明操作（常用于比较器或哈希函数，但此处仅标记）。

------

功能

- **作用**：将输入值原样返回，不做任何转换或投影。
- **返回值**：保持输入的值类别（左值引用、右值引用等）。

------

示例代码

以下是一些使用 std::identity 的例子：

1. **基本用法**

cpp

```cpp
#include <functional>
#include <iostream>

int main() {
    std::identity id{};
    int x = 42;
    std::cout << id(x) << "\n"; // 输出: 42
    std::cout << id(3.14) << "\n"; // 输出: 3.14
}
```

- id(x) 直接返回 x，无任何修改。
- **与 Ranges 库结合**

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>
#include <functional>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    auto result = std::ranges::min_element(vec, std::less<>{}, std::identity{});
    std::cout << *result << "\n"; // 输出: 1
}
```

- **std::ranges::min_element**：
  - 第三个参数是投影函数，默认是 std::identity。
  - 这里显式使用 std::identity，表示直接比较元素本身。
- **投影对比**

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>
#include <functional>

struct Data {
    int value;
};

int main() {
    std::vector<Data> vec = {{1}, {3}, {2}};
    
    // 使用 std::identity，直接比较 Data 对象（需要定义 < 运算符）
    // auto min1 = std::ranges::min_element(vec, std::less<>{}, std::identity{});
    // 假设 Data 未定义 <，这会编译失败

    // 使用成员投影
    auto min2 = std::ranges::min_element(vec, std::less<>{}, &Data::value);
    std::cout << min2->value << "\n"; // 输出: 1
}
```

- **std::identity**：直接返回 Data，需要 Data 支持比较。
- **成员指针**：投影到 value，比较具体字段。

------

为什么使用 std::identity？

1. **默认投影**：
   - Ranges 算法（如 std::ranges::sort、std::ranges::min_element）接受投影函数，std::identity 是默认值，表示不投影。
2. **一致性**：
   - 提供统一的函数对象接口，避免手写 lambda（如 [](auto&& x) { return std::forward<decltype(x)>(x); }）。
3. **性能**：
   - constexpr 和 noexcept，编译器可优化为零成本抽象。

------

与 Lambda 的对比

可以用 lambda 替代 std::identity，但后者更简洁：

cpp

```cpp
auto id_lambda = [](auto&& x) { return std::forward<decltype(x)>(x); };
std::identity id{};
int x = 42;
id(x);      // 等价于
id_lambda(x);
```

- **std::identity** 是标准化的，避免手动定义。

------

使用场景

- **Ranges 算法**：
  - 当不需要投影时，显式指定 std::identity 以提高可读性。
- **泛型编程**：
  - 作为占位符，允许在需要函数对象的地方传递“无操作”。
- **调试**：
  - 测试算法行为时，确保不引入额外变换。

------

总结

- **std::identity** 是一个简单的恒等函数对象，返回输入值。
- 在 C++20 的 Ranges 库中常用作默认投影。
- 支持通用引用，保持值类别，性能高效。
- 提供了标准化的方式，避免手写等价 lambda。

如果你有具体问题或需要更多示例，请告诉我！