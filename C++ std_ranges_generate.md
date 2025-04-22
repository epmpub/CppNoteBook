

# std::ranges::generate 

```C++
#include <algorithm>
#include <array>
#include <iostream>
#include <random>
#include <string_view>

// 生成随机骰子点数（1 到 6）
auto dice() {
    // use fixed seed for reproducibility
    static std::uniform_int_distribution<int> distr{ 1, 6 };

    static std::random_device device;
    static std::mt19937 engine{ device() };

    return distr(engine);
}

// 生成递增序列
void iota(auto& r, int init) {
    std::ranges::generate(r, [init]() mutable { return init++; });
}

// 打印范围的工具函数
void print(std::string_view comment, const auto& v) {
    std::cout << comment;
    for (int i : v) std::cout << i << ' ';
    std::cout << '\n';
}

int main() {
    std::array<int, 2> v;

    // 使用基于迭代器的重载
    //std::ranges::generate(v.begin(), v.end(), dice);
    //print("骰子 (迭代器): ", v);

    //// 使用基于范围的重载
    //std::ranges::generate(v, dice);
    //print("骰子 (范围): ", v);

    //// 使用 iota 生成从 1 开始的序列
    iota(v, 1);
    print("递增序列: ", v);
}
```

C++20 中的 std::ranges::generate 算法属于 Ranges 库，用于将函数对象生成的连续值赋值给指定范围内的每个元素。它是传统 std::generate 算法的现代化、基于范围的版本，提供了更高的灵活性和与范围概念的组合性。以下是基于 C++ 标准和常见用法的详细解释，翻译成中文。

------

概述

std::ranges::generate 通过反复调用一个可调用对象（例如函数、lambda 表达式或函数对象）生成的返回值，依次赋值给范围中的每个元素。它有两种重载形式：

1. **基于迭代器的版本**：操作由迭代器和哨兵定义的范围。
2. **基于范围的版本**：操作整个范围对象（例如容器或视图）。

该算法定义在 <algorithm> 头文件中，是一个 *niebloid*（可定制的函数对象，支持范围操作）。

------

语法

cpp

```cpp
namespace std::ranges {
// (1) 基于迭代器的重载
template<std::input_or_output_iterator O, std::sentinel_for<O> S, std::copy_constructible F>
  requires std::invocable<F&> && std::indirectly_writable<O, std::invoke_result_t<F&>>
constexpr O generate(O first, S last, F gen);

// (2) 基于范围的重载
template<class R, std::copy_constructible F>
  requires std::invocable<F&> && ranges::output_range<R, std::invoke_result_t<F&>>
constexpr ranges::borrowed_iterator_t<R> generate(R&& r, F gen);
}
```

------

参数

- **first, last**：迭代器-哨兵对，定义范围 [first, last)。first 必须是输出迭代器，last 是标记范围结束的哨兵。
- **r**：范围对象（例如 std::vector 等容器或视图）。范围必须满足 output_range 约束，匹配 gen 返回的类型。
- **gen**：可调用对象（例如函数、lambda 或函数对象），不接受参数，返回可赋值给范围元素的值。gen 会被反复调用以生成值。

------

返回值

- 基于迭代器的重载：返回迭代器 first，它被移动到最后一个赋值元素之后的下一个位置（通常等于 last）。
- 基于范围的重载：返回 borrowed_iterator_t<R>，即范围 r 的结束迭代器。

------

约束

- 迭代器 O 必须满足 input_or_output_iterator，范围 [first, last) 必须可写入 gen 的返回值。
- 范围 R 必须满足 output_range，匹配 gen 返回的类型。
- F 必须是可复制构造的且可调用，其返回值必须可写入范围的元素。
- 精确执行 ranges::distance(first, last) 或 ranges::distance(r) 次 gen() 调用，之后进行赋值。

------

示例

以下是一个展示 std::ranges::generate 两种重载的示例代码：

cpp

```cpp
#include <algorithm>
#include <array>
#include <iostream>
#include <random>
#include <string_view>

// 生成随机骰子点数（1 到 6）
auto dice() {
    static std::uniform_int_distribution<int> distr{1, 6};
    static std::random_device device;
    static std::mt19937 engine{device()};
    return distr(engine);
}

// 生成递增序列
void iota(auto& r, int init) {
    std::ranges::generate(r, [init]() mutable { return init++; });
}

// 打印范围的工具函数
void print(std::string_view comment, const auto& v) {
    std::cout << comment;
    for (int i : v) std::cout << i << ' ';
    std::cout << '\n';
}

int main() {
    std::array<int, 8> v;

    // 使用基于迭代器的重载
    std::ranges::generate(v.begin(), v.end(), dice);
    print("骰子 (迭代器): ", v);

    // 使用基于范围的重载
    std::ranges::generate(v, dice);
    print("骰子 (范围): ", v);

    // 使用 iota 生成从 1 开始的序列
    iota(v, 1);
    print("递增序列: ", v);
}
```

**示例输出**（随机值可能不同）：

```text
骰子 (迭代器): 3 5 2 6 1 4 3 2
骰子 (范围): 4 1 6 3 2 5 4 1
递增序列: 1 2 3 4 5 6 7 8
```

------

主要特性

1. **惰性求值**：与传统 std::generate 不同，std::ranges::generate 与 Ranges 库集成，支持视图和惰性求值管道。
2. **类型安全**：使用 C++20 概念确保生成器的返回值类型与范围元素兼容。
3. **组合性**：与范围适配器（如 std::views::filter、std::views::transform）无缝协作，支持函数式编程风格。
4. **Niebloid**：std::ranges::generate 是一个定制点对象，支持依赖查找以避免不必要的非限定查找。

------

与 std::generate 的比较

- **传统 std::generate**：仅支持迭代器对，不支持范围操作，缺乏概念约束。
- **std::ranges::generate**：支持迭代器对和范围，使用概念提升类型安全性，与 Ranges 生态系统集成。

------

注意事项

- 如果 gen 抛出异常，行为取决于执行策略（如果使用）。对于标准策略，调用 std::terminate；否则，行为由实现定义。
- 如果内存分配失败（例如容器调整大小时），可能抛出 std::bad_alloc。
- 如果需要生成固定数量的元素，可考虑使用 std::ranges::generate_n，它接受数量而非范围结束。

------

参考资料

- C++20 标准，[alg.generate]
- cppreference.com: std::ranges::generate

