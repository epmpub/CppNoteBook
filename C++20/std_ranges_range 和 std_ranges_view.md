# std::ranges::range 和 std::ranges::view

std::ranges::range 和 std::ranges::view 是 C++20 Ranges 库中的两个核心概念，它们都用于描述可迭代的序列，但在用途、语义和约束上有着显著的区别。以下是对两者的详细解释及其对比。

------

1. std::ranges::range

定义

std::ranges::range 是一个概念（concept），用于约束一个类型是否是一个“范围”（range）。一个类型 T 满足 std::ranges::range 概念，如果它可以通过 std::ranges::begin(T) 和 std::ranges::end(T) 获取迭代器和哨兵（sentinel），从而支持迭代。

要求

- **迭代器访问**：必须支持 std::ranges::begin(T) 返回一个迭代器，std::ranges::end(T) 返回一个迭代器或哨兵，定义范围的边界。
- **语义**：range 表示一个可迭代的元素序列，但不对序列的所有权、拷贝行为或修改行为做额外约束。

示例

满足 std::ranges::range 的类型包括：

- 标准容器：std::vector, std::list, std::array, std::string
- 临时范围：std::span, std::ranges::subrange
- Ranges 视图：std::ranges::iota_view, std::ranges::filter_view
- 自定义类型：只要实现了 begin() 和 end() 方法的类。

代码示例

cpp

```cpp
#include <vector>
#include <ranges>

static_assert(std::ranges::range<std::vector<int>>); // 标准容器是 range
```

特点

- **通用性**：range 是一个广义的概念，涵盖所有可迭代的序列，无论它们是否拥有数据。
- **不关心所有权**：range 可以是拥有数据的容器（如 std::vector）或不拥有数据的视图（如 std::ranges::iota_view）。
- **不要求高效性**：不要求 size() 或其他操作是 O(1)。

------

2. std::ranges::view

定义

std::ranges::view 是一个更严格的概念，约束一个类型是否是一个“视图”（view）。视图是一种特殊的 range，它是对某个序列的轻量、非拥有性（non-owning）引用，通常用于延迟计算或高效操作。

要求

一个类型 T 满足 std::ranges::view 概念，需要满足以下条件：

1. **是 range**：T 必须满足 std::ranges::range。
2. **高效移动和拷贝**：
   - 移动构造和移动赋值是 O(1) 且不抛出异常（noexcept）。
   - 拷贝构造和拷贝赋值（如果支持）是 O(1)。
3. **语义**：
   - 视图不拥有底层数据，只是提供一种访问或变换数据的方式。
   - 视图通常是轻量的，适合在 Ranges 管道中组合使用（如通过 | 操作符）。
4. **继承 std::ranges::view_base**（可选）：标准库的视图类型通常继承自 std::ranges::view_base 来明确标识自己是视图。

示例

满足 std::ranges::view 的类型包括：

- Ranges 视图：std::ranges::iota_view, std::ranges::filter_view, std::ranges::transform_view, std::ranges::take_view
- 某些非拥有范围：std::string_view, std::span
- 自定义视图：实现了 view 语义的类型。

不满足 std::ranges::view 的类型：

- 标准容器：std::vector, std::list（它们拥有数据，拷贝可能是 O(n)）。
- 昂贵的自定义范围：如果拷贝/移动不是 O(1)。

代码示例

cpp

```cpp
#include <ranges>

static_assert(std::ranges::view<std::ranges::iota_view<int, int>>); // iota_view 是 view
static_assert(!std::ranges::view<std::vector<int>>); // vector 不是 view
```

特点

- **非拥有性**：视图不拥有数据，只是引用或变换现有数据。例如，std::ranges::filter_view 只是过滤底层范围的元素。
- **高效性**：视图的构造、拷贝和移动是轻量的，通常是 O(1)。
- **延迟计算**：视图通常是惰性求值的，只有在迭代时才会计算结果。
- **管道操作**：视图设计用于 Ranges 的管道语法，如 range | std::views::filter(...) | std::views::transform(...)。

------

关键区别

| 特性          | std::ranges::range                             | std::ranges::view                                            |
| ------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| 定义          | 表示任何可迭代的序列                           | 表示轻量、非拥有的范围，适合延迟计算                         |
| 约束          | 只需要支持 begin() 和 end()                    | 必须是 range，且拷贝/移动高效（O(1)）                        |
| 数据所有权    | 可以拥有或不拥有数据                           | 不拥有数据，仅引用或变换                                     |
| 拷贝/移动开销 | 无限制（可能昂贵，如 O(n)）                    | 必须是 O(1) 且 noexcept                                      |
| 典型用途      | 通用范围，适用于任何可迭代对象                 | 高效的 Ranges 管道操作，延迟计算                             |
| 示例          | std::vector, std::list, std::ranges::iota_view | std::ranges::iota_view, std::ranges::filter_view, std::string_view |
| 管道操作      | 不一定适合管道                                 | 专为管道操作设计（如 `                                       |

------

你的代码上下文

在你的原始代码中：

cpp

```cpp
template<typename Coll, std::ranges::input_range T>
```

你使用了 std::ranges::input_range 来约束模板参数 T 是一个输入范围。std::ranges::input_range 是 std::ranges::range 的子概念，要求范围的迭代器至少支持单次遍历（std::input_iterator）。

- **与 range 的关系**：std::ranges::input_range 要求 T 是一个 std::ranges::range，因此你的代码适用于任何可迭代的序列（包括视图和容器）。
- **与 view 的关系**：如果 T 是一个 std::ranges::view（如 std::ranges::iota_view），它也会满足 std::ranges::input_range，但视图的轻量和非拥有性语义可能影响你的 add 函数的行为。例如，视图可能是临时的，迭代它可能需要注意生命周期。

如果你明确只需要视图，可以将约束改为 std::ranges::view：

cpp

```cpp
template<typename Coll, std::ranges::view T>
```

这会限制 T 只能是视图类型（如 std::ranges::filter_view），而排除拥有数据的容器（如 std::vector）。

------

示例：区分 range 和 view

cpp

```cpp
#include <vector>
#include <ranges>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    auto iota = std::ranges::iota_view(1, 6);

    // vec 是 range，但不是 view
    static_assert(std::ranges::range<decltype(vec)>);
    static_assert(!std::ranges::view<decltype(vec)>);

    // iota 是 range，也是 view
    static_assert(std::ranges::range<decltype(iota)>);
    static_assert(std::ranges::view<decltype(iota)>);

    // 使用 view 进行管道操作
    auto filtered = vec | std::views::filter([](int x) { return x % 2 == 0; });
    static_assert(std::ranges::view<decltype(filtered)>);

    for (int x : filtered) {
        std::cout << x << " "; // 输出: 2 4
    }
}
```

------

总结

- **std::ranges::range** 是一个广义概念，表示任何可迭代的序列，适用于容器、视图或自定义范围。
- **std::ranges::view** 是一个更严格的概念，表示轻量、非拥有的范围，专为高效、惰性计算和管道操作设计。
- 在你的代码中，std::ranges::input_range 已经足够通用，涵盖了 view 和非 view 的范围。如果需要限制为视图，可以使用 std::ranges::view 约束。