# std::next_permutation 和 std::prev_permutation 

C++ 标准库中的 std::next_permutation 和 std::prev_permutation 来生成排列，分别用于两个不同的场景。

------

1. 第一个例子：使用 std::next_permutation

cpp

```cpp
#include <algorithm>
#include <vector>

std::vector<int> data{1, 2, 3};
do {
    // Iterate over:
    // 123, 132, 213, 231, 312, 321
} while(std::next_permutation(data.begin(), data.end()));
// data == {1, 2, 3}
```

解释

- **std::next_permutation**:
  - 定义在 <algorithm> 中，用于将当前序列转换为“字典序”（lexicographical order）中的下一个排列。
  - 如果当前序列已经是最后一个排列（如 321），则返回 false，并将序列重置为字典序的第一个排列（如 123）。
  - 要求输入序列是可比较的（默认使用 < 运算符）。
- **代码逻辑**:
  1. 初始化 data 为 {1, 2, 3}，这是字典序中最小的排列。
  2. do-while 循环从当前排列开始，依次生成所有可能的排列，直到没有下一个排列为止。
  3. 每次循环中，std::next_permutation 修改 data 的内容为下一个排列，并返回 true；当到达最后一个排列后，返回 false，循环结束。
- **生成的排列**:
  - {1, 2, 3} → {1, 3, 2} → {2, 1, 3} → {2, 3, 1} → {3, 1, 2} → {3, 2, 1}
  - 总共有 3! = 6 个排列，按字典序递增。
- **循环结束后**:
  - 当 std::next_permutation 返回 false 时，它会将 data 重置为字典序中最小的排列 {1, 2, 3}。
  - 因此，循环结束后 data == {1, 2, 3}。
- **用途**:
  - 生成所有可能的排列，常用于组合数学问题、搜索算法等。

------

2. 第二个例子：使用 std::prev_permutation

cpp

```cpp
#include <vector>
#include <bitset>

std::vector<bool> bits(4);
bits[0] = 1;
bits[1] = 1;
do {
    // Iterate over all 4 bit numbers with 2 bits set to 1
    // 1100, 1010, 1001, 0110, 0101, 0011
} while (std::prev_permutation(bits.begin(), bits.end()));
// bits == {1, 1, 0, 0}
```

解释

- **std::prev_permutation**:
  - 定义在 <algorithm> 中，用于将当前序列转换为字典序中的上一个排列。
  - 如果当前序列已经是字典序中最小的排列（如 0011），则返回 false，并将序列重置为字典序中最大的排列（如 1100）。
  - 与 std::next_permutation 相反，它是逆向遍历排列。
- **代码逻辑**:
  1. 初始化 bits 为一个长度为 4 的 std::vector<bool>，默认值全为 false（即 {0, 0, 0, 0}）。
  2. 手动设置 bits[0] = 1 和 bits[1] = 1，使初始值为 {1, 1, 0, 0}，这是 4 位中恰好有 2 个 1 的字典序最大排列。
  3. do-while 循环从当前排列开始，依次生成字典序中上一个排列，直到没有更小的排列为止。
  4. 每次循环中，std::prev_permutation 修改 bits 为上一个排列，并返回 true；当到达最小排列后，返回 false，循环结束。
- **生成的排列**:
  - {1, 1, 0, 0} → {1, 0, 1, 0} → {1, 0, 0, 1} → {0, 1, 1, 0} → {0, 1, 0, 1} → {0, 0, 1, 1}
  - 这些是 4 位二进制数中恰好有 2 个 1 的所有可能排列（共 C(4, 2) = 6 个），按字典序从大到小遍历。
  - 用二进制表示为：1100、1010、1001、0110、0101、0011。
- **循环结束后**:
  - 当 std::prev_permutation 返回 false 时，它会将 bits 重置为字典序中最大的排列 {1, 1, 0, 0}。
  - 因此，循环结束后 bits == {1, 1, 0, 0}。
- **为什么用 std::vector<bool>**:
  - std::vector<bool> 是标准库中对布尔值的特化实现，适合表示位序列。
  - 这里模拟了一个 4 位二进制数，初始状态为 1100。
- **用途**:
  - 生成特定模式的所有排列，例如固定数量的 1 和 0 的组合。
  - 常用于位运算、组合计数等问题。

------

关键点对比

| 特性       | std::next_permutation  | std::prev_permutation  |
| ---------- | ---------------------- | ---------------------- |
| 方向       | 从小到大（字典序递增） | 从大到小（字典序递减） |
| 初始要求   | 通常从最小排列开始     | 通常从最大排列开始     |
| 结束时状态 | 重置为最小排列         | 重置为最大排列         |
| 返回值     | 有下一个排列返回 true  | 有上一个排列返回 true  |

------

实际应用

1. **std::next_permutation**:
   - 全排列问题：如列出 {1, 2, 3} 的所有可能顺序。
   - 搜索问题：尝试所有可能的顺序以找到满足条件的解。
2. **std::prev_permutation**:
   - 位组合问题：如生成所有 4 位中 2 个 1 的二进制数。
   - 逆向遍历：当需要从最大排列开始递减时使用。

------

完整示例输出

为了更清晰地理解，我们可以添加打印语句：

std::next_permutation

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> data{1, 2, 3};
    do {
        for (int x : data) std::cout << x << " ";
        std::cout << "\n";
    } while (std::next_permutation(data.begin(), data.end()));
    std::cout << "Final: ";
    for (int x : data) std::cout << x << " ";
    std::cout << "\n";
    return 0;
}
```

**输出**:

```text
1 2 3 
1 3 2 
2 1 3 
2 3 1 
3 1 2 
3 2 1 
Final: 1 2 3 
```

std::prev_permutation

cpp

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<bool> bits(4);
    bits[0] = 1;
    bits[1] = 1;
    do {
        for (bool b : bits) std::cout << b;
        std::cout << "\n";
    } while (std::prev_permutation(bits.begin(), bits.end()));
    std::cout << "Final: ";
    for (bool b : bits) std::cout << b;
    std::cout << "\n";
    return 0;
}
```

**输出**:

```text
1100
1010
1001
0110
0101
0011
Final: 1100
```

------

希望这个解释清楚地解答了你的疑问！如果有更多问题或需要进一步探讨，欢迎继续提问。