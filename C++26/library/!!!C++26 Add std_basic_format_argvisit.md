**Member `std::basic_format_arg::visit()`** 是 C++26 `<format>` 的一项库改进，来源于 WG21 提案 **P2637R3**。

它解决的问题其实很具体：

> 以前访问 `std::basic_format_arg` 内部保存的值，必须使用非成员函数 `std::visit_format_arg()`；C++26 改为提供成员函数 `.visit()`，并废弃旧接口。

------

## 1. 先了解 `std::basic_format_arg`

`std::basic_format_arg` 是 `<format>` 内部使用的类型擦除（type erasure）容器。

当你写：

```cpp
std::format("{} {}", 42, 3.14);
```

标准库实际上会把参数包装成：

```cpp
basic_format_arg
```

类似：

```cpp
42
↓
basic_format_arg

3.14
↓
basic_format_arg
```

然后交给格式化引擎处理。

------

简化理解：

```cpp
std::basic_format_arg
≈
std::variant<
    int,
    long,
    double,
    const char*,
    string_view,
    ...
>
```

当然真实实现不是 variant。

------

## 2. C++20 的访问方式

C++20 引入：

```cpp
std::visit_format_arg()
```

例如：

```cpp
#include <format>

struct Visitor
{
    template<typename T>
    void operator()(T const& value) const
    {
        std::cout << value << '\n';
    }
};
```

访问：

```cpp
auto arg = std::make_format_args(42);

std::visit_format_arg(
    Visitor{},
    std::get<0>(arg)
);
```

思想类似：

```cpp
std::visit()
```

访问 variant。

------

## 3. 存在的问题

问题在于：

```cpp
std::visit_format_arg(visitor, arg)
```

语法风格与现代 STL 不一致。

比较：

### variant

```cpp
std::visit(visitor, var);
```

------

### optional

```cpp
opt.transform(...);
```

------

### expected

```cpp
exp.transform(...);
```

------

### format_arg

```cpp
std::visit_format_arg(visitor, arg);
```

显得比较老式。

------

而且：

```cpp
arg
```

本身就知道自己的类型擦除内容。

因此更自然的接口应该是：

```cpp
arg.visit(visitor);
```

------

## 4. C++26 的改进

新增：

```cpp
template<class Visitor>
decltype(auto) visit(this basic_format_arg self,
                     Visitor&& vis);
```

因此可以写：

```cpp
arg.visit(visitor);
```

------

例如：

```cpp
auto result =
    arg.visit(
        [](auto&& value)
        {
            return sizeof(value);
        });
```

------

## 5. 示例

### C++20 写法

```cpp
std::visit_format_arg(
    [](auto&& x)
    {
        std::cout << x;
    },
    arg
);
```

------

### C++26 写法

```cpp
arg.visit(
    [](auto&& x)
    {
        std::cout << x;
    });
```

更符合现代 STL 风格。

------

## 6. 返回值支持

例如：

```cpp
auto size =
    arg.visit(
        [](auto&& value)
        {
            return sizeof(value);
        });
```

根据实际存储类型返回：

```cpp
sizeof(int)
```

或者：

```cpp
sizeof(double)
```

等。

------

## 7. 一个完整示例

```cpp
#include <format>
#include <iostream>

int main()
{
    auto store =
        std::make_format_args(
            42,
            3.14,
            "hello"
        );

    auto arg = store.get(0);

    arg.visit(
        [](auto const& value)
        {
            std::cout
                << value
                << '\n';
        });
}
```

输出：

```text
42
```

------

## 8. 与 `std::variant::visit` 的关系

思想非常接近：

| 类型                            | 访问方式                  |
| ------------------------------- | ------------------------- |
| `std::variant`                  | `std::visit()`            |
| `std::basic_format_arg` (C++20) | `std::visit_format_arg()` |
| `std::basic_format_arg` (C++26) | `arg.visit()`             |

------

可以理解成：

```cpp
std::visit_format_arg
```

终于被成员函数化了。

------

## 9. 旧接口状态

C++26 中：

```cpp
std::visit_format_arg
```

被标记为废弃（deprecated）。

即：

```cpp
[[deprecated]]
std::visit_format_arg(...)
```

未来可能移除。

推荐迁移：

```cpp
arg.visit(...)
```

------

## 10. 为什么标准委员会要改

主要有三个原因：

### 原因1：一致性

现代 STL 越来越偏向：

```cpp
obj.operation(...)
```

而不是：

```cpp
operation(obj,...)
```

------

### 原因2：可发现性

看到：

```cpp
basic_format_arg
```

自然会找：

```cpp
.visit()
```

而不是去翻文档找：

```cpp
visit_format_arg()
```

------

### 原因3：简化实现

许多实现内部已经把：

```cpp
visit_format_arg
```

转发到对象内部访问器。

标准化成员函数更直接。

------

## 11. 实际影响

对于普通用户：

```cpp
std::format(...)
```

完全没有变化。

因为：

```cpp
basic_format_arg
```

属于格式化框架底层设施。

主要受益者是：

- 自定义 formatter 作者
- 格式化库开发者
- logging 框架开发者
- fmtlib 风格库维护者

例如：

- [fmtlib](https://fmt.dev/?utm_source=chatgpt.com)
- [spdlog](https://github.com/gabime/spdlog?utm_source=chatgpt.com)

这类库经常直接操作 `basic_format_arg`。

------

## 总结

C++26 的 **Member `std::basic_format_arg::visit()`** 做了一件简单但重要的事情：

```cpp
// C++20
std::visit_format_arg(visitor, arg);

// C++26
arg.visit(visitor);
```

同时：

```cpp
std::visit_format_arg()
```

被废弃。

它没有改变 `<format>` 的能力，而是让 `basic_format_arg` 的访问方式：

- 更符合现代 STL 设计风格；
- 与 `variant`、`optional`、`expected` 等类型保持一致；
- 提高可读性和可发现性；
- 简化格式化框架实现。