# C++26 placeholder name _

```C++
#include <map>
#include <print>
#include <mutex>

int main() {
    std::map<int, int> data{{1,2},{2,1},{3,4},{4,3}}; 

    // Iterate over values, ignoring keys
    for (auto& [_, value] : data) {
        std::print("{} ", value);
    }
    std::println("");

    std::mutex mux;
    {
        // RAII-only objects do not need to be named
        auto _ = std::unique_lock{mux}; 
        /*
        critical section
        */
    } // lock released

    // Multiple _ named variables with dynamic storage duration,
    // non-static member variables, bindings and captures
    // can redeclare previous _ in the same scope
    auto _ = 42;
    auto _ = 7;

    struct S {
        int _;
        int _;
    } s{42, 7};

    // Unique (i.e. not re-declared) _ can still be referenced
    {
        auto _ = [](auto _) { return _; };
        int x = _(42);
        // x == 42;
        std::println("x == {}", x);
    }
}
```

这段代码展示了 C++ 中一些现代特性和惯用法，包括结构化绑定、占位符变量 _ 的使用、RAII（资源获取即初始化）以及 lambda 表达式。代码通过示例展示了这些特性的灵活性和限制。以下是逐步解释。

------

代码概览

- 使用结构化绑定遍历 std::map，忽略键。
- 使用 _ 作为未使用的变量名，展示其在不同上下文中的行为。
- 结合 std::mutex 和 std::unique_lock 展示 RAII。
- 测试 _ 的重声明和引用规则。

------

关键组件

1. **头文件**

cpp

```cpp
#include <map>
#include <print>
#include <mutex>
```

- <map>：提供 std::map。
- <print>：C++23 的 std::println 和 std::print 用于输出。
- <mutex>：提供 std::mutex 和 std::unique_lock。
- **结构化绑定与 _**

cpp

```cpp
std::map<int, int> data{{1,2},{2,1},{3,4},{4,3}};
for (auto& [_, value] : data) {
    std::print("{} ", value);
}
std::println("");
```

- **data**：

  - 包含键值对：{{1, 2}, {2, 1}, {3, 4}, {4, 3}}。

- **auto& [_, value]**：

  - 结构化绑定，解构 std::pair。
  - _ 表示忽略键，value 绑定到值。

- **输出**：

  - 遍历值：2 1 4 3。

  ```text
  2 1 4 3
  ```

- **RAII 与 _**

cpp

```cpp
std::mutex mux;
{
    auto _ = std::unique_lock{mux};
}
```

- **std::unique_lock**：
  - RAII 对象，锁定 mux，作用域结束时自动解锁。
- **auto _**：
  - 使用 _ 命名，表示无需访问该对象。
  - 仅用于生命周期管理。
- **作用**：
  - 展示 _ 在不需要引用变量时的用途。
- **多重声明 _**

cpp

```cpp
auto _ = 42;
auto _ = 7;
struct S {
    int _;
    int _;
} s{42, 7};
```

- **auto _**：
  - 在同一作用域内多次声明 _，每次覆盖前一个。
  - C++ 允许 _ 的重声明，前提是动态存储或非静态成员。
- **struct S**：
  - 定义两个成员 _，但这会导致编译错误。
  - **修正**：C++ 不允许同一结构体中重名成员，示例中应为笔误，应为不同名称。
  - 假设改为 int a; int b;，s{42, 7} 初始化为 {a=42, b=7}。
- **行为**：
  - _ 的重声明合法，但无法引用之前的 _。
- **唯一 _ 的引用**

cpp

```cpp
{
    auto _ = [](auto _) { return _; };
    int x = _(42);
    std::println("x == {}", x);
}
```

- **auto _ = lambda**：

  - 定义 lambda，捕获参数也命名为 _。
  - 外部 _ 是 lambda 对象，内部 _ 是参数。

- **_(42)**：

  - 调用 lambda，返回 42。

- **x**：

  - 赋值为 42。

- **输出**：

  ```text
  x == 42
  ```

- **规则**：

  - 唯一的 _（未被重声明）可被引用。

------

为什么这样工作？

1. **结构化绑定**：
   - _ 作为占位符，忽略不需要的部分。
2. **_ 的重声明**：
   - C++ 允许在动态存储（如局部变量）或捕获中重用 _，但不保留历史值。
   - 静态成员或全局变量中 _ 不可重声明。
3. **RAII**：
   - std::unique_lock 的生命周期由作用域控制，_ 表示无需命名访问。
4. **Lambda**：
   - _ 可在不同上下文中独立使用（变量名、参数名）。

------

输出

```text
2 1 4 3
x == 42
```

------

使用场景

- **忽略变量**：
  - 使用 _ 表示未使用的值或对象。
- **RAII**：
  - 临时锁定资源，无需命名。
- **简洁代码**：
  - 在 lambda 或绑定中简化语法。

------

总结

- 结构化绑定用 _ 忽略键，输出 2 1 4 3。
- _ 用于 RAII 和多重声明，展示其灵活性。
- 唯一 _ 可引用，lambda 示例输出 42。
- 代码展示了 _ 的惯用法和限制。