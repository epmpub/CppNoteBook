`std::to_underlying` 和 `std::is_scoped_enum` 都是围绕 **enum（枚举）** 改进的现代 C++ 工具。

- `std::to_underlying`：C++23
- `std::is_scoped_enum`：C++23

它们主要解决：

> 类型安全枚举（`enum class`）使用不方便的问题。

------

# 1. std::to_underlying

头文件：

```cpp
#include <utility>
```

定义大致为：

```cpp
template<class Enum>
constexpr std::underlying_type_t<Enum>
to_underlying(Enum e) noexcept;
```

等价于：

```cpp
template<class Enum>
constexpr auto to_underlying(Enum e) noexcept
{
    return static_cast<std::underlying_type_t<Enum>>(e);
}
```

------

## 为什么需要它

### 传统写法

```cpp
enum class Color
{
    Red = 1,
    Green = 2,
    Blue = 3
};

int value = static_cast<int>(Color::Green);
```

必须写：

```cpp
static_cast<int>
```

如果底层类型改了：

```cpp
enum class Color : uint8_t
{
    Red,
    Green,
    Blue
};
```

那么：

```cpp
static_cast<int>(Color::Green)
```

已经不是底层类型了。

------

### C++23

```cpp
int value = std::to_underlying(Color::Green);
```

或者：

```cpp
auto value = std::to_underlying(Color::Green);
```

结果：

```cpp
value == 2
```

且类型自动匹配底层类型。

------

## 示例

```cpp
#include <iostream>
#include <utility>

enum class Color : unsigned char
{
    Red = 1,
    Green = 2,
    Blue = 3
};

int main()
{
    auto v = std::to_underlying(Color::Green);

    std::cout << +v << '\n';
}
```

输出：

```text
2
```

这里：

```cpp
decltype(v)
```

实际上是：

```cpp
unsigned char
```

不是 `int`。

------

## 位标志（Flags）场景

这是最常见用途。

```cpp
enum class Permission : uint32_t
{
    Read  = 1,
    Write = 2,
    Exec  = 4
};
```

以前：

```cpp
auto mask =
    static_cast<uint32_t>(Permission::Read)
  | static_cast<uint32_t>(Permission::Write);
```

现在：

```cpp
auto mask =
    std::to_underlying(Permission::Read)
  | std::to_underlying(Permission::Write);
```

可读性明显更好。

------

# 2. std::is_scoped_enum

头文件：

```cpp
#include <type_traits>
```

定义：

```cpp
template<class T>
struct is_scoped_enum;
```

变量模板：

```cpp
std::is_scoped_enum_v<T>
```

------

## 什么是 Scoped Enum

C++11 引入：

```cpp
enum class Color
{
    Red,
    Green,
    Blue
};
```

称为：

```text
Scoped Enumeration
```

即：

```cpp
enum class
enum struct
```

------

与传统枚举区别：

### 普通 enum

```cpp
enum Color
{
    Red,
    Green,
    Blue
};
```

允许：

```cpp
int x = Red;
```

自动转换。

------

### enum class

```cpp
enum class Color
{
    Red,
    Green,
    Blue
};
```

不允许：

```cpp
int x = Color::Red; // ERROR
```

必须：

```cpp
int x = std::to_underlying(Color::Red);
```

------

## 判断示例

```cpp
#include <type_traits>
#include <iostream>

enum OldColor
{
    Red
};

enum class NewColor
{
    Red
};

int main()
{
    std::cout
        << std::boolalpha
        << std::is_scoped_enum_v<OldColor>
        << '\n';

    std::cout
        << std::is_scoped_enum_v<NewColor>
        << '\n';
}
```

输出：

```text
false
true
```

------

# 与 is_enum 的区别

假设：

```cpp
enum OldColor {};
enum class NewColor {};
```

结果：

| Trait              | OldColor | NewColor |
| ------------------ | -------- | -------- |
| `is_enum_v`        | true     | true     |
| `is_scoped_enum_v` | false    | true     |

即：

```cpp
std::is_enum_v<T>
```

判断：

```text
是不是枚举
```

而：

```cpp
std::is_scoped_enum_v<T>
```

判断：

```text
是不是 enum class
```

------

# 实际用途

写泛型库时经常需要区分：

```cpp
enum Old {};
enum class New {};
```

例如：

```cpp
template<typename T>
void print(T value)
{
    if constexpr (std::is_scoped_enum_v<T>)
    {
        std::cout
            << std::to_underlying(value);
    }
    else
    {
        std::cout << value;
    }
}
```

------

# 两者配合使用

非常常见：

```cpp
template<typename E>
requires std::is_scoped_enum_v<E>
constexpr auto enum_value(E e)
{
    return std::to_underlying(e);
}
```

使用：

```cpp
enum class Color : uint8_t
{
    Red = 1,
    Green = 2
};

auto x = enum_value(Color::Green);
```

得到：

```cpp
uint8_t{2}
```

------

# 标准库可能的实现

```
std::to_underlying
template<class Enum>
constexpr auto to_underlying(Enum e) noexcept
{
    return static_cast<
        std::underlying_type_t<Enum>
    >(e);
}
```

------

```
std::is_scoped_enum
```

大致逻辑：

```cpp
is_enum_v<T>
&&
!is_convertible_v<T,int>
```

因为：

```cpp
enum
```

可隐式转 `int`，

而：

```cpp
enum class
```

不可隐式转 `int`。

实际实现会使用编译器内建特性（intrinsics），更高效且更准确。

------

总结：

| 特性                       | 标准  | 作用                    |
| -------------------------- | ----- | ----------------------- |
| `std::to_underlying(e)`    | C++23 | 获取枚举底层整数值      |
| `std::is_scoped_enum_v<T>` | C++23 | 判断是否为 `enum class` |
| `std::is_enum_v<T>`        | C++11 | 判断是否为枚举类型      |

典型组合：

```cpp
template<typename E>
requires std::is_scoped_enum_v<E>
auto value = std::to_underlying(e);
```

这是现代 C++ 处理类型安全枚举（`enum class`）的推荐方式。