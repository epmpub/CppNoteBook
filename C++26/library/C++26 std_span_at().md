C++26 `std::span::at()`  

```c
#include <span>
#include <array>
#include <print>

int main()
{
    std::array arr{10,20,30};

    std::span sp(arr);

    std::println("{}", sp.at(1));

    try {
        auto r = sp.at(3); // contract violation
        std::println("{}", r);
    } catch (const std::out_of_range& e) {
        std::println("Caught exception: {}", e.what());
    }

    return 0;
}
```

`std::span::at()` 是 C++26 对 `std::span` 的一个小而实用的增强，来源于 **P2821R5《span.at()》**。

它解决的问题非常简单：

> `std::span` 一直缺少带边界检查的元素访问接口，而标准容器几乎都有 `at()`。

例如：

```cpp
std::vector<int> v;
v.at(0);
std::array<int,10> a;
a.at(0);
std::string s;
s.at(0);
```

都有。

但 C++20/C++23：

```cpp
std::span<int> sp;
```

只有：

```cpp
operator[]
front()
back()
data()
```

没有：

```cpp
at()
```

导致接口不一致。

------

## C++23 的问题

```cpp
std::span<int> sp(v);

int x = sp[100];
```

如果越界：

```text
Undefined Behavior
```

标准不要求检查。

和：

```cpp
std::vector::operator[]
```

一样。

------

很多人自然会写：

```cpp
sp.at(100);
```

结果：

```text
error: no member named 'at'
```

因为根本不存在。

------

## C++26 新增

```cpp
constexpr reference at(size_type idx) const;
```

现在：

```cpp
std::span<int> sp(v);

sp.at(0);
```

合法。

------

## 示例

```cpp
#include <span>
#include <vector>
#include <print>

int main()
{
    std::vector<int> v {1,2,3};

    std::span sp(v);

    std::println("{}", sp.at(1));
}
```

输出：

```text
2
```

------

## 越界行为

```cpp
sp.at(100);
```

不会产生 UB。

而是违反：

```cpp
precondition
```

并触发标准规定的运行时检查机制。

在支持 Contracts 的实现中，通常表现为：

```text
contract violation
```

或程序终止。

------

## 与 vector::at 的区别

很多人会想到：

```cpp
std::vector::at()
```

抛出：

```cpp
std::out_of_range
```

那么：

```cpp
span::at()
```

也是这样吗？

不是。

------

因为 `std::span` 是 Freestanding 类型。

委员会不希望：

```cpp
span
```

依赖：

```cpp
<exception>
std::out_of_range
```

特别是：

```cpp
Embedded
Kernel
Freestanding
```

环境。

因此设计成：

```text
Contract-based check
```

而不是：

```text
throw std::out_of_range
```

------

## 实现效果

大多数实现会类似：

```cpp
constexpr reference
at(size_type idx) const
{
    if(idx >= size())
        contract_violation();

    return (*this)[idx];
}
```

而不是：

```cpp
throw std::out_of_range(...);
```

------

## constexpr 支持

由于是：

```cpp
constexpr
```

所以：

```cpp
constexpr int arr[] = {1,2,3};

constexpr std::span sp(arr);

static_assert(sp.at(1) == 2);
```

合法。

------

如果越界：

```cpp
static_assert(sp.at(100) == 0);
```

编译失败。

------

## 与 Contracts 的关系

P2821 的设计实际上是为未来 Contracts 做准备。

例如：

```cpp
sp.at(i);
```

语义类似：

```cpp
[[expects: i < sp.size()]]
```

然后：

```cpp
return sp[i];
```

因此：

```text
at()
=
带标准化前置条件检查的 operator[]
```

------

## 为什么重要

考虑：

```cpp
void process(std::span<int> data)
{
    auto x = data[0];
}
```

如果：

```cpp
data.empty()
```

则：

```text
UB
```

而：

```cpp
auto x = data.at(0);
```

至少能够检测错误。

------

## 与 gsl::span 的统一

微软 GSL 很早就有：

```cpp
gsl::span::at()
```

例如：

```cpp
gsl::at(span, i);
```

用于安全访问。

C++26 的标准 `span::at()` 在很大程度上吸收了这类实践经验。

------

## 一个典型例子

```cpp
#include <span>
#include <array>
#include <print>

int main()
{
    std::array arr{10,20,30};

    std::span sp(arr);

    std::println("{}", sp.at(1));

    sp.at(10); // contract violation
}
```

输出：

```text
20
```

随后运行时检测失败。

------

### 与 operator[] 对比

| 特性         | operator[] | at()       |
| ------------ | ---------- | ---------- |
| C++20        | 有         | 无         |
| C++26        | 有         | 有         |
| 越界检查     | 无         | 有         |
| UB           | 是         | 否         |
| constexpr    | 是         | 是         |
| Freestanding | 是         | 是         |
| 抛异常       | 否         | 否（通常） |

总结一下：`std::span::at()` 是 C++26 为 `span` 补上的“最后一块拼图”。它提供与标准容器一致的安全访问接口，但不像 `vector::at()` 那样依赖异常，而是采用契约（Contract）风格的边界检查，因此既适合 Hosted 环境，也适合 Embedded/Freestanding 环境。