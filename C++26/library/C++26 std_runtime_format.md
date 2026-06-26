Dynamic format strings

```c
#include <format>
#include <print>
#include <string>
#include <string_view>
 
int main()
{
    std::print("Hello {}!\n", "world");
 
    std::string fmt;
    for (int i{}; i != 3; ++i)
    {
        fmt += "{} "; // constructs the formatting string
        std::print("{} : ", fmt);
        std::println(std::runtime_format(fmt), "alpha", 'Z', 3.14, "unused");
    }
}
```

`Dynamic format strings` 是 C++26 对 `<format>` 的一次重要增强，来源于 **P2918R2 Runtime format strings**。它解决了 C++20/23 `std::format` 一个长期存在的痛点：

> 想在运行时决定格式字符串时，要么编译不过，要么必须绕过类型检查。

### C++20/23 的问题

为了获得编译期格式检查，`std::format` 的签名是：

```cpp
template<class... Args>
std::string format(
    std::format_string<Args...> fmt,
    Args&&... args
);
```

例如：

```cpp
std::format("{} {}", 1, 2);
```

编译器会检查：

```text
{} 的数量
参数数量
格式说明符是否合法
类型是否匹配
```

------

但如果格式串来自运行时：

```cpp
std::string fmt = "{} {}";

std::format(fmt, 1, 2);
```

C++23 编译失败：

```text
fmt is not a constant expression
```

因为：

```cpp
std::format_string<Args...>
```

要求编译期常量。

------

### C++23 的解决办法

只能使用：

```cpp
std::vformat()
```

例如：

```cpp
std::string fmt = "{} {}";

auto s =
    std::vformat(
        fmt,
        std::make_format_args(1,2)
    );
```

缺点：

- 写法繁琐
- 需要 `make_format_args`
- 与 `std::format` 风格不一致

------

## C++26 新增 `std::runtime_format`

新增：

```cpp
std::runtime_format(...)
```

用于显式告诉库：

```text
这个格式串来自运行时
不要进行编译期检查
```

例如：

```cpp
std::string fmt = "{} {}";

auto s =
    std::format(
        std::runtime_format(fmt),
        1,
        2
    );
```

合法。

------

### 最简单示例

```cpp
#include <format>
#include <print>

int main()
{
    std::string fmt = "[{}] {}";

    auto s =
        std::format(
            std::runtime_format(fmt),
            100,
            "hello"
        );

    std::println("{}", s);
}
```

输出：

```text
[100] hello
```

------

### 为什么不直接允许 std::string？

委员会故意没有这样做：

```cpp
std::format(fmt, args...)
```

因为会丢失编译期检查。

例如：

```cpp
std::format(
    "{} {} {}",
    1,
    2
);
```

当前：

```text
编译错误
```

非常安全。

如果允许任意字符串：

```cpp
std::string fmt = "{} {} {}";
```

错误只能运行时发现。

------

因此设计成显式 opt-in：

```cpp
std::runtime_format(fmt)
```

开发者明确表示：

```text
我知道这是运行时格式串
接受运行时检查
```

------

## 与 Python 的类比

Python：

```python
fmt = "{} {}"

fmt.format(1,2)
```

天然运行时。

C++20：

```cpp
std::format("{} {}",1,2);
```

天然编译期检查。

C++26：

```cpp
std::format(
    std::runtime_format(fmt),
    1,
    2
);
```

兼顾两种模式。

------

## 用户输入场景

例如配置文件：

```json
{
    "log_format":"[{:%T}] {}"
}
```

读取后：

```cpp
std::string fmt = config.log_format;

std::format(
    std::runtime_format(fmt),
    now,
    message
);
```

以前只能：

```cpp
std::vformat(...)
```

现在直接：

```cpp
std::format(...)
```

即可。

------

## 本质实现

新增类型：

```cpp
std::runtime_format_string<>
```

以及工厂函数：

```cpp
std::runtime_format()
```

大致效果：

```cpp
auto rf =
    std::runtime_format(fmt);
```

得到：

```cpp
runtime_format_string<char>
```

然后：

```cpp
std::format(rf,args...)
```

走运行时解析路径。

------

## 错误检查

编译期：

```cpp
std::format("{:d}", "hello");
```

编译错误。

------

运行时：

```cpp
std::string fmt = "{:d}";

std::format(
    std::runtime_format(fmt),
    "hello"
);
```

编译通过。

运行时抛出：

```cpp
std::format_error
```

------

### 对比

| 特性                 | C++23 format | C++26 runtime_format     |
| -------------------- | ------------ | ------------------------ |
| 编译期检查           | 是           | 否                       |
| 格式串必须 constexpr | 是           | 否                       |
| 用户输入格式串       | 不支持       | 支持                     |
| 配置文件格式串       | 不支持       | 支持                     |
| 错误发现时机         | 编译期       | 运行期                   |
| 使用方式             | format()     | format(runtime_format()) |

------

### 实际价值

最受益的是：

- 日志系统
- 模板引擎
- 配置驱动输出
- 国际化（i18n）
- 数据库报表
- 用户自定义格式

例如：

```cpp
std::string fmt =
    get_template_from_db();

return std::format(
    std::runtime_format(fmt),
    value1,
    value2
);
```

以前必须：

```cpp
std::vformat(
    fmt,
    std::make_format_args(...)
);
```

现在可以继续使用熟悉的：

```cpp
std::format(...)
```

接口。

总结一下：C++20/23 的 `<format>` 强调“编译期安全”，代价是格式串必须是常量；C++26 新增 `std::runtime_format()`，允许开发者显式选择“运行时格式串”，从而让 `std::format` 可以直接处理配置文件、数据库、网络或用户输入产生的格式模板，而无需退回到 `std::vformat()`。