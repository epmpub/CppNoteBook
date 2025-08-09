# std::copy 和 std::copy_backward

我来为你详细解释这段代码，逐步分析 std::copy 和 std::copy_backward 的作用以及它们在操作 std::vector<int> 时的行为。这段代码展示了如何使用 C++ 标准库中的算法来复制序列，并处理范围重叠的情况。

```c++
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> data{ 1, 2, 3, 4, 5, 6, 7, 8, 9 };

    // OK for std::copy
    //         [ source range      ]
    // [ destination range ]
    std::copy(data.begin() + 1, data.end(), data.begin());
    // data == {2, 3, 4, 5, 6, 7, 8, 9, 9}

    for (auto v : data)
        std::cout << v << " ";
    std::cout << '\n';

    data = {1, 2, 3, 4, 5, 6, 7, 8, 9};

    // OK for std::copy_backward
    // [ source range      ]
    //         [ destination range ]
    std::copy_backward(data.begin(), data.begin() + 8, data.end());
    // data == {1, 1, 2, 3, 4, 5, 6, 7, 8}

    for (auto v : data)
        std::cout << v << " ";
    std::cout << '\n';
}
```



------

1. 使用 std::copy

cpp

```cpp
#include <algorithm>
#include <vector>

std::vector<int> data{ 1, 2, 3, 4, 5, 6, 7, 8, 9 };

std::copy(data.begin() + 1, data.end(), data.begin());
// data == {2, 3, 4, 5, 6, 7, 8, 9, 9}
```

解释

- **std::copy**:

  - 定义在 <algorithm> 中，用于将一个范围内的元素复制到另一个范围。

  - 函数签名：

    cpp

    ```cpp
    template <class InputIt, class OutputIt>
    OutputIt copy(InputIt first, InputIt last, OutputIt d_first);
    ```

  - 参数：

    - first, last: 源范围 [first, last)，即要复制的元素。
    - d_first: 目标范围的起始位置。

  - 返回值：目标范围的结束迭代器（即复制后最后一个元素之后的迭代器）。

- **代码逻辑**:

  1. **初始状态**: data = {1, 2, 3, 4, 5, 6, 7, 8, 9}。
  2. **源范围**: data.begin() + 1 到 data.end()，即 {2, 3, 4, 5, 6, 7, 8, 9}（从第 2 个元素到最后）。
  3. **目标范围**: 从 data.begin() 开始，即从第 1 个元素（索引 0）开始覆盖。
  4. **复制过程**:
     - 将 {2, 3, 4, 5, 6, 7, 8, 9} 复制到从 data[0] 开始的位置。
     - 源范围有 8 个元素（end() - (begin() + 1) = 9 - 1 = 8），目标范围从 data[0] 开始覆盖 8 个元素。
     - 结果：data = {2, 3, 4, 5, 6, 7, 8, 9, 9}。
  5. **为什么最后是 9**:
     - 源范围只包含 8 个元素（2 到 9），复制到目标范围后，data[8]（第 9 个元素）保持不变，仍为原始的 9。

- **重叠问题**:

  - 源范围 [data.begin() + 1, data.end()) 和目标范围 [data.begin(), data.begin() + 8) 有重叠。
  - std::copy 是从左到右复制的：
    - data[0] = data[1]（2 覆盖 1）
    - data[1] = data[2]（3 覆盖 2）
    - ...
    - data[7] = data[8]（9 覆盖 8）
  - 因为复制是从头开始，源数据在被覆盖前已经被读取，所以结果是正确的。

- **结果**:

  - data == {2, 3, 4, 5, 6, 7, 8, 9, 9}。

------

2. 使用 std::copy_backward

cpp

```cpp
data = {1, 2, 3, 4, 5, 6, 7, 8, 9};

std::copy_backward(data.begin(), data.begin() + 8, data.end());
// data == {1, 1, 2, 3, 4, 5, 6, 7, 8}
```

解释

- **std::copy_backward**:

  - 定义在 <algorithm> 中，与 std::copy 类似，但从右向左复制。

  - 函数签名：

    cpp

    ```cpp
    template <class BidirIt1, class BidirIt2>
    BidirIt2 copy_backward(BidirIt1 first, BidirIt1 last, BidirIt2 d_last);
    ```

  - 参数：

    - first, last: 源范围 [first, last)。
    - d_last: 目标范围的结束位置（复制从右向左进行）。

  - 返回值：目标范围的起始迭代器（复制后第一个元素之前的迭代器）。

  - 要求：迭代器必须是双向迭代器（Bidirectional Iterator），如 std::vector 的迭代器。

- **代码逻辑**:

  1. **初始状态**: data = {1, 2, 3, 4, 5, 6, 7, 8, 9}。
  2. **源范围**: data.begin() 到 data.begin() + 8，即 {1, 2, 3, 4, 5, 6, 7, 8}（前 8 个元素）。
  3. **目标范围**: 以 data.end() 为结束位置，向前复制 8 个元素，即 [data.begin() + 1, data.end())。
  4. **复制过程**:
     - 从右向左复制：
       - data[8] = data[7]（8 覆盖 9）
       - data[7] = data[6]（7 覆盖 8）
       - data[6] = data[5]（6 覆盖 7）
       - ...
       - data[1] = data[0]（1 覆盖 2）
     - 结果：data = {1, 1, 2, 3, 4, 5, 6, 7, 8}。
  5. **为什么第一个是 1**:
     - 源范围有 8 个元素（1 到 8），复制到目标范围 [data[1], data[9])。
     - data[0] 未被覆盖，保持原始值 1。

- **重叠问题**:

  - 源范围 [data.begin(), data.begin() + 8) 和目标范围 [data.begin() + 1, data.end()) 有重叠。
  - std::copy_backward 从右到左复制：
    - 当复制 data[1] = data[0] 时，data[0] 的值（1）还未被修改。
  - 这种方向保证了重叠时源数据的正确性。

- **结果**:

  - data == {1, 1, 2, 3, 4, 5, 6, 7, 8}。

------

std::copy 与 std::copy_backward 的对比

| 特性       | std::copy                  | std::copy_backward         |
| ---------- | -------------------------- | -------------------------- |
| 复制方向   | 从左到右                   | 从右到左                   |
| 目标参数   | 目标范围的起始位置         | 目标范围的结束位置         |
| 迭代器要求 | 输入迭代器和输出迭代器     | 双向迭代器                 |
| 重叠处理   | 适用于目标在源左侧或无重叠 | 适用于目标在源右侧或无重叠 |

- **为什么需要两种算法**:
  - 当源范围和目标范围重叠时，复制方向决定了结果的正确性。
  - std::copy 适合从前往后移动数据（如第一个例子）。
  - std::copy_backward 适合从后往前移动数据（如第二个例子）。

------

逐步分解

std::copy 示例

初始：{1, 2, 3, 4, 5, 6, 7, 8, 9}

- data[0] = data[1]: {2, 2, 3, 4, 5, 6, 7, 8, 9}
- data[1] = data[2]: {2, 3, 3, 4, 5, 6, 7, 8, 9}
- ...
- data[7] = data[8]: {2, 3, 4, 5, 6, 7, 8, 9, 9} 最终：{2, 3, 4, 5, 6, 7, 8, 9, 9}

std::copy_backward 示例

初始：{1, 2, 3, 4, 5, 6, 7, 8, 9}

- data[8] = data[7]: {1, 2, 3, 4, 5, 6, 7, 8, 8}
- data[7] = data[6]: {1, 2, 3, 4, 5, 6, 7, 7, 8}
- ...
- data[1] = data[0]: {1, 1, 2, 3, 4, 5, 6, 7, 8} 最终：{1, 1, 2, 3, 4, 5, 6, 7, 8}

------

实际应用

- **std::copy**:
  - 删除首元素并向左移动（如第一个例子）。
  - 将数据从一个容器复制到另一个容器。
- **std::copy_backward**:
  - 删除末元素并向右移动（如第二个例子）。
  - 在重叠范围中安全移动数据。

希望这个解释清楚地解答了你的疑问！如果需要更多示例或深入探讨，欢迎继续提问。