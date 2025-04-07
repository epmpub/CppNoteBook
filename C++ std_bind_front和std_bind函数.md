#  std::bind_front 和 std::bind 函数

这段代码展示了 C++ 中 <functional> 头文件提供的 std::bind_front 和 std::bind 函数的用法，以及它们在函数绑定和参数处理上的区别。代码还涉及可调用对象（Callable）、lambda 表达式和占位符（placeholders）的使用。

```C++
#include <functional>
#include <iostream>

struct Callable {
    void operator()(auto&&...) && {}
};

int main() {
    auto plus = [](int left, int right) { return left + right; };
    auto add10 = std::bind_front(plus, 10);

    auto r = add10(4);
    // r == 14

    std::cout << "r == " << r << "\n";

    // std::bind_front(f, bound_args...)(call_args...) is always equivalent to
    // std::invoke(f, bound_args..., call_args...)
    std::bind_front(Callable{}, 10, 20)();

    // std::bind(Callable{}, 10, 20)(); // Wouldn't compile

    auto var = [](auto&&...) {};
    // std::bind_front doesn't fix number of arguments
    auto bound = std::bind_front(var, 10, 20);
    bound(10); // OK
    bound(10, 20, 30); // OK

    using namespace std::placeholders;
    auto old_bound = std::bind(var, 10, 20, _1, _2, _3);
    old_bound(10, 20, 30); // OK
    // old_bound(10); // Wouldn't compile
}
```



------

代码概览

- 定义了一个通用的可调用对象 Callable。
- 使用 std::bind_front 和 std::bind 绑定函数参数，展示了它们的特性。
- 比较了两者在参数数量固定性上的不同行为。

------

关键组件

1. **头文件**

cpp

```cpp
#include <functional>
```

- <functional>：提供 std::bind_front、std::bind、std::invoke 和占位符（如 _1、_2）。
- **Callable 结构体**

cpp

```cpp
struct Callable {
    void operator()(auto&&...) && {}
};
```

- **定义**：
  - 一个可调用对象，重载了 operator()。
  - auto&&... 表示接受任意数量的参数（通用引用）。
  - && 表示仅右值实例可调用（移动语义）。
- **用途**：
  - 模拟一个通用的函数对象，用于测试绑定。
- **std::bind_front 简单示例**

cpp

```cpp
auto plus = [](int left, int right) { return left + right; };
auto add10 = std::bind_front(plus, 10);
auto r = add10(4);
// r == 14
```

- **plus**：
  - 一个 lambda，接受两个 int 参数，返回它们的和。
- **std::bind_front(plus, 10)**：
  - 将 plus 的第一个参数绑定为 10，返回一个新函数对象。
  - 新函数接受剩余参数（这里是一个 int）。
- **调用**：
  - add10(4) 等价于 plus(10, 4)。
  - r = 10 + 4 = 14。
- **std::bind_front 等价性**

cpp

```cpp
// std::bind_front(f, bound_args...)(call_args...)
// is always equivalent to
// std::invoke(f, bound_args..., call_args...)
std::bind_front(Callable{}, 10, 20)();
```

- **等价性**：
  - std::bind_front(f, bound_args...) 创建一个函数对象，其调用等价于 std::invoke(f, bound_args..., call_args...)。
- **示例**：
  - std::bind_front(Callable{}, 10, 20) 绑定 Callable 的前两个参数为 10 和 20。
  - 调用 () 等价于 std::invoke(Callable{}, 10, 20)。
  - Callable 的 operator() 接受任意参数，这里为空调用，执行空操作。
- **std::bind 的对比**

cpp

```cpp
// std::bind(Callable{}, 10, 20)(); // Wouldn't compile
```

- **std::bind**：
  - 与 std::bind_front 不同，std::bind 固定所有参数，除非使用占位符（如 _1）。
- **问题**：
  - std::bind(Callable{}, 10, 20) 绑定了两个参数，但未指定占位符。
  - 调用 () 时，参数数量不匹配（预期 0 个，但 Callable 可接受更多），导致编译错误。
- **注释**：
  - 表明 std::bind 不如 std::bind_front 灵活。
- **std::bind_front 的灵活性**

cpp

```cpp
auto var = [](auto&&...) {};
auto bound = std::bind_front(var, 10, 20);
bound(10); // OK
bound(10, 20, 30); // OK
```

- **var**：
  - 一个 lambda，接受任意数量的参数（auto&&...）。
- **std::bind_front(var, 10, 20)**：
  - 绑定前两个参数为 10 和 20，但不固定剩余参数数量。
- **调用**：
  - bound(10) → var(10, 20, 10)。
  - bound(10, 20, 30) → var(10, 20, 10, 20, 30)。
- **特性**：
  - std::bind_front 不限制未绑定参数的数量，适合变参函数。
- **std::bind 与占位符**

cpp

```cpp
using namespace std::placeholders;
auto old_bound = std::bind(var, 10, 20, _1, _2, _3);
old_bound(10, 20, 30); // OK
// old_bound(10); // Wouldn't compile
```

- **std::bind(var, 10, 20, _1, _2, _3)**：
  - 绑定前两个参数为 10 和 20。
  - 使用占位符 _1、_2、_3 表示后续三个参数。
- **调用**：
  - old_bound(10, 20, 30) → var(10, 20, 10, 20, 30)。
  - 参数数量固定为 3 个，少于 3 个（如 old_bound(10)）会编译失败。
- **特性**：
  - std::bind 要求调用时的参数数量与占位符匹配。

------

为什么这样工作？

1. **std::bind_front**：
   - 只绑定前置参数，剩余参数由调用者提供。
   - 等价于 std::invoke，保持函数的原始灵活性。
2. **std::bind**：
   - 固定所有参数（包括占位符指定的），调用时必须匹配。
   - 更严格，适合需要精确控制参数的情况。
3. **类型擦除**：
   - std::bind_front 和 std::bind 返回函数对象，隐藏原始函数类型。

------

输出

- r == 14 是唯一计算结果，但代码未输出。
- 其他部分是编译期行为展示。

------

使用场景

- **std::bind_front**：
  - 简化前置参数绑定，适合变参函数或回调。
- **std::bind**：
  - 需要精确控制参数位置和数量（如事件处理）。
- **回调设计**：
  - 结合可调用对象（如 Callable）实现灵活的函数绑定。

------

总结

- std::bind_front 绑定前置参数，保持剩余参数灵活性。
- std::bind 固定所有参数，需要占位符支持动态调用。
- 代码对比了两者的行为，展示了 C++ 函数绑定的强大功能。