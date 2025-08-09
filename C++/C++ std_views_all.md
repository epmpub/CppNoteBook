# std *::views::all*

std *::views::all*可能看起来有点无意义，生成一个范围内所有元素的视图。

然而，*std::views::all*将根据参数的值类别以及它是否是借用范围产生不同的类型，确保我们不会产生悬垂视图。

还可以通过辅助*std::views::all_t*访问结果类型。



```C++
#include <vector>
#include <ranges>

int main() {
    std::vector<int> data{1,2,3,4,5};
    
    // constructed from lvalue, v1 will be a ref_view
    auto v1 = std::views::all(data);
    // decltype(v1) == std::ranges::ref_view<std::vector<int>>
    static_assert(std::is_same_v<decltype(v1), std::ranges::ref_view<std::vector<int>>>);

    // we can also obtain the type using std::views::all_t
    using t1 = std::views::all_t<std::vector<int>&>;
    // t1 == std::ranges::ref_view<std::vector<int>>
    static_assert(std::is_same_v<t1, std::ranges::ref_view<std::vector<int>>>);

    // constructed from rvalue, v2 will be an owning_view
    auto v2 = std::views::all(std::vector<int>{1,2,3,4,5});
    // decltype(v2) == std::ranges::owning_view<std::vector<int>>
    static_assert(std::is_same_v<decltype(v2), std::ranges::owning_view<std::vector<int>>>);

    using t2 = std::views::all_t<std::vector<int>&&>;
    // t2 == std::ranges::owning_view<std::vector<int>>
    static_assert(std::is_same_v<t2, std::ranges::owning_view<std::vector<int>>>);

    // Wouldn't compile, owning_view does not support copy, only move
    // auto v3 = v2;

    // constructed from borrowed range, v4 will be std::span
    auto v4 = std::views::all(std::span(data));
    // decltype(v4) == std::span<int>
    static_assert(std::is_same_v<decltype(v4), std::span<int>>);

    using t4 = std::views::all_t<std::span<int>>;
    // t4 == std::span<int>
    static_assert(std::is_same_v<t4, std::span<int>>);   
}
```

这段代码展示了 C++20 中 <ranges> 库提供的 std::views::all 视图适配器，用于从不同类型的范围（range）创建视图。代码通过示例展示了 std::views::all 根据输入的值类别（lvalue 或 rvalue）和类型生成不同视图（ref_view、owning_view 或 span）。以下是逐步解释。

------

代码概览

- 使用 std::views::all 处理左值、右值和借用范围。
- 通过 static_assert 验证生成的视图类型。
- 展示视图类型的特性和限制。

------

关键组件

1. **头文件**

cpp

```cpp
#include <vector>
#include <ranges>
```

- <vector>：提供 std::vector。
- <ranges>：提供 std::views::all、std::ranges::ref_view 和 std::ranges::owning_view。
- **main 函数**

**左值构造：ref_view**

cpp

```cpp
std::vector<int> data{1,2,3,4,5};
auto v1 = std::views::all(data);
static_assert(std::is_same_v<decltype(v1), std::ranges::ref_view<std::vector<int>>>);
using t1 = std::views::all_t<std::vector<int>&>;
static_assert(std::is_same_v<t1, std::ranges::ref_view<std::vector<int>>>);
```

- **std::views::all(data)**：
  - data 是左值（std::vector<int>&）。
  - 返回 std::ranges::ref_view，一个轻量视图，引用 data。
- **ref_view**：
  - 持有对 data 的引用，不拥有数据。
  - 生命周期依赖 data。
- **std::views::all_t**：
  - 类型别名，推导 std::views::all 的返回类型。
- **验证**：
  - v1 和 t1 均为 ref_view<std::vector<int>>。

**右值构造：owning_view**

cpp

```cpp
auto v2 = std::views::all(std::vector<int>{1,2,3,4,5});
static_assert(std::is_same_v<decltype(v2), std::ranges::owning_view<std::vector<int>>>);
using t2 = std::views::all_t<std::vector<int>&&>;
static_assert(std::is_same_v<t2, std::ranges::owning_view<std::vector<int>>>);
```

- **std::views::all(std::vector<int>{...})**：
  - 输入是右值（临时对象）。
  - 返回 std::ranges::owning_view，拥有临时对象的所有权。
- **owning_view**：
  - 接管临时对象的生命周期，延长其存在时间。
  - 不支持拷贝，只支持移动（避免资源重复释放）。
- **验证**：
  - v2 和 t2 均为 owning_view<std::vector<int>>。

**拷贝限制**

cpp

```cpp
// auto v3 = v2; // 不编译
```

- **owning_view**：
  - 禁用拷贝构造函数，确保单一所有权。
  - 可移动（如 auto v3 = std::move(v2)）。

**借用范围：span**

cpp

```cpp
auto v4 = std::views::all(std::span(data));
static_assert(std::is_same_v<decltype(v4), std::span<int>>);
using t4 = std::views::all_t<std::span<int>>;
static_assert(std::is_same_v<t4, std::span<int>>);
```

- **std::span(data)**：
  - 创建一个借用视图，引用 data 的连续内存。
- **std::views::all(std::span)**：
  - 输入已是借用范围，返回 std::span<int>。
- **span**：
  - 轻量、非拥有型视图，直接引用数据。
- **验证**：
  - v4 和 t4 均为 std::span<int>。

------

为什么这样工作？

1. **std::views::all**：
   - 根据输入类型动态选择视图：
     - 左值 → ref_view（引用）。
     - 右值 → owning_view（拥有）。
     - 借用范围（如 span）→ 保持原类型。
2. **ref_view**：
   - 避免拷贝，依赖外部对象。
3. **owning_view**：
   - 管理临时对象，延长生命周期。
4. **span**：
   - 表示连续内存的视图，无需额外包装。

------

输出

- 无运行时输出，代码验证编译期行为。

------

使用场景

- **ref_view**：
  - 处理现有容器，避免拷贝。
- **owning_view**：
  - 在管道中延长临时对象生命。
- **span**：
  - 操作连续内存块，如数组或向量。

------

总结

- std::views::all(data) 生成 ref_view，引用 data。
- std::views::all(temporary) 生成 owning_view，拥有临时对象。
- std::views::all(span) 返回 span，保持借用语义。
- 代码展示了 std::views::all 的多态性和类型推导能力。