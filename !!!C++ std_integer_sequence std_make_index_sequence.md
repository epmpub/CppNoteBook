#  *std::integer_sequence* std::make_index_sequence

The *std::integer_sequence* is a C++14 metaprogramming utility. As its name suggests, it can represent a compile-time sequence of integers.

The underlying integer type can be specified, and the *std::index_sequence* alias offers a shortcut for *std::size_t*.

*std::make_integer_sequence* and *std::make_index_sequence* are helpers that produce sequences of integers from *0* to *N-1*.

The primary use case of *std::integer_sequence* is for manipulating parameter packs.



```C++
#include <tuple>
#include <utility>
#include <cstdint>
#include <algorithm>
#include <array>
#include <print>

// C++14 example, reorder elements in a tuple
template <typename Tuple, size_t... I>
constexpr auto shuffle(const Tuple& t, std::index_sequence<I...>) {
    return std::make_tuple(std::get<I>(t)...);
}

// C++17 example, runtime version of std::get
template <size_t idx, typename Tuple, typename T>
bool set_if(const Tuple& t, size_t index, T& out) {
    // if idx == index, set out to the corresponding value
    // and return true
    return (index == idx) && (out = std::get<idx>(t), true);
}

template <typename Tuple, typename T, size_t... I>
void get_impl(const Tuple& t, size_t index, T& out, std::index_sequence<I...>) {
    // linear iteration over all elements of the tuple
    // logical or provides early termination
    (set_if<I>(t, index, out) || ...);
}

template <typename Tuple, typename T>
void get(Tuple& t, size_t index, T& out) {
    get_impl(t, index, out, std::make_index_sequence<std::tuple_size_v<Tuple>>{});
}

// C++20, sort elements of a tuple by size

// Make a sorted list of indices by size
template <typename Tuple, size_t... I>
constexpr auto sort_impl(std::index_sequence<I...>) {
    // Array of indices
    std::array<size_t, sizeof...(I)> idxs{{I...}};
    // Array of element sizes
    std::array<size_t, sizeof...(I)> sizes{{sizeof(std::tuple_element_t<I, Tuple>)...}};
    // Sort the array of indices
    std::ranges::sort(idxs, [&](size_t l, size_t r) {
        return sizes[l] < sizes[r];
    });
    // Return the sorted indices
    return idxs;
}

// Convert an array of size_t into an index sequence
template <auto Arr, std::size_t... I>
constexpr auto to_sequence(std::index_sequence<I...>) {
    return std::index_sequence<Arr[I]...>{};
}

template <typename Tuple>
auto sort(Tuple t) {
    using idxs = std::make_index_sequence<std::tuple_size_v<Tuple>>;
    // We can re-use our C++14 shuffle function
    return shuffle(t, to_sequence<sort_impl<Tuple>(idxs{})>(idxs{}));
}

int main() {
    std::tuple<int, double, bool> t1{42, 3.14, false};
    auto shuffled = shuffle(t1, std::index_sequence<1,2,0>{});
    // decltype(shuffled) == std::tuple<double,bool,int>
    // shuffled == {3.14, false, 42}

    static_assert(std::is_same_v<decltype(shuffled), std::tuple<double,bool,int>>);
    static_assert(shuffle(std::tuple{1,2}, std::index_sequence<1,0>{}) == std::tuple{2,1});

    std::println("shuffled == [{}, {}, {}]", get<0>(shuffled), get<1>(shuffled), get<2>(shuffled));

    std::tuple<int, double, bool> t2{2, 4.2, false};
    int v;
    get(t2, 0, v);
    // v == 2

    std::println("v == {}", v);

    get(t2, 1, v);
    // v == 4

    std::println("v == {}", v);

    get(t2, 2, v);
    // v == 0

    std::println("v == {}", v);

    auto sorted = sort(std::tuple{1,4.2,false});
    // decltype(sorted) == std::tuple<bool,int,double>
    // sorted == {false, 1, 4.2}

    std::println("sorted == [{}, {}, {}]", get<0>(sorted), get<1>(sorted), get<2>(sorted));
    static_assert(std::is_same_v<decltype(sorted), std::tuple<bool,int,double>>);
}
```

我来详细解释这段 C++ 代码，它展示了如何使用模板元编程操作 std::tuple，包括重新排序、运行时访问和按元素大小排序。

------

代码结构

使用的头文件：

- <tuple>：提供 std::tuple 和相关工具。
- <utility>：提供 std::index_sequence 等。
- <cstdint>：提供 size_t。
- <algorithm> 和 <array>：用于排序和数组操作。
- <print>：提供 std::println（C++23 特性）。

代码包含三个主要功能：

1. **shuffle**：基于编译期索引序列重新排序元组（C++14）。
2. **get**：运行时根据索引获取元组元素（C++17）。
3. **sort**：按元素大小排序元组（C++20）。

------

1. shuffle - 编译期重新排序元组

定义

cpp

```cpp
template <typename Tuple, size_t... I>
constexpr auto shuffle(const Tuple& t, std::index_sequence<I...>) {
    return std::make_tuple(std::get<I>(t)...);
}
```

解释

- **功能**：根据给定的索引序列 I... 重新排列元组 t 的元素。

- **参数**：

  - Tuple：元组类型。
  - std::index_sequence<I...>：编译期索引序列，例如 <1,2,0>。

- **实现**：

  - 使用 std::get<I>(t) 提取指定位置的元素。
  - 通过参数包展开（...）构造新元组。

- **示例**：

  cpp

  ```cpp
  std::tuple<int, double, bool> t1{42, 3.14, false};
  auto shuffled = shuffle(t1, std::index_sequence<1,2,0>{});
  // shuffled == std::tuple<double, bool, int>{3.14, false, 42}
  ```

- **特点**：编译期操作，高效且类型安全。

------

2. get - 运行时获取元组元素

定义

cpp

```cpp
template <size_t idx, typename Tuple, typename T>
bool set_if(const Tuple& t, size_t index, T& out) {
    return (index == idx) && (out = std::get<idx>(t), true);
}

template <typename Tuple, typename T, size_t... I>
void get_impl(const Tuple& t, size_t index, T& out, std::index_sequence<I...>) {
    (set_if<I>(t, index, out) || ...);
}

template <typename Tuple, typename T>
void get(Tuple& t, size_t index, T& out) {
    get_impl(t, index, out, std::make_index_sequence<std::tuple_size_v<Tuple>>{});
}
```

解释

- **功能**：在运行时根据索引 index 从元组中提取元素并赋值给 out。

- **实现细节**：

  - set_if：检查当前索引 idx 是否等于目标 index，如果是则赋值并返回 true。
  - get_impl：使用折叠表达式 (set_if<I> || ...) 遍历所有索引，找到匹配时赋值并提前终止。
  - get：生成索引序列（0,1,2,...）并调用 get_impl。

- **示例**：

  cpp

  ```cpp
  std::tuple<int, double, bool> t2{2, 4.2, false};
  int v;
  get(t2, 0, v);  // v == 2
  get(t2, 1, v);  // v == 4 (隐式转换为 int)
  get(t2, 2, v);  // v == 0 (false 转换为 0)
  ```

- **特点**：运行时操作，类型 T 必须兼容元组元素类型，否则可能编译失败或行为未定义。

------

3. sort - 按元素大小排序元组

定义

cpp

```cpp
template <typename Tuple, size_t... I>
constexpr auto sort_impl(std::index_sequence<I...>) {
    std::array<size_t, sizeof...(I)> idxs{{I...}};  // 索引数组
    std::array<size_t, sizeof...(I)> sizes{{sizeof(std::tuple_element_t<I, Tuple>)...}};  // 大小数组
    std::ranges::sort(idxs, [&](size_t l, size_t r) {
        return sizes[l] < sizes[r];
    });
    return idxs;
}

template <auto Arr, std::size_t... I>
constexpr auto to_sequence(std::index_sequence<I...>) {
    return std::index_sequence<Arr[I]...>{};
}

template <typename Tuple>
auto sort(Tuple t) {
    using idxs = std::make_index_sequence<std::tuple_size_v<Tuple>>;
    return shuffle(t, to_sequence<sort_impl<Tuple>(idxs{})>(idxs{}));
}
```

解释

- **功能**：按元素类型的大小（sizeof）对元组重新排序。

- **实现步骤**：

  1. **sort_impl**：
     - 创建索引数组 idxs（如 {0,1,2}）。
     - 创建大小数组 sizes（如 {4,8,1} 表示 int, double, bool）。
     - 使用 std::ranges::sort 按大小对 idxs 排序（如 {2,0,1}）。
  2. **to_sequence**：
     - 将排序后的数组（如 {2,0,1}）转换为 std::index_sequence<2,0,1>。
  3. **sort**：
     - 调用 shuffle 使用排序后的索引序列重新排列元组。

- **示例**：

  cpp

  ```cpp
  auto sorted = sort(std::tuple{1, 4.2, false});
  // sorted == std::tuple<bool, int, double>{false, 1, 4.2}
  // sizeof(bool) == 1, sizeof(int) == 4, sizeof(double) == 8
  ```

- **特点**：编译期计算排序顺序，运行时构造新元组。

------

主函数输出

cpp

```cpp
int main() {
    // shuffle
    std::tuple<int, double, bool> t1{42, 3.14, false};
    auto shuffled = shuffle(t1, std::index_sequence<1,2,0>{});
    std::println("shuffled == [{}, {}, {}]", get<0>(shuffled), get<1>(shuffled), get<2>(shuffled));
    // 输出: shuffled == [3.14, false, 42]

    // get
    std::tuple<int, double, bool> t2{2, 4.2, false};
    int v;
    get(t2, 0, v); std::println("v == {}", v);  // v == 2
    get(t2, 1, v); std::println("v == {}", v);  // v == 4
    get(t2, 2, v); std::println("v == {}", v);  // v == 0

    // sort
    auto sorted = sort(std::tuple{1,4.2,false});
    std::println("sorted == [{}, {}, {}]", get<0>(sorted), get<1>(sorted), get<2>(sorted));
    // 输出: sorted == [false, 1, 4.2]
}
```

------

关键概念

1. **模板元编程**：
   - shuffle 和 sort 使用编译期索引序列操作元组。
   - std::index_sequence 是生成和处理索引的关键工具。
2. **运行时与编译期结合**：
   - get 在运行时处理索引，依赖折叠表达式实现提前终止。
   - sort 在编译期计算排序顺序，运行时应用。
3. **类型安全性**：
   - 使用 static_assert 验证类型正确性。
   - sort 基于 sizeof 而非值，确保确定性。

这段代码展示了现代 C++（C++14/17/20）的强大能力，结合元编程和运行时特性操作元组。如果有具体部分需要深入探讨，请告诉我！