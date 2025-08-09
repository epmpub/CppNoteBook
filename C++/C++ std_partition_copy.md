# std::partition_copy 的用法

在 C++ 里，`std::partition_copy` 是标准库中的一个算法，它的作用是依据给定的**谓词**（一个返回布尔值的函数），把输入范围里的元素**分别复制**到**两个不同的输出范围**，从而实现**分区**的效果。下面从**功能、参数、示例、与 `std::partition` 的区别**这几个方面来详细理解：

### **1. 核心功能**

- **不改变原容器**：和 `std::partition` 不一样，`std::partition_copy` 不会直接对**输入范围的元素顺序**进行调整，而是把元素**复制**到两个**新的输出范围**。
- 分区逻辑：
  - 对于**满足谓词条件**的元素（比如 `predicate(element) == true`），会被复制到**第一个输出范围**。
  - 对于**不满足谓词条件**的元素，会被复制到**第二个输出范围**。

### **2. 函数签名（C++11 及以后）**

cpp

```cpp
template <class InputIt, class OutputIt1, class OutputIt2, class UnaryPredicate>
std::pair<OutputIt1, OutputIt2> partition_copy(
    InputIt first, InputIt last,
    OutputIt1 d_first1, OutputIt2 d_first2,
    UnaryPredicate p
);
```

- 参数说明

  ：

  - `first, last`：表示输入范围（需要进行分区的源数据范围）。
  - `d_first1, d_first2`：分别是两个输出范围的起始迭代器。
  - `p`：是一个谓词，用于判断元素应该被分到哪个分区。

- **返回值**：返回一个 `std::pair`，包含两个输出范围的**结束迭代器**（即 `d_first1 + 满足条件的元素个数` 和 `d_first2 + 不满足条件的元素个数`）。

### **3. 示例解析**

**需求**：把一个 `vector` 中的**偶数**和**奇数**分别复制到两个不同的 `vector` 中，同时保持原 `vector` 不变。

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6};
    std::vector<int> evens, odds; // 两个输出容器

    // 使用 std::partition_copy 进行分区复制
    auto [even_end, odd_end] = std::partition_copy(
        nums.begin(), nums.end(),
        std::back_inserter(evens),  // 第一个输出范围（偶数）
        std::back_inserter(odds),   // 第二个输出范围（奇数）
        [](int x) { return x % 2 == 0; } // 谓词：判断是否为偶数
    );

    // 输出结果
    std::cout << "原容器: ";
    for (int x : nums) std::cout << x << " "; // 原顺序不变：1 2 3 4 5 6

    std::cout << "\n偶数: ";
    for (int x : evens) std::cout << x << " "; // 偶数：2 4 6（顺序与原容器相同）

    std::cout << "\n奇数: ";
    for (int x : odds) std::cout << x << " "; // 奇数：1 3 5（顺序与原容器相同）

    return 0;
}
```

### **4. 与 `std::partition` 的对比**

| 特性               | `std::partition`                         | `std::partition_copy`                        |
| ------------------ | ---------------------------------------- | -------------------------------------------- |
| **是否修改原容器** | 是（会重排元素的顺序）                   | 否（只是复制元素，原容器保持不变）           |
| **输出方式**       | 直接在原容器内分区，返回**划分点迭代器** | 复制到**两个独立的输出范围**                 |
| **适用场景**       | 想要**原地分区**并且节省空间时           | 需要**保留原数据**，或者**输出到不同容器**时 |

### **5. 关键细节**

- **输出范围的大小**：调用者要保证输出范围有足够的空间，或者使用 `std::back_inserter` 等插入迭代器来动态扩展容器。
- **元素顺序**：输入范围内元素的**相对顺序**在输出时会被保留（这一点和 `stable_partition` 类似）。
- **复杂度**：时间复杂度是线性的，即 `O(N)`，其中 `N` 是输入范围中元素的个数。

### **总结**

`std::partition_copy` 就像是一个 **“分区复印机”**，它能把数据按照条件 “复印” 到两个不同的地方，同时不会对原数据造成任何改动。当你需要**保留原始数据**或者**将数据分流到不同容器 ** 时，这个算法就非常合适。

