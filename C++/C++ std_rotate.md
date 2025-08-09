# std::rotate

std::rotate 是 C++ 标准库中的一个算法，定义在 <algorithm> 头文件中。它用于将范围内的元素进行**旋转**操作，使得指定位置的元素成为新的“首元素”，其余元素按顺序重新排列。旋转操作不会改变范围的大小或元素内容，只是调整元素的顺序。

以下是对 std::rotate 的详细解释：

------

定义

cpp

```cpp
#include <algorithm>

namespace std {
    template<class ForwardIt>
    ForwardIt rotate(ForwardIt first, ForwardIt new_first, ForwardIt last);
}
```

- **ForwardIt**: 正向迭代器类型（支持至少正向遍历）。
- **first**: 范围的起始迭代器。
- **new_first**: 将要成为新首元素的迭代器。
- **last**: 范围的结束迭代器。
- 返回值：指向旋转后“旧首元素”的迭代器。

------

行为

- **std::rotate**：
  - 将范围 [first, last) 的元素重新排列，使得 new_first 指向的元素成为新的首元素。
  - 原 [first, new_first) 的元素被移动到范围末尾。
- **旋转逻辑**：
  - 假设范围是 [a, b, c, d, e]，new_first 指向 c。
  - 旋转后：[c, d, e, a, b]。
- **返回值**：
  - 指向旋转后“旧首元素”（first 原来指向的元素）的新位置。

------

前提条件

- **迭代器要求**：
  - ForwardIt 必须是正向迭代器（std::forward_iterator_tag）。
  - [first, last) 必须是有效范围。
- **元素要求**：
  - 元素必须支持移动赋值或交换（std::swap）。
- **new_first**：
  - 必须在 [first, last) 内。

------

示例代码

示例 1：基本用法

cpp

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    std::rotate(vec.begin(), vec.begin() + 2, vec.end());

    std::cout << "Rotated: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Rotated: 3 4 5 1 2
```

- **解释**：
  - 原始：{1, 2, 3, 4, 5}。
  - new_first 是 vec.begin() + 2（指向 3）。
  - 旋转后：{3, 4, 5, 1, 2}。
  - 返回值指向 1（新位置）。

示例 2：旋转到开头

cpp

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    std::rotate(vec.begin(), vec.begin(), vec.end()); // 无变化

    std::cout << "Rotated: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Rotated: 1 2 3 4 5
```

- **解释**：
  - new_first == first，不发生旋转。

示例 3：旋转到末尾

cpp

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    auto it = std::rotate(vec.begin(), vec.end(), vec.end());

    std::cout << "Rotated: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    std::cout << "New position of old first: " << *it << "\n";

    return 0;
}
```

输出

```text
Rotated: 1 2 3 4 5
New position of old first: 1
```

- **解释**：
  - new_first == last，不发生旋转。
  - 返回值是 vec.begin()。

------

时间复杂度

- **O(n)**：
  - n 是范围大小（last - first）。
  - 使用高效的元素交换算法（如三向旋转或 GCD 优化）。

------

使用场景

1. **循环移位**：

   - 将数组或列表中的元素向左或右移动。

   cpp

   ```cpp
   std::rotate(vec.begin(), vec.begin() + 1, vec.end()); // 左移
   ```

2. **重新排序**：

   - 将特定元素移到开头或特定位置。

3. **算法辅助**：

   - 在排序或分区中调整元素顺序。

------

注意事项

1. **C++98 起可用**：
   - 基础功能自 C++98，支持 C++20 的 constexpr（若迭代器支持）。
2. **迭代器类型**：
   - 需要正向迭代器，std::list 支持，std::forward_list 不完全适用（需双向迭代器优化）。
3. **返回值**：
   - 可用于跟踪旋转后的元素位置。
4. **边界检查**：
   - new_first 超出范围会导致未定义行为。

------

与其他算法的对比

| 算法             | 功能          | 时间复杂度 | 要求       |
| ---------------- | ------------- | ---------- | ---------- |
| std::rotate      | 旋转范围      | O(n)       | 正向迭代器 |
| std::shift_left  | 左移（C++20） | O(n)       | 正向迭代器 |
| std::shift_right | 右移（C++20） | O(n)       | 双向迭代器 |
| std::reverse     | 反转范围      | O(n)       | 双向迭代器 |

------

实现原理（简化）

std::rotate 通常基于以下步骤：

1. 计算 new_first - first（左移距离）和 last - new_first（右移部分）。
2. 使用 std::swap 或移动操作交换元素。
3. 优化时可能利用 GCD（最大公约数）减少交换次数。

------

总结

std::rotate 是一个高效的算法，用于在范围内旋转元素：

- 将 new_first 指定的元素移到开头，其余元素按顺序调整。
- 返回旧首元素的新位置。
- 时间复杂度 O(n)，适用于需要重新排列元素的场景。 它是 C++ 标准库中的经典工具，广泛用于数据操作。如果你有具体问题或想探讨用法，请告诉我！