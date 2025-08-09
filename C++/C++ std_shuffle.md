# std::shuffle

std::shuffle 是 C++ 标准库中的一个算法函数，定义在 <algorithm> 头文件中。它在 C++11 中引入，用于将范围内的元素随机重新排列（洗牌）。与旧的 std::random_shuffle（C++98 引入，C++17 移除）相比，std::shuffle 提供了更现代和灵活的随机数生成接口。

以下是对 std::shuffle 的详细解释：

------

定义

cpp

```cpp
#include <algorithm>

template <class RandomIt, class URBG>
void shuffle(RandomIt first, RandomIt last, URBG&& g);
```

- **first, last**: 定义要洗牌的范围 [first, last) 的迭代器对。
- **URBG**: 随机数生成器类型（Uniform Random Bit Generator），通常是 <random> 中的引擎（如 std::mt19937）。
- **返回值**: 无（void）。

------

行为

- std::shuffle 对范围 [first, last) 中的元素进行随机重排。
- 它使用提供的随机数生成器 g 来生成均匀分布的随机索引。
- 每次调用都会产生不同的排列，具体取决于随机数生成器的状态。

------

前提条件

- **迭代器类型**：
  - 必须是随机访问迭代器（RandomAccessIterator），如 std::vector 或数组的迭代器。
  - 不支持前向迭代器（如 std::list）。
- **随机数生成器**：
  - URBG 必须满足均匀随机位生成器要求（提供 operator() 返回随机值，且有 min() 和 max() 成员）。
- **范围有效性**：
  - [first, last) 必须有效。

------

示例代码

示例 1：基本使用

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <random>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 创建随机数生成器
    std::random_device rd; // 真随机种子
    std::mt19937 gen(rd()); // 梅森扭转生成器
    
    //std::mt19937 gen(std::random_device{}()); // 上述写法的简写

    // 洗牌
    std::shuffle(vec.begin(), vec.end(), gen);

    // 输出结果
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出（示例）

```text
3 1 5 2 4
```

- 每次运行结果不同，因为使用了随机种子。

示例 2：固定种子

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <random>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 使用固定种子
    std::mt19937 gen(42); // 固定种子，产生可重复的结果

    std::shuffle(vec.begin(), vec.end(), gen);

    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出（示例）

```text
4 1 5 2 3
```

- 使用固定种子，每次运行结果相同。

------

时间复杂度

- **O(n)**：
  - 其中 n = last - first。
  - 使用 Fisher-Yates（或 Knuth）洗牌算法，保证均匀随机排列。
- 每次迭代生成一个随机索引并交换元素，总共 n - 1 次操作。

------

与 std::random_shuffle 的区别

| 特性     | std::shuffle         | std::random_shuffle    |
| -------- | -------------------- | ---------------------- |
| 引入版本 | C++11                | C++98                  |
| 移除版本 | 未移除               | C++17                  |
| 随机源   | 显式传递 URBG        | 默认 rand() 或用户函数 |
| 可控性   | 更高（自定义生成器） | 较低（依赖全局状态）   |
| 安全性   | 更现代、更安全       | 依赖 rand() 不推荐     |

- std::random_shuffle 在 C++14 中标记为弃用，C++17 中移除，推荐使用 std::shuffle。

------

实现原理（概念性）

std::shuffle 通常基于 Fisher-Yates 洗牌算法：

1. 从最后一个元素开始（i = n - 1）。
2. 在 [0, i] 中生成一个随机索引 j。
3. 交换位置 i 和 j 的元素。
4. 递减 i，重复直到 i = 0。

伪代码：

cpp

```cpp
for (i = n - 1; i > 0; --i) {
    j = random(0, i); // 使用 URBG 生成随机索引
    swap(vec[i], vec[j]);
}
```

------

使用场景

1. **随机化数据**：

   - 打乱数组或向量中的元素。

   cpp

   ```cpp
   std::shuffle(vec.begin(), vec.end(), gen);
   ```

2. **模拟和测试**：

   - 生成随机排列用于测试算法。

   cpp

   ```cpp
   std::vector<int> cards(52);
   std::iota(cards.begin(), cards.end(), 1);
   std::shuffle(cards.begin(), cards.end(), gen); // 洗牌
   ```

3. **游戏开发**：

   - 随机化卡牌、关卡顺序等。

------

注意事项

1. **随机数生成器**：
   - 必须显式提供 URBG，如 std::mt19937。
   - 使用 std::random_device 获取真随机种子，避免伪随机重复。
2. **迭代器要求**：
   - 只支持随机访问迭代器，不适用于 std::list 或 std::set。
3. **均匀性**：
   - 依赖随机数生成器的质量，std::mt19937 是推荐选择（比 rand() 更均匀）。
4. **异常**：
   - 如果交换操作抛出异常，std::shuffle 可能部分完成。

------

常见随机数生成器

- **std::mt19937**：梅森扭转算法，高质量伪随机数。
- **std::random_device**：真随机数，用于种子。
- 避免使用 rand()（质量低且已不推荐）。

------

总结

std::shuffle 是 C++11 引入的现代洗牌算法，用于随机重排范围内的元素。它比 std::random_shuffle 更安全、更灵活，要求显式传递随机数生成器，时间复杂度为 O(n)。适用于需要随机化的场景，如测试、游戏或数据处理。如果有具体问题或想探讨用法，欢迎继续提问！