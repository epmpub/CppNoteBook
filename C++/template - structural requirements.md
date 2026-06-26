##  structural requirements

```cpp
template<std::string_view Str>
```

在 C++20 中，虽然 NTTP（Non-Type Template Parameter，非类型模板参数）扩展支持了类类型，但要求该类型必须是“结构化类型（structural type）”。

而 `std::string_view` 不是 structural type，因此不能直接作为 NTTP。

正确做法通常有两种：

1. 使用 `const char[N]` 包装
2. 自定义一个 structural fixed_string

这是 C++20 中最常见的方案。

可正常编译的版本如下：

```cpp
#include <iostream>
#include <string_view>

// C++20 fixed_string
template <size_t N>
struct fixed_string
{
    char value[N];

    constexpr fixed_string(const char (&str)[N])
    {
        for (size_t i = 0; i < N; ++i)
        {
            value[i] = str[i];
        }
    }

    constexpr operator std::string_view() const
    {
        return { value, N - 1 };
    }
};

// NTTP
template <fixed_string Str>
void print_message()
{
    std::cout << std::string_view(Str) << "\n";
}

int main()
{
    print_message<"Hello, C++20 NTTP!">();
    constexpr fixed_string msg = "Template String";
    print_message<msg>();
}
```

输出：

```text
Hello, C++20 NTTP!
Template String
```

这里的核心机制是：

```cpp
template <fixed_string Str>
```

其中：

```cpp
fixed_string
```

是一个满足 C++20 structural type 要求的字面量类，因此可以作为 NTTP。

这也是很多现代 C++ 库（例如：

- fmt
- CTRE
- Boost.Hana

）实现编译期字符串的基础技术。

另外，你的示例：

```cpp
constexpr std::string_view msg = "Template String";
print_message<msg>();
```

即使在 GCC 14 / Clang 18 / MSVC 最新版下也仍然非法，因为：

- `std::string_view` 不是 structural type
- 它内部含有指针成员
- 标准库实现也未声明其满足 structural requirements

