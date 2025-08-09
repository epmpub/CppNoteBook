# std::ranges::iter_swap 

std::ranges::iter_swap 是 C++20 引入的一个函数，定义在 <iterator> 头文件中（注意，不是 <algorithm>）。它是 Ranges 库的一部分，是传统 std::iter_swap 的现代版本。与 std::iter_swap 类似，它用于交换两个迭代器所指向的元素，但它被设计为与 Ranges 生态系统兼容，并提供了更强的类型安全和一致性。

以下是对 std::ranges::iter_swap 的详细解释：

------

定义

cpp

```cpp
#include <iterator>

namespace std::ranges {
    inline constexpr auto iter_swap = [](auto&& it1, auto&& it2) 
        noexcept(noexcept(ranges::swap(*it1, *it2))) 
        -> decltype(ranges::swap(*it1, *it2)) 
    {
        return ranges::swap(*it1, *it2);
    };
}
```

- **it1, it2**: 两个迭代器（通常是前向迭代器或更强的类型），指向要交换的元素。
- **返回值**: 调用 ranges::swap(*it1, *it2) 的结果（通常是 void）。
- **noexcept**: 如果 ranges::swap(*it1, *it2) 是无异常的，则整个函数无异常。

------

行为

- std::ranges::iter_swap(it1, it2) 交换迭代器 it1 和 it2 所指向的两个元素。
- 它通过调用 std::ranges::swap（而不是传统的 std::swap）完成交换。
- std::ranges::swap 是一个定制点（customization point），会优先使用元素类型的自定义 swap（通过 ADL），否则回退到标准交换逻辑。

------

前提条件

- **迭代器类型**：
  - it1 和 it2 必须是可解引用的迭代器（支持 *it）。
  - 通常要求是前向迭代器（std::forward_iterator）或更强的类型。
- **可交换性**：
  - *it1 和 *it2 必须是可交换的（支持 ranges::swap）。
- **有效性**：
  - 迭代器必须指向有效元素。

------

示例代码

示例 1：基本使用

cpp

```cpp
#include <iostream>
#include <vector>
#include <iterator>

int main() {
    std::vector<int> vec = {1, 2, 3, 4};

    // 交换第 1 个和第 3 个元素
    std::ranges::iter_swap(vec.begin() + 1, vec.begin() + 3);

    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出

```text
1 4 3 2
```

示例 2：自定义类型

cpp

```cpp
#include <iostream>
#include <vector>
#include <iterator>

struct Point {
    int x, y;
    Point(int x_ = 0, int y_ = 0) : x(x_), y(y_) {}
    friend std::ostream& operator<<(std::ostream& os, const Point& p) {
        return os << "(" << p.x << ", " << p.y << ")";
    }
    friend void swap(Point& a, Point& b) noexcept { // 自定义 swap
        std::swap(a.x, b.x);
        std::swap(a.y, b.y);
    }
};

int main() {
    std::vector<Point> vec = {{1, 2}, {3, 4}, {5, 6}};

    // 交换第 0 个和第 2 个元素
    std::ranges::iter_swap(vec.begin(), vec.begin() + 2);

    for (const auto& p : vec) {
        std::cout << p << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出

```text
(5, 6) (3, 4) (1, 2)
```

------

与 std::iter_swap 的区别

| 特性     | std::iter_swap           | std::ranges::iter_swap    |
| -------- | ------------------------ | ------------------------- |
| 头文件   | <algorithm>              | <iterator>                |
| 命名空间 | std                      | std::ranges               |
| 实现方式 | 函数模板                 | 内联函数对象（constexpr） |
| 交换函数 | 调用 std::swap           | 调用 std::ranges::swap    |
| C++ 版本 | C++98                    | C++20                     |
| 定制性   | 通过 ADL 调用自定义 swap | 通过 ranges::swap 定制点  |

- **std::ranges::swap**：
  - 是一个定制点对象，比传统的 std::swap 更灵活。
  - 支持 ADL 查找自定义 swap，并在必要时回退到移动赋值。
- **函数对象**：
  - std::ranges::iter_swap 是一个 constexpr 函数对象，而非普通函数模板，这使其更适合 Ranges 生态系统。

------

时间复杂度

- **O(1)**：
  - 与 std::iter_swap 一样，复杂度取决于底层 ranges::swap 的实现。
  - 对于内置类型或简单对象，通常是常数时间。

------

使用场景

1. **Ranges 算法**：

   - 在 C++20 的 Ranges 算法（如 std::ranges::sort）中，std::ranges::iter_swap 是交换元素的标准工具。

   cpp

   ```cpp
   std::ranges::sort(vec); // 内部可能使用 iter_swap
   ```

2. **容器操作**：

   - 交换容器中特定位置的元素，与传统 std::iter_swap 用法相同。

   cpp

   ```cpp
   std::ranges::iter_swap(vec.begin(), vec.end() - 1); // 交换首尾
   ```

3. **自定义迭代器**：

   - 与自定义迭代器或 Ranges 兼容的场景。

------

注意事项

1. **C++20 要求**：
   - 需要支持 C++20 的编译器和标准库。
2. **迭代器兼容性**：
   - it1 和 it2 不必来自同一容器，但必须指向可交换的元素。
3. **异常安全**：
   - 如果 ranges::swap(*it1, *it2) 抛出异常，std::ranges::iter_swap 会传播该异常。
4. **ADL 支持**：
   - **如果元素类型定义了自定义 swap，会被优先调用。**

------

实现（概念性）

std::ranges::iter_swap 的实现是一个 lambda 表达式，简化为：

cpp

```cpp
auto iter_swap = [](auto&& it1, auto&& it2) {
    return std::ranges::swap(*it1, *it2);
};
```

- 它依赖 std::ranges::swap，后者通过 ADL 或默认交换逻辑完成操作。

------

总结

std::ranges::iter_swap 是 C++20 Ranges 库中的现代化交换工具，与传统 std::iter_swap 功能相同，但通过 std::ranges::swap 提供更好的定制性和一致性。它是一个 constexpr 函数对象，适合与 Ranges 算法和现代 C++ 风格配合使用。如果你有具体问题或想探讨某个用法，欢迎继续提问！