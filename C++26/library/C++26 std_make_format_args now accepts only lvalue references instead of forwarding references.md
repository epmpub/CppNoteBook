## C++26 std::make_format_args now accepts only lvalue references instead of forwarding references

C++26 中的这个变化（DR20，对应提案 P2905R2） 是针对 std::make_format_args / std::make_wformat_args 的一个故意引入的 breaking change（同时作为 Defect Report 回溯应用到 C++20/23）。 

en.cppreference.com

背景与问题在 C++20 最初设计中，make_format_args 使用 forwarding references（Args&&...）：

cpp

```cpp
template<class Context = format_context, class... Args>
format-arg-store<Context, Args...>
    make_format_args(Args&&... fmt_args);  // 旧行为
```

这允许传递 临时对象（rvalue），但会导致严重的安全隐患（dangling references）。经典的 UB 示例（在 C++23 及之前可能编译通过）：

cpp

```cpp
std::string str = "{}";
std::filesystem::path path = "path/etic/experience";

auto args = std::make_format_args(path.string());  // 临时 string 对象！
std::string msg = std::vformat(str, args);         // UB：临时已被销毁
```

basic_format_arg（内部存储）对用户自定义类型持有引用（reference semantics），不会延长临时对象的生命周期。临时对象在 make_format_args 返回后立即销毁，导致后续 vformat 使用 dangling reference。 

open-std.org

C++26 的修复现在签名改为只接受 lvalue references（Args&...）：

cpp

```cpp
template<class Context = format_context, class... Args>
format-arg-store<Context, Args...>
    make_format_args(Args&... args);   // 新行为（C++26 / DR）

// 类似地 make_wformat_args 也改为 Args&...
```

- 拒绝 rvalue / 临时对象（编译错误）。
- 必须传入有名变量（lvalue），确保对象在 format_args 使用期间存活。
- 对正确用法（已经用 lvalue）几乎无影响。 en.cppreference.com

为什么这么改？

- 安全性优先：防止用户无意中创建 dangling reference，这是 std::format 整个设施的核心安全目标之一（与 compile-time format string check 一致）。
- {fmt} 库早已这么做了，并捕获了真实 bug。
- 用法统计显示 make_format_args 的直接使用量很低，大部分场景可以轻松修复（存到变量、或改用新的 std::dynamic_format / 之前的 runtime_format）。 sandordargo.com

实际影响与迁移

- 正确代码（传入变量）：无需修改。
- 错误代码（传入临时）：现在编译失败，必须改成：

cpp

```cpp
auto tmp = path.string();
auto args = std::make_format_args(tmp);  // OK
```

或直接用 std::format + dynamic_format（C++26 推荐的 runtime format string 方式）。

这个变化已由三大编译器实现（GCC 14+、Clang 18+、MSVC 19.40+），并作为 DR 回溯应用。 

sandordargo.com

一句话总结：这是 std::format 安全收紧的一部分——从“允许但危险的 forwarding”改为“只允许安全的 lvalue”，主动拒绝可能导致 dangling 的临时对象使用场景。属于“短期 breaking，长期正确”的典型 DR。 如果你有具体使用 make_format_args 的代码，可以贴出来帮你看怎么迁移。