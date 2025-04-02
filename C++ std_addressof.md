# std::addressof

std::addressof 是 C++ 标准库中的一个工具函数，定义在 <memory> 头文件中。它用于获取对象的真实地址（即内存地址），即使该对象重载了取地址运算符（operator&）。这是 std::addressof 相对于普通 & 运算符的主要优势，尤其在处理特殊类型时非常有用。

以下是对 std::addressof 的详细解释：

------

定义

cpp

```cpp
#include <memory>

namespace std {
    template<class T>
    constexpr T* addressof(T& r) noexcept;

    template<class T>
    const T* addressof(const T&&) = delete;
}
```

- **T**: 对象类型。
- **r**: 对象的引用。
- 返回值：T*，指向 r 的指针。
- **noexcept**：保证不抛出异常。
- **删除右值版本**：防止对临时对象使用。

------

行为

- **std::addressof**：

  - 返回对象的真实内存地址。
  - 绕过重载的 operator&，直接访问底层地址。

- **实现方式**：

  - 通常使用 reinterpret_cast 或类似技巧，确保获取原始地址。

  - 标准实现可能类似：

    cpp

    ```cpp
    template<class T>
    constexpr T* addressof(T& r) noexcept {
        return reinterpret_cast<T*>(&reinterpret_cast<char&>(r));
    }
    ```

------

前提条件

- **参数**：
  - 必须是左值引用（T&），不能是右值（T&&）。
- **对象**：
  - 可以是任何类型，包括内置类型和自定义类型。

------

示例代码

示例 1：基本用法

cpp

```cpp
#include <memory>
#include <iostream>

int main() {
    int x = 42;
    int* ptr = std::addressof(x);

    std::cout << "Address: " << ptr << "\n";
    std::cout << "Value: " << *ptr << "\n";

    return 0;
}
```

输出

```text
Address: 0x7ffee4c0a4ac  // 地址因环境而异
Value: 42
```

- **解释**：
  - 获取 x 的地址，与 &x 等价。

示例 2：重载 operator&

cpp

```cpp
#include <memory>
#include <iostream>

struct Sneaky {
    Sneaky(int v) : value(v) {}
    int value;
    int* operator&() const { // 重载取地址运算符
        std::cout << "Sneaky operator& called\n";
        return nullptr; // 返回假地址
    }
};

int main() {
    Sneaky s(42);

    std::cout << "Using &: " << &s << "\n";              // 调用重载
    std::cout << "Using addressof: " << std::addressof(s) << "\n"; // 真实地址

    std::cout << "Value via addressof: " << std::addressof(s)->value << "\n";

    return 0;
}
```

输出

```text
Sneaky operator& called
Using &: 0
Using addressof: 0x7ffee4c0a4ac  // 真实地址
Value via addressof: 42
```

- **解释**：
  - &s 调用重载的 operator&，返回 nullptr。
  - std::addressof(s) 绕过重载，获取 s 的真实地址。

------

时间复杂度

- **O(1)**：
  - 编译期优化，直接返回地址，无运行时开销。

------

使用场景

1. **重载 operator& 的类型**：

   - 当类自定义了取地址运算符，需确保获取真实地址。

   cpp

   ```cpp
   T* ptr = std::addressof(obj);
   ```

2. **模板编程**：

   - 在泛型代码中安全获取地址，避免类型依赖问题。

3. **指针操作**：

   - 与智能指针（如 std::unique_ptr）或低级内存操作配合。

------

注意事项

1. **C++11 起可用**：

   - 自 C++11 引入，C++20 增加 constexpr 支持。

2. **只接受左值**：

   - 右值（如临时对象）被删除：

     cpp

     ```cpp
     std::addressof(42); // 编译错误
     ```

3. **noexcept**：

   - 保证不抛异常，适合实时系统。

4. **与 & 的区别**：

   - & 可能被重载，std::addressof 始终返回真实地址。

------

与普通 & 的对比

| 特性     | &                    | std::addressof |
| -------- | -------------------- | -------------- |
| 类型支持 | 任意左值             | 任意左值       |
| 重载处理 | 调用 operator&       | 绕过 operator& |
| 右值     | 支持（返回右值地址） | 禁用           |
| 异常     | 无（除非重载抛出）   | noexcept       |
| 复杂度   | O(1)                 | O(1)           |

------

总结

std::addressof 是 C++ 中一个可靠的工具，用于获取对象的真实地址：

- 绕过重载的 operator&，确保正确性。
- 返回类型为 T*，仅接受左值引用。
- 自 C++11 起可用，广泛用于模板和低级编程。 它是处理复杂类型或需要地址可靠性时的首选工具。如果你有具体问题或想探讨用法，请告诉我！