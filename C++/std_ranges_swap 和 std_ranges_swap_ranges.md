# std::ranges::swap 和 std::ranges::swap_ranges

代码:

```C++
#include <algorithm>
#include <vector>
#include <print>

int main() {
    std::vector<int> first{1,2,3,4};
    std::vector<int> second{9,8,7,6};
    second.reserve(128);

    // first == {1,2,3,4}, second == {9,8,7,6}
    // first.capacity() == 4, second.capacity() == 128
    std::println("first == {}, second == {}", first, second);
    std::println("first.capacity() == {}, second.capacity() == {}\n",
        first.capacity(), second.capacity());

    std::ranges::swap(first, second);
    // first == {9,8,7,6}, second == {1,2,3,4}
    // first.capacity() == 128, second.capacity() == 4
    std::println("first == {}, second == {}", first, second);
    std::println("first.capacity() == {}, second.capacity() == {}\n",
        first.capacity(), second.capacity());

    std::ranges::swap_ranges(first, second);
    // first == {1,2,3,4}, second == {9,8,7,6}
    // first.capacity() == 128, second.capacity() == 4
    std::println("first == {}, second == {}", first, second);
    std::println("first.capacity() == {}, second.capacity() == {}\n",
        first.capacity(), second.capacity());

    // Unlike swap, swap_ranges can swap between ranges of different types
    std::array<int, 4> arr{0,0,0,0};

    std::ranges::swap_ranges(first, arr);
    // first == {0,0,0,0}, arr == {1,2,3,4}
    std::println("first == {}, arr == {}", first, arr);
}
```

这段代码展示了 C++ 中两种交换相关的算法：std::ranges::swap 和 std::ranges::swap_ranges，它们分别用于交换整个容器和容器中的元素范围。代码通过示例展示了它们的行为和区别，特别是容量交换和类型兼容性。以下是逐步解释。

------

代码概览

- 使用 std::ranges::swap 交换两个向量的内容和容量。
- 使用 std::ranges::swap_ranges 交换两个向量中的元素。
- 使用 std::ranges::swap_ranges 交换向量和数组的元素。

------

关键组件

1. **头文件**

cpp

```cpp
#include <algorithm>
#include <vector>
#include <print>
```

- <algorithm>：提供 std::ranges::swap 和 std::ranges::swap_ranges（通过 <ranges>）。
- <vector>：提供 std::vector。
- <print>：C++23 的 std::println 用于格式化输出。
- **初始状态**

cpp

```cpp
std::vector<int> first{1,2,3,4};
std::vector<int> second{9,8,7,6};
second.reserve(128);
std::println("first == {}, second == {}", first, second);
std::println("first.capacity() == {}, second.capacity() == {}\n",
    first.capacity(), second.capacity());
```

- **first 和 second**：

  - first = {1, 2, 3, 4}，容量默认等于大小（4）。
  - second = {9, 8, 7, 6}，调用 reserve(128) 后容量为 128。

- **输出**：

  ```text
  first == {1, 2, 3, 4}, second == {9, 8, 7, 6}
  first.capacity() == 4, second.capacity() == 128
  ```

- **std::ranges::swap**

cpp

```cpp
std::ranges::swap(first, second);
std::println("first == {}, second == {}", first, second);
std::println("first.capacity() == {}, second.capacity() == {}\n",
    first.capacity(), second.capacity());
```

- **std::ranges::swap**：

  - 原型：swap(obj1, obj2)。
  - 交换两个对象的内容，包括内部状态（如容量）。
  - 对于 std::vector，交换指针、元素和容量。

- **效果**：

  - first 变为 {9, 8, 7, 6}，容量 128。
  - second 变为 {1, 2, 3, 4}，容量 4。

- **输出**：

  ```text
  first == {9, 8, 7, 6}, second == {1, 2, 3, 4}
  first.capacity() == 128, second.capacity() == 4
  ```

- **std::ranges::swap_ranges**

cpp

```cpp
std::ranges::swap_ranges(first, second);
std::println("first == {}, second == {}", first, second);
std::println("first.capacity() == {}, second.capacity() == {}\n",
    first.capacity(), second.capacity());
```

- **std::ranges::swap_ranges**：

  - 原型：swap_ranges(range1, range2)。
  - 交换两个范围中的元素，不影响容器属性（如容量）。
  - 要求两个范围长度相等。

- **效果**：

  - first = {1, 2, 3, 4}，容量仍为 128。
  - second = {9, 8, 7, 6}，容量仍为 4。

- **输出**：

  ```text
  first == {1, 2, 3, 4}, second == {9, 8, 7, 6}
  first.capacity() == 128, second.capacity() == 4
  ```

- **不同类型交换**

cpp

```cpp
std::array<int, 4> arr{0,0,0,0};
std::ranges::swap_ranges(first, arr);
std::println("first == {}, arr == {}", first, arr);
```

- **arr**：

  - 初始为 {0, 0, 0, 0}。

- **std::ranges::swap_ranges**：

  - 支持不同类型的范围（如 std::vector 和 std::array），只要元素类型兼容。
  - 交换 first 和 arr 的元素。

- **效果**：

  - first = {0, 0, 0, 0}，容量仍为 128。
  - arr = {1, 2, 3, 4}。

- **输出**：

  ```text
  first == {0, 0, 0, 0}, arr == {1, 2, 3, 4}
  ```

------

为什么这样工作？

1. **std::ranges::swap**：
   - 调用 std::vector 的 swap 成员函数，交换所有内部状态。
   - 时间复杂度：O(1)。
2. **std::ranges::swap_ranges**：
   - 逐元素交换，仅修改内容，不影响容器属性。
   - 时间复杂度：O(n)，n 为范围长度。
3. **类型兼容性**：
   - swap_ranges 支持不同容器类型，依赖元素的可交换性。

------

输出

```text
first == {1, 2, 3, 4}, second == {9, 8, 7, 6}
first.capacity() == 4, second.capacity() == 128

first == {9, 8, 7, 6}, second == {1, 2, 3, 4}
first.capacity() == 128, second.capacity() == 4

first == {1, 2, 3, 4}, second == {9, 8, 7, 6}
first.capacity() == 128, second.capacity() == 4

first == {0, 0, 0, 0}, arr == {1, 2, 3, 4}
```

------

使用场景

- **std::ranges::swap**：
  - 快速交换整个容器，包括容量。
- **std::ranges::swap_ranges**：
  - 交换特定范围，或不同类型容器间的元素。
- **性能优化**：
  - 根据需求选择完整交换还是元素交换。

------

总结

- std::ranges::swap 交换 first 和 second，包括容量。
- std::ranges::swap_ranges 仅交换元素，保留容量。
- 支持 std::vector 和 std::array 的元素交换。
- 代码展示了两种算法的区别和灵活性。
- 