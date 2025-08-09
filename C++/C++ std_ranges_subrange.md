# std::ranges::subrange

**std::ranges::subrange** 是 C++20 中 <ranges> 库引入的一个工具，用于表示一对迭代器（通常是范围的起点和终点），并将其封装为一个范围（range）。它本质上是对传统迭代器对的现代化包装，旨在与 Ranges 库的其他功能（如视图、算法）无缝协作。std::ranges::subrange 特别适合在需要传递子范围或从现有范围中提取部分数据时使用。

以下是对 std::ranges::subrange 的详细解释，包括其定义、用法、特性和代码示例。

------

**基础知识**

- **定义**: std::ranges::subrange 是一个模板类，封装了一个迭代器对（begin 和 end），并可选地存储范围的大小。

- **头文件**: <ranges>。

- **模板参数**:

  cpp

  ```cpp
  template <std::input_or_output_iterator I, std::sentinel_for<I> S = I, std::ranges::subrange_kind K = std::ranges::subrange_kind::unsized>
  class subrange;
  ```

  - I: 迭代器类型，表示范围的起点。
  - S: 哨兵类型，表示范围的终点（默认与 I 相同）。
  - K: subrange_kind，决定是否存储大小（sized 或 unsized）。

- **用途**:

  - 从现有范围中提取子范围。
  - 作为 Ranges 算法的输入或输出。
  - 与视图（views）结合使用。

------

**关键特性**

1. **范围接口**:
   - 实现 begin() 和 end()，使其符合 Ranges 库的 range 概念。
   - 可以直接用于 Ranges 算法（如 std::ranges::for_each）。
2. **大小信息**:
   - subrange_kind::sized: 如果迭代器支持大小计算（如随机访问迭代器），可以存储范围的大小。
   - subrange_kind::unsized: 不存储大小，适用于不需要或无法计算大小的范围。
3. **灵活性**:
   - 支持任何迭代器和哨兵组合，只要满足 std::sentinel_for（即 S 可以作为 I 的终点）。
   - 可以从容器、数组或视图构造。
4. **类型推导**:
   - 提供辅助函数 std::ranges::subrange，通过构造函数推导模板参数。

------

**基本用法**

**构造**

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 从迭代器构造 subrange
    auto sub = std::ranges::subrange(vec.begin() + 1, vec.begin() + 4);
    for (int x : sub) {
        std::cout << x << " "; // 输出: 2 3 4
    }
}
```

- **subrange(vec.begin() + 1, vec.begin() + 4)**: 表示从第 2 个元素（索引 1）到第 4 个元素（索引 3）的子范围。
- **迭代**: sub 是一个范围，可以用范围 for 循环遍历。

**与 Ranges 算法结合**

cpp

```cpp
#include <ranges>
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    auto sub = std::ranges::subrange(vec.begin() + 1, vec.begin() + 4);
    std::ranges::for_each(sub, [](int x) {
        std::cout << x << " "; // 输出: 2 3 4
    });
}
```

- **std::ranges::for_each**: 直接接受 subrange 作为范围。

------

**带大小的 subrange**

如果需要显式指定大小，可以使用 subrange_kind::sized：

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    auto sub = std::ranges::subrange<std::vector<int>::iterator, std::vector<int>::iterator, std::ranges::subrange_kind::sized>(
        vec.begin() + 1, vec.begin() + 4, 3); // 显式指定大小为 3
    std::cout << "Size: " << sub.size() << "\n"; // 输出: Size: 3
    for (int x : sub) {
        std::cout << x << " "; // 输出: 2 3 4
    }
}
```

- **size()**: 对于 sized 的 subrange，可以通过 size() 获取范围长度。

------

**与视图结合**

subrange 常用于从视图中提取部分范围：

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    auto view = vec | std::views::drop(1) | std::views::take(3); // 丢弃第一个元素，取 3 个
    auto sub = std::ranges::subrange(view);
    for (int x : sub) {
        std::cout << x << " "; // 输出: 2 3 4
    }
}
```

- **std::views::drop 和 std::views::take**: 创建视图，subrange 将其转换为可迭代的范围。

------

**与 std::ranges::mismatch 结合**

结合你之前的问题，subrange 可以作为 std::ranges::mismatch 的输入：

cpp

```cpp
#include <ranges>
#include <string>
#include <vector>
#include <iostream>

int main() {
    std::string text1 = "Welcome to the underworld!";
    std::string text2 = "Welcome to the overworld!";
    auto sub1 = std::ranges::subrange(text1.begin(), text1.end());
    auto sub2 = std::ranges::subrange(text2.begin(), text2.end());
    auto result = std::ranges::mismatch(sub1, sub2);
    std::cout << *result.in1 << " " << *result.in2 << "\n"; // 输出: u o
}
```

- **subrange**: 将字符串的迭代器对封装为范围，传递给 std::ranges::mismatch。

------

**关键方法**

- **begin()**: 返回起点迭代器。
- **end()**: 返回终点迭代器或哨兵。
- **size()**: 如果是 sized，返回范围大小。
- **empty()**: 检查范围是否为空。
- **advance(n)**: 将起点向前移动 n 个位置。

------

**中文解释**

**功能**

- **std::ranges::subrange**: 将一对迭代器封装为范围，表示子范围。
- **用途**: 用于提取部分数据，或在 Ranges 算法和视图中使用。

**构造**

- 从迭代器对（如 begin() + 1, end()）创建。
- 可选指定大小（sized）或不指定（unsized）。

**特性**

- **范围支持**: 提供 begin() 和 end()，兼容 Ranges 库。
- **灵活性**: 支持任何迭代器类型（如随机访问、双向等）。
- **视图集成**: 可从视图构造或转为视图。

**示例**

1. **基本使用**: 从向量提取 {2, 3, 4}。
2. **带大小**: 显式指定子范围长度。
3. **与算法**: 与 std::ranges::for_each 或 mismatch 结合。

------

**注意事项**

1. **性能**:
   - subrange 本身是轻量级包装，不复制数据，仅存储迭代器。
   - sized 版本会额外存储大小，增加少量开销。
2. **生命周期**:
   - subrange 中的迭代器必须有效，避免悬垂引用。
3. **与传统迭代器的区别**:
   - 比手动管理 begin 和 end 更安全、更现代化。

------

**总结**

std::ranges::subrange 是 C++20 Ranges 库中的一个实用工具，用于表示子范围。它简化了迭代器对的管理，与 Ranges 生态无缝集成，非常适合现代 C++ 编程。如果你有具体问题或想深入某个用法，请告诉我！

## std::ranges::sized_range的用法

------

**std::ranges::sized_range** 是 C++20 中 <ranges> 库引入的一个**概念（concept）**，用于约束一个范围（range）必须支持大小查询操作。也就是说，任何满足 std::ranges::sized_range 概念的类型必须能够通过某种方式提供其元素数量（大小），通常是通过 size() 成员函数或等效的机制。这个概念在需要知道范围长度的场景中非常有用，例如在算法中分配空间或进行边界检查。

以下是对 std::ranges::sized_range 的详细解释，包括其定义、要求、用法和代码示例。

------

**基础知识**

- **定义**: std::ranges::sized_range 是一个概念，约束范围类型必须满足 std::ranges::range（基本范围要求）并提供大小信息。

- **头文件**: <ranges>。

- **概念定义**（简化版）:

  cpp

  ```cpp
  template<typename R>
  concept sized_range = std::ranges::range<R> && requires(R& r) {
      { std::ranges::size(r) } -> std::same_as<std::ranges::range_size_t<R>>;
  };
  ```

  - **std::ranges::range<R>**: 要求 R 是一个范围（有 begin() 和 end()）。
  - **std::ranges::size(r)**: 要求可以调用 std::ranges::size 获取大小。
  - **std::ranges::range_size_t<R>**: 返回类型必须是范围的大小类型（通常是 std::size_t 或类似类型）。

- **作用**: 用于模板编程，确保传入的范围支持大小查询。

------

**满足 sized_range 的条件**

一个类型 R 满足 std::ranges::sized_range 如果：

1. 它是一个范围（有 begin() 和 end()）。
2. 可以通过 std::ranges::size(r) 获取其大小，方式包括：
   - r.size()（成员函数）。
   - size(r)（非成员函数，通过 ADL 或标准库提供）。
   - 对于随机访问迭代器，end() - begin()（自动推导）。

**常见满足的类型**

- **std::vector**: 有 size() 成员。
- **std::array**: 有 size() 成员。
- **std::string**: 有 size() 成员。
- **C 风格数组**: 通过 std::ranges::size 支持。
- **std::ranges::subrange（sized 版本）**: 如果指定了大小。

**不满足的类型**

- **std::list**: 有 size()，但不是常量时间操作（C++11 后通常是 O(1)，但标准不保证）。
- **视图（如 std::views::filter）**: 通常不提供大小，因为大小可能动态变化或难以计算。
- **std::ranges::subrange（unsized 版本）**: 未指定大小。

------

**用法示例**

**1. 基本使用**

cpp

```cpp
#include <ranges>
#include <vector>
#include <array>
#include <iostream>

template <std::ranges::sized_range R>
void print_size(R&& r) {
    std::cout << "Size: " << std::ranges::size(r) << "\n";
}

int main() {
    std::vector<int> vec = {1, 2, 3};
    std::array<int, 4> arr = {4, 5, 6, 7};
    int c_arr[] = {8, 9};

    print_size(vec);    // 输出: Size: 3
    print_size(arr);    // 输出: Size: 4
    print_size(c_arr);  // 输出: Size: 2
}
```

- **解释**:
  - print_size 约束参数必须是 sized_range。
  - std::ranges::size 调用底层的 size() 或推导大小。

**2. 与 subrange 结合**

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

template <std::ranges::sized_range R>
void print_subrange_size(R&& r) {
    auto sub = std::ranges::subrange(r.begin() + 1, r.end(), r.size() - 1);
    std::cout << "Subrange size: " << std::ranges::size(sub) << "\n";
}

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    print_subrange_size(vec); // 输出: Subrange size: 4
}
```

- **解释**:
  - subrange 使用 sized 版本，保留大小信息。
  - 因为 vec 满足 sized_range，可以安全调用 size()。

**3. 编译期检查**

cpp

```cpp
#include <ranges>
#include <list>
#include <vector>

template <std::ranges::sized_range R>
void constrained_function(R&&) {}

int main() {
    std::vector<int> vec = {1, 2, 3};
    constrained_function(vec); // OK

    // std::list<int> lst = {1, 2, 3};
    // constrained_function(lst); // 编译错误：std::list 不保证 O(1) size
}
```

- **解释**:
  - std::list 的 size() 在 C++11 后通常是 O(1)，但标准不要求，因此不严格满足 sized_range。

------

**与 std::ranges::range 的区别**

- **std::ranges::range**:
  - 只要求有 begin() 和 end()。
  - 不要求提供大小。
  - 示例：std::list、std::views::filter。
- **std::ranges::sized_range**:
  - 是 range 的子集，要求额外支持 size()。
  - 示例：std::vector、std::array。

------

**关键点**

1. **大小计算**:
   - std::ranges::size(r) 优先调用 r.size()。
   - 如果没有 size()，但迭代器是随机访问的，则用 end() - begin()。
2. **性能**:
   - sized_range 假设 size() 是 O(1) 操作（尽管标准不强制）。
3. **视图支持**:
   - 某些视图（如 std::views::take）可能是 sized_range，但动态视图（如 std::views::filter）通常不是。

------

**中文解释**

**功能**

- **std::ranges::sized_range**: 一个概念，要求范围支持大小查询。
- **用途**: 确保模板参数可以提供元素数量，用于需要长度信息的场景。

**要求**

- 必须是 range（有 begin() 和 end()）。
- 必须支持 std::ranges::size（如 size() 成员或推导）。

**示例**

1. **基本使用**: 打印 vector、array 和 C 数组的大小。
2. **与 subrange**: 创建带大小的子范围。
3. **约束**: 限制函数只接受支持 size() 的范围。

**适用类型**

- 支持：std::vector、std::string、std::array。
- 不支持：std::list（非严格 O(1)）、动态视图。

------

**总结**

std::ranges::sized_range 是 C++20 Ranges 库中的一个重要概念，用于约束范围类型必须提供大小信息。它增强了类型安全性，适用于需要知道范围长度的算法或操作。与 std::ranges::subrange 结合使用时，可以创建带大小的子范围，进一步提升 Ranges 编程的灵活性。

如果你有具体问题或想深入某个用法，请告诉我！