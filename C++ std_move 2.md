# std::move（传统算法）、std::ranges::move（Ranges 版本）

代码:

```C++
#include <vector>
#include <algorithm>
#include <utility>
#include <functional>
#include <string_view>
#include <iostream>

struct Movable {
    int v = 42;
    Movable() = default;
    Movable(Movable&& other) : v(std::exchange(other.v, -1)) {}
    Movable& operator=(Movable&& other) {
        v = std::exchange(other.v, -1);
        return *this;
    }
};

void format_range(std::string_view label, auto& rng, auto projection);

int main() {
    std::vector<Movable> data;
    data.emplace_back(); data.emplace_back(); data.emplace_back();

    std::vector<Movable> out(3);
    // std::ranges::copy(data, out.begin()); // Wouldn't compile
    std::ranges::move(data, out.begin()); // OK
    // out == {{42}, {42}, {42}}, data == {{-1}, {-1}, {-1}}

    format_range("out", out, &Movable::v);
    format_range("data", data, &Movable::v);

    // Overlapping ranges
    std::vector<int> rng{1,2,3,4,5,6,7,8,9};
    // The begining of output range cannot overlap with input range
    //     | input range |
    // | output range | 
    std::move(
        rng.begin()+4, rng.end(), // input
        rng.begin()); // output
    // rng == {5, 6, 7, 8, 9, _, _, _, _}

    format_range("rng", rng, std::identity{});

    rng = {1,2,3,4,5,6,7,8,9};
    // The begining of input range cannot overlap with output range
    // | input range | 
    //    | output range |
    std::move_backward(
        rng.begin(), rng.end()-4, // input
        rng.end()); // output
    // rng == {_, _, _, _, 1, 2, 3, 4, 5}

    format_range("rng", rng, std::identity{});

    // Copy-only types:
    struct CopyOnly {};
    const std::vector<CopyOnly> src(5); // immutable source
    std::vector<CopyOnly> dst(5);

    // We cannot move from an immutable container, defaults to copy
    std::move(src.begin(), src.end(), dst.begin());
    std::ranges::move(src, dst.begin());
}

#include <iostream>
#include <string>
#include <utility>

void format_range(std::string_view label, auto& rng, auto projection) {
    std::string delim = "";
    std::cout << label << " == {";
    for (auto &v : rng)
        std::cout << std::exchange(delim, ", ") << std::invoke(projection, v);
    std::cout << "}\n";
}
```

这段代码展示了 C++ 中移动语义的实现和使用，涉及 std::move（传统算法）、std::ranges::move（Ranges 版本）、std::move_backward，以及它们在不同场景下的行为。代码还定义了一个支持移动但不支持拷贝的类型 Movable，并处理重叠范围和不可移动类型的情况。以下是逐步解释。

------

代码概览

- 定义 Movable 类型，支持移动但不支持拷贝。
- 使用移动算法将数据从一个容器转移到另一个容器。
- 处理重叠范围的移动。
- 处理不可移动类型的特殊情况。
- 使用 format_range 输出结果。

------

关键组件

1. **头文件**

cpp

```cpp
#include <vector>
#include <algorithm>
#include <utility>
#include <functional>
#include <string_view>
#include <iostream>
```

- <vector>：提供 std::vector。
- <algorithm>：提供 std::move 和 std::move_backward。
- <utility>：提供 std::exchange 和 std::move。
- <functional>：提供 std::invoke 和 std::identity。
- <string_view>：提供 std::string_view。
- <iostream>：用于输出。
- **Movable 类型**

cpp

```cpp
struct Movable {
    int v = 42;
    Movable() = default;
    Movable(Movable&& other) : v(std::exchange(other.v, -1)) {}
    Movable& operator=(Movable&& other) {
        v = std::exchange(other.v, -1);
        return *this;
    }
};
```

- **特性**：
  - 默认构造函数初始化 v = 42。
  - 移动构造函数和赋值运算符使用 std::exchange 将源对象的 v 替换为 -1，实现移动。
  - 无拷贝构造函数或赋值运算符（隐式删除）。
- **format_range 函数**

cpp

```cpp
void format_range(std::string_view label, auto& rng, auto projection) {
    std::string delim = "";
    std::cout << label << " == {";
    for (auto &v : rng)
        std::cout << std::exchange(delim, ", ") << std::invoke(projection, v);
    std::cout << "}\n";
}
```

- **功能**：
  - 输出范围内容，使用 projection 提取值（如 Movable::v）。
  - std::exchange(delim, ", ") 动态切换分隔符，首次为空，后续为逗号。
- **用法**：
  - 支持任意范围和投影函数（如成员指针或 std::identity）。
- **main 函数**

**移动 Movable**

cpp

```cpp
std::vector<Movable> data;
data.emplace_back(); data.emplace_back(); data.emplace_back();
std::vector<Movable> out(3);
// std::ranges::copy(data, out.begin()); // Wouldn't compile
std::ranges::move(data, out.begin());
format_range("out", out, &Movable::v);
format_range("data", data, &Movable::v);
```

- **data**：

  - 初始化 3 个 Movable 对象，v = 42。

- **out**：

  - 预分配 3 个默认构造的 Movable（v = 42）。

- **std::ranges::copy**：

  - 失败，因为 Movable 无拷贝构造函数。

- **std::ranges::move**：

  - 成功，将 data 中的元素移动到 out，data 的元素被置为 v = -1。

- **结果**：

  - out == {42, 42, 42}。
  - data == {-1, -1, -1}。

- **输出**：

  ```text
  out == {42, 42, 42}
  data == {-1, -1, -1}
  ```

**重叠范围：std::move**

cpp

```cpp
std::vector<int> rng{1,2,3,4,5,6,7,8,9};
std::move(rng.begin()+4, rng.end(), rng.begin());
format_range("rng", rng, std::identity{});
```

- **std::move**：

  - 从 [begin+4, end)（{5, 6, 7, 8, 9}）移动到 begin。
  - 重叠规则：输出范围起点不能在输入范围之前。

- **过程**：

  - rng[0] = 5, rng[1] = 6, ..., rng[4] = 9。
  - 后半部分未定义（_）。

- **结果**：

  - rng == {5, 6, 7, 8, 9, _, _, _, _}。

- **输出**：

  ```text
  rng == {5, 6, 7, 8, 9, _, _, _, _}
  ```

**重叠范围：std::move_backward**

cpp

```cpp
rng = {1,2,3,4,5,6,7,8,9};
std::move_backward(rng.begin(), rng.end()-4, rng.end());
format_range("rng", rng, std::identity{});
```

- **std::move_backward**：

  - 从 [begin, end-4)（{1, 2, 3, 4, 5}）向后移动到 end。
  - 重叠规则：输入范围起点不能在输出范围之后。

- **过程**：

  - 从右向左：rng[8] = 5, rng[7] = 4, ..., rng[4] = 1。
  - 前半部分未定义（_）。

- **结果**：

  - rng == {_, _, _, _, 1, 2, 3, 4, 5}。

- **输出**：

  ```text
  rng == {_, _, _, _, 1, 2, 3, 4, 5}
  ```

**不可移动类型**

cpp

```cpp
struct CopyOnly {};
const std::vector<CopyOnly> src(5);
std::vector<CopyOnly> dst(5);
std::move(src.begin(), src.end(), dst.begin());
std::ranges::move(src, dst.begin());
```

- **CopyOnly**：
  - 无移动语义，默认支持拷贝。
- **src**：
  - const 向量，无法移动。
- **std::move 和 std::ranges::move**：
  - 检测到源不可移动，退化为拷贝。
  - dst 接收 src 的拷贝。
- **结果**：
  - 无输出，但 dst 被填充为 src 的副本。

------

为什么这样工作？

1. **std::ranges::move**：
   - 调用移动构造函数/赋值运算符，适合 Movable。
2. **std::move vs std::move_backward**：
   - 处理重叠范围，方向不同：
     - std::move：从左到右。
     - std::move_backward：从右到左。
3. **退化为拷贝**：
   - 当源不可移动（如 const），move 调用拷贝。
4. **投影**：
   - format_range 使用投影提取值，灵活输出。

------

输出

```text
out == {42, 42, 42}
data == {-1, -1, -1}
rng == {5, 6, 7, 8, 9, _, _, _, _}
rng == {_, _, _, _, 1, 2, 3, 4, 5}
```

------

使用场景

- **资源转移**：
  - 将对象从一个容器移动到另一个。
- **重叠操作**：
  - 在同一容器内调整数据。
- **兼容性**：
  - 处理不可移动类型。

------

总结

- Movable 展示了移动语义的实现。
- std::ranges::move 转移数据，std::move 和 std::move_backward 处理重叠。
- const 源退化为拷贝。
- 代码展示了移动算法的灵活性和应用。