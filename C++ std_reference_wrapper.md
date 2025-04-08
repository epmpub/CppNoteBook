# std::reference_wrapper

```C++
#include <vector>
#include <functional>
#include <algorithm>
#include <random>

void fun(const std::vector<int>&, int) {}

int main() {
    std::vector<int> data{1,2,3,4,5};

    // The result of bind_front has value semantics, 
    // meaning the results are stored by copy.
    auto fn1 = std::bind_front(fun, data);
    fn1(42);

    // std::ref returns a std::reference_wrapper, avoiding the copy.
    // However, now fn2 cannot outlive data.
    auto fn2 = std::bind_front(fun, std::ref(data));
    fn2(42);

    int x{};
    // std::make_pair and std::make_tuple have special handling
    // for std::reference_wrapper
    auto t = std::make_tuple(x, std::ref(x), std::cref(x));
    // decltype(t) == std::tuple<int, int&, const int&>
    static_assert(std::is_same_v<decltype(t), std::tuple<int,int&,const int&>>);

    std::vector<int> rand;
    std::mt19937 rng;
    // We could wrap rng in a lambda, but we can also use
    // a std::reference_wrapper, which also provides operator()
    std::generate_n(std::back_inserter(rand), 10, std::ref(rng));

    // Wouldn't compile:
    // std::reference_wrapper<int> wrap1(42); // Cannot wrap temporaries
    // std::reference_wrapper<int> wrap2; // Cannot be "null"
}
```



这段代码展示了 C++ 中 <functional> 提供的工具（如 std::bind_front、std::ref 和 std::cref）以及 <utility> 中 std::make_tuple 的用法，用于处理函数绑定和引用包装。代码通过示例展示了这些工具的行为和限制。以下是逐步解释。

------

代码概览

- 使用 std::bind_front 绑定函数参数，比较值语义和引用语义。
- 使用 std::ref 和 std::cref 创建引用包装器，应用于元组和随机数生成。
- 测试引用包装器的限制。

------

关键组件

1. **头文件**

cpp

```cpp
#include <vector>
#include <functional>
#include <algorithm>
#include <random>
```

- <vector>：提供 std::vector。
- <functional>：提供 std::bind_front、std::ref 和 std::cref。
- <algorithm>：提供 std::generate_n。
- <random>：提供 std::mt19937。
- **fun 函数**

cpp

```cpp
void fun(const std::vector<int>&, int) {}
```

- 接受一个常量向量引用和整数，无具体实现。
- **main 函数**

**绑定函数（值语义）**

cpp

```cpp
std::vector<int> data{1,2,3,4,5};
auto fn1 = std::bind_front(fun, data);
fn1(42);
```

- **std::bind_front**：
  - C++20 引入，将 fun 的第一个参数绑定为 data。
  - 返回一个可调用对象，存储参数副本（值语义）。
- **fn1**：
  - 类型为绑定后的函数对象，持有 data 的拷贝。
  - 调用 fn1(42) 等价于 fun(data_copy, 42)。
- **特点**：
  - data 的副本独立于原对象，生命周期安全。

**绑定函数（引用语义）**

cpp

```cpp
auto fn2 = std::bind_front(fun, std::ref(data));
fn2(42);
```

- **std::ref**：
  - 返回 std::reference_wrapper<const std::vector<int>>，包装 data 的引用。
- **fn2**：
  - 绑定时存储引用而非副本。
  - 调用 fn2(42) 等价于 fun(data, 42)。
- **注意**：
  - fn2 的生命周期不能超过 data，否则导致悬垂引用。

**元组中的引用**

cpp

```cpp
int x{};
auto t = std::make_tuple(x, std::ref(x), std::cref(x));
static_assert(std::is_same_v<decltype(t), std::tuple<int,int&,const int&>>);
```

- **std::make_tuple**：
  - 创建元组，特殊处理 std::reference_wrapper：
    - 普通值（如 x）按值存储。
    - std::ref(x) 解包为 int&。
    - std::cref(x) 解包为 const int&。
- **t**：
  - 类型为 std::tuple<int, int&, const int&>。
- **验证**：
  - static_assert 确认类型正确。

**随机数生成**

cpp

```cpp
std::vector<int> rand;
std::mt19937 rng;
std::generate_n(std::back_inserter(rand), 10, std::ref(rng));
```

- **std::mt19937**：
  - 随机数生成器，提供 operator()。
- **std::ref(rng)**：
  - 包装 rng 为 std::reference_wrapper，支持 operator()。
- **std::generate_n**：
  - 生成 10 个随机数，填充 rand。
- **作用**：
  - 避免拷贝 rng，直接使用引用。

**引用包装器限制**

cpp

```cpp
// std::reference_wrapper<int> wrap1(42); // 非法
// std::reference_wrapper<int> wrap2;     // 非法
```

- **限制 1**：
  - std::reference_wrapper 不能包装临时对象（如 42），必须绑定到已有变量。
- **限制 2**：
  - 无默认构造函数，不能创建“空”包装器。
- **原因**：
  - 设计为轻量引用代理，不支持 null 或临时状态。

------

为什么这样工作？

1. **std::bind_front**：
   - 默认值语义，复制参数；结合 std::ref 可改为引用。
2. **std::ref 和 std::cref**：
   - 创建引用包装器，解包时保留引用语义。
3. **std::make_tuple**：
   - 智能处理包装器，生成正确类型的元组。
4. **std::reference_wrapper**：
   - 提供 operator()，适配需要调用的对象。

------

输出

- 无显式输出，代码验证编译期行为。
- rand 包含 10 个随机整数（值依赖 rng 状态）。

------

使用场景

- **函数绑定**：
  - std::bind_front 简化参数传递。
- **避免拷贝**：
  - std::ref 用于大对象或需修改的引用。
- **元组操作**：
  - 混合值和引用类型。
- **随机数**：
  - 高效复用生成器。

------

总结

- fn1 复制 data，fn2 引用 data。
- std::make_tuple 处理引用包装器，生成混合元组。
- std::ref(rng) 用于随机数生成。
- 代码展示了引用管理工具的灵活性和限制。