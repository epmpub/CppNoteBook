

## std::ignore

 的形式化规范（Formal Specification） 是 C++26 中的一个缺陷报告修复（DR），提案为 P2968R2（Make std::ignore a first-class object），解决了 LWG 2933 问题。背景在 C++26 之前，std::ignore 的类型被描述为“unspecified”（未指定），仅在 std::tie 的上下文中正式定义。

虽然所有主流实现都允许 std::ignore = some_expression; 来丢弃返回值（尤其是 [[nodiscard]] 函数），但标准并未正式认可这种用法，导致严格符合标准的代码可能不被允许。C++26 正式将其提升为 first-class object，明确规定其行为。C++26 中的正式定义

cpp

```cpp
// 在 <tuple>（以及 <utility>）中
constexpr /*ignore-type*/ ignore;   // inline since C++17
```

其中 /*ignore-type*/ 是**暴露仅限（exposition-only）**的类型，规范大致如下（基于提案和 cppreference）：

cpp

```cpp
struct /*ignore-type*/ {
    template<class T>
    constexpr const /*ignore-type*/& operator=(const T&) const noexcept {
        return *this;
    }
};
```

关键特性（C++26 正式保证）：

- 支持任意类型（非 void）赋值到 std::ignore。
- 赋值操作是 constexpr 的。
- 赋值是 no-op（无操作），不产生任何副作用。
- 操作符返回对 ignore 自身的 const 引用（支持链式赋值，如 std::ignore = a = b;）。
- noexcept。
- 不能赋值 void 表达式或 volatile bit-field（这是有意限制）。

用法示例

cpp

```cpp
#include <tuple>
#include <utility>   // C++26 也可在 <utility> 中使用

[[nodiscard]] int compute() { return 42; }

int main() {
    // 1. 丢弃 [[nodiscard]] 函数的返回值（推荐写法）
    std::ignore = compute();

    // 2. 与 std::tie 配合（传统用法）
    std::set<std::string> s;
    bool inserted;
    std::tie(std::ignore, inserted) = s.insert("hello");

    // 3. 其他创意用法
    std::ignore = std::printf("hello\n");  // 合法且有明确定义
}
```

与之前版本的对比

| 方面               | C++23 及以前               | C++26 (P2968R2)             |
| ------------------ | -------------------------- | --------------------------- |
| 类型描述           | “unspecified”              | 正式的 exposition-only 类型 |
| std::ignore = expr | 实际可用，但未被标准认可   | 正式保证的行为              |
| constexpr          | 部分支持                   | 明确要求                    |
| 推荐用于丢弃返回值 | 社区实践 / Core Guidelines | 标准背书                    |
| 头文件             | 仅 <tuple>                 | 也可在 <utility> 中可用     |

设计动机

- 提供比 (void)expr; 或 static_cast<void>(expr); 更清晰、意图自文档化（self-documenting）的写法。
- 符合 C++ Core Guidelines（ES.48：避免强制转换）。
- 解决安全关键系统编程中丢弃返回值的教条问题。
- 使 std::ignore 成为真正的“first-class”工具，而非仅限 std::tie 的占位符。

总结：C++26 通过 P2968R2 给 std::ignore 提供了精确、形式化的类型规范，使其从“实现细节”变成了标准明确支持的特性。现在你可以放心地在任何地方使用 std::ignore = ...; 来表达“故意忽略这个值”的意图。 更多细节可参考：

- cppreference: [`std::ignore`](https://en.cppreference.com/w/cpp/utility/tuple/ignore)
- 提案: [P2968R2](https://wg21.link/P2968R2)

```c++

#include <tuple>
#include <utility>   // C++26 也可在 <utility> 中使用
#include <set>
#include <print>
#include <iostream>

[[nodiscard]] int compute() { return 42; }

int main() {
    // 1. 丢弃 [[nodiscard]] 函数的返回值（推荐写法）
    std::ignore = compute();

    // 2. 与 std::tie 配合（传统用法）
    std::set<std::string> s;

    bool inserted;
    
    std::tie(std::ignore,inserted) = s.insert("hello world c++");

    std::cout << inserted << "\n";

    for(const auto&i : s)
        std::cout << i << "\n";
}
```

