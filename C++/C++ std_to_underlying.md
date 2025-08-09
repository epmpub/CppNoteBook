# std::to_underlying 

std::to_underlying 是 C++23 引入的一个实用函数，定义在 <utility> 头文件中。它的作用是将枚举类型（enum）的值转换为其底层整数类型的值。这个函数简化了从枚举值到整数的显式转换，同时提高了代码的可读性和安全性。

以下是对 std::to_underlying 的详细解释：

------

定义

cpp

```cpp
#include <utility>

template <typename E>
constexpr std::underlying_type_t<E> to_underlying(E e) noexcept;
```

- **E**：一个枚举类型（enum 或 enum class）。
- **返回值**：E 的底层类型的值（由 std::underlying_type_t<E> 确定）。
- **noexcept**：保证不抛出异常。

------

背景

- 在 C++ 中，枚举类型的值通常由某种整数类型（如 int、unsigned int 等）表示，这个类型称为“底层类型”（underlying type）。

- 在 C++23 之前，要获取枚举的底层值，通常需要显式使用 static_cast，例如：

  cpp

  ```cpp
  static_cast<int>(MyEnum::Value);
  ```

- std::to_underlying 提供了一种更简洁、更语义化的方式，避免手动指定底层类型。

------

使用方式

std::to_underlying 接受一个枚举值，返回其底层类型的值。底层类型可以通过 std::underlying_type（C++11 引入的类型特性）推导。

------

示例代码

示例 1：基本枚举

cpp

```cpp
#include <iostream>
#include <utility>

enum Color {
    Red = 1,
    Green = 2,
    Blue = 3
};

int main() {
    Color c = Green;
    auto value = std::to_underlying(c);
    std::cout << "Green 的底层值: " << value << std::endl;
    std::cout << "底层类型大小: " << sizeof(std::underlying_type_t<Color>) << " 字节" << std::endl;
    return 0;
}
```

输出

```text
Green 的底层值: 2
底层类型大小: 4 字节
```

示例 2：枚举类（enum class）

cpp

```cpp
#include <iostream>
#include <utility>

enum class Status : short {
    Off = 0,
    On = 1
};

int main() {
    Status s = Status::On;
    auto value = std::to_underlying(s);
    std::cout << "On 的底层值: " << value << std::endl;
    std::cout << "底层类型大小: " << sizeof(std::underlying_type_t<Status>) << " 字节" << std::endl;
    return 0;
}
```

输出

```text
On 的底层值: 1
底层类型大小: 2 字节
```

------

关键特性

1. **类型安全性**：
   - std::to_underlying 自动推导底层类型，避免了手动指定类型时的错误。
   - 例如，static_cast<int> 可能与底层类型不匹配，而 std::to_underlying 总是正确的。
2. **constexpr 和 noexcept**：
   - 可以在编译时使用，且保证不抛出异常，适合性能敏感场景。
3. **支持任意枚举**：
   - 适用于普通枚举（enum）和强类型枚举（enum class）。
   - 支持自定义底层类型（如 enum E : char）。

------

与 static_cast 的对比

使用 static_cast

cpp

```cpp
enum class MyEnum : short { A = 1 };
auto value = static_cast<short>(MyEnum::A); // 手动指定底层类型
```

使用 std::to_underlying

cpp

```cpp
auto value = std::to_underlying(MyEnum::A); // 自动推导底层类型
```

- **优势**：
  - 更简洁，语义更明确。
  - 无需显式指定底层类型，减少出错可能。

------

使用场景

1. **与底层值交互**：

   - 当需要将枚举值传递给需要整数的接口（如位运算、序列化）时。

   cpp

   ```cpp
   enum class Flags : unsigned { Read = 1, Write = 2 };
   unsigned combined = std::to_underlying(Flags::Read) | std::to_underlying(Flags::Write);
   ```

2. **调试或日志**：

   - 输出枚举的底层值以便调试。

   cpp

   ```cpp
   std::cout << "Enum value: " << std::to_underlying(some_enum) << '\n';
   ```

3. **与 C 风格 API 交互**：

   - 将枚举值转换为整数以传递给需要底层类型的函数。

------

注意事项

1. **C++23 要求**：
   - std::to_underlying 是 C++23 特性，需要支持 C++23 的编译器（如 GCC 13+、Clang 16+）。
2. **仅限枚举**：
   - 参数必须是枚举类型，否则会导致编译错误。
3. **底层类型默认**：
   - 如果未显式指定底层类型（如 enum E {}），默认是 int。

------

底层实现（概念性）

std::to_underlying 的实现非常简单，类似于：

cpp

```cpp
template <typename E>
constexpr std::underlying_type_t<E> to_underlying(E e) noexcept {
    return static_cast<std::underlying_type_t<E>>(e);
}
```

- 它依赖 std::underlying_type_t（C++11 的类型特性）来获取底层类型。

------

总结

std::to_underlying 是一个小而美的 C++23 工具，用于将枚举值转换为其底层整数类型。它替代了繁琐的 static_cast，提供了更清晰的语义和类型安全性。适用于需要操作枚举底层值的场景，尤其在与低级代码交互时。如果你有具体问题或使用案例，欢迎进一步探讨！

sample code:

```cpp
#include <utility>
#include <type_traits>
#include <cstdint>
#include <typeinfo>
#include <iostream>
enum Color {
    RED,
    GREEN,
    BLUE
};

enum Info {
    LIMIT = UINT64_MAX
};

enum class MyColor {
    RED,
    GREEN,
    BLUE
};


/*

int main()
{
    constexpr auto a = std::to_underlying(RED);
    // a == 0, sizeof(a) == sizeof(int) (concrete type is impl.defined)

    constexpr auto b = std::to_underlying(LIMIT);
    // b == UINT64_MAX, decltype(b) an unsigned type of size uint64_t

    // Before C++23, we would have to rely on static_cast
    auto c = static_cast<std::underlying_type<Info>::type>(LIMIT); // C++11
    auto d = static_cast<std::underlying_type_t<Info>>(LIMIT); // C++14

	static_assert(std::is_same_v<decltype(c), int>);

    //print typeid name of d
	std::cout << typeid(d).name() << std::endl;
    
    // b == c == d

    // Scoped enumerations have a well defined underlying type
    constexpr auto e = std::to_underlying(MyColor::GREEN);
    // e == 1, decltype(e) == int
    return 0;
}


*/



#include <cstdint>
#include <iomanip>
#include <iostream>
#include <type_traits>
#include <utility>

enum class E1 : char { e };
static_assert(std::is_same_v<char, decltype(std::to_underlying(E1::e))>);

enum struct E2 : long { e };
static_assert(std::is_same_v<long, decltype(std::to_underlying(E2::e))>);

enum E3 : unsigned { e };
static_assert(std::is_same_v<unsigned, decltype(std::to_underlying(e))>);

int main()
{
    enum class ColorMask : std::uint32_t
    {
        red = 0xFF, green = (red << 8), blue = (green << 8), alpha = (blue << 8)
    };

    std::cout << std::hex << std::uppercase << std::setfill('0')
        << std::setw(8) << std::to_underlying(ColorMask::red) << '\n'
        << std::setw(8) << std::to_underlying(ColorMask::green) << '\n'
        << std::setw(8) << std::to_underlying(ColorMask::blue) << '\n'
        << std::setw(8) << std::to_underlying(ColorMask::alpha) << '\n';

    //  std::underlying_type_t<ColorMask> x = ColorMask::alpha; // Error: no known conversion
    [[maybe_unused]]
    constexpr std::underlying_type_t<ColorMask> y = std::to_underlying(ColorMask::alpha); // OK
}
```

