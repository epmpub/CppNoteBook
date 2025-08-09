# std::views::adjacent_transform , std::views::pairwise_transform

以下代码展示了 C++20 Ranges 库中三个视图适配器的用法：std::views::adjacent_transform、std::views::adjacent 结合 std::views::transform，以及 std::views::pairwise_transform。

这些适配器用于对相邻元素执行变换操作，生成新的视图。以下是逐步解释。

------

完整代码

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> data{5, 1, 2, 4, 3};

    auto med3 = [](int a, int b, int c) {
        if (a >= b) {
            if (b >= c) return b;
            if (a >= c) return c;
            return a;      
        } else {
            if (c >= b) return b;
            if (a >= c) return a;
            return c;
        }
    };

    auto medians = data | std::views::adjacent_transform<3>(med3);
    // medians == {2, 2, 3}

    for (auto e : medians)
        std::cout << e << " ";
    std::cout << "\n";    

    auto medians_twostep = data | std::views::adjacent<3> | 
        std::views::transform([&](auto&& e) {
            return std::apply(med3, e);
        });
    // medians_twostep == {2, 2, 3}

    for (auto e : medians_twostep)
        std::cout << e << " ";
    std::cout << "\n";

    auto adjacent_difference = data |
        std::views::pairwise_transform(std::minus<>{});
    // adjacent_difference == {4, -1, -2, 1}
    
    for (auto e : adjacent_difference)
        std::cout << e << " ";
    std::cout << "\n";

    return 0;
}
```

------

逐步解释

**初始化**

cpp

```cpp
std::vector<int> data{5, 1, 2, 4, 3};
```

- **data**：
  - 包含 5 个元素：{5, 1, 2, 4, 3}。

**中值函数**

cpp

```cpp
auto med3 = [](int a, int b, int c) {
    if (a >= b) {
        if (b >= c) return b;
        if (a >= c) return c;
        return a;      
    } else {
        if (c >= b) return b;
        if (a >= c) return a;
        return c;
    }
};
```

- **med3**：
  - 一个 lambda 函数，计算三个整数的中值（中间大小的数）。
  - 逻辑：
    - 如果 a >= b：
      - b >= c：返回 b。
      - a >= c：返回 c。
      - 否则返回 a。
    - 如果 a < b：
      - c >= b：返回 b。
      - a >= c：返回 a。
      - 否则返回 c。
  - 示例：
    - med3(5, 1, 2)：1 < 2 < 5，返回 2。
    - med3(1, 2, 4)：1 < 2 < 4，返回 2。
    - med3(2, 4, 3)：2 < 3 < 4，返回 3。

------

**部分 1：std::views::adjacent_transform<3>**

cpp

```cpp
auto medians = data | std::views::adjacent_transform<3>(med3);
// medians == {2, 2, 3}
```

- **std::views::adjacent_transform<N>**：

  - 对范围中每组连续 N 个元素应用变换函数，生成新视图。
  - 视图大小为 data.size() - N + 1（5 - 3 + 1 = 3）。

- **过程**：

  - data = {5, 1, 2, 4, 3}：
    - [0, 2]: med3(5, 1, 2) = 2。
    - [1, 3]: med3(1, 2, 4) = 2。
    - [2, 4]: med3(2, 4, 3) = 3。

- **结果**：

  - medians = {2, 2, 3}。

- **输出**：

  ```text
  2 2 3
  ```

------

**部分 2：std::views::adjacent<3> + std::views::transform**

cpp

```cpp
auto medians_twostep = data | std::views::adjacent<3> | 
    std::views::transform([&](auto&& e) {
        return std::apply(med3, e);
    });
// medians_twostep == {2, 2, 3}
```

- **std::views::adjacent<N>**：

  - 生成每组连续 N 个元素的元组视图。
  - 视图元素为 std::tuple<int, int, int>。
  - 大小同上：3。

- **过程**：

  - data = {5, 1, 2, 4, 3}：
    - [0, 2]: (5, 1, 2)。
    - [1, 3]: (1, 2, 4)。
    - [2, 4]: (2, 4, 3)。

- **std::views::transform**：

  - 对每个元组应用变换函数。
  - std::apply(med3, e)：将元组解包并调用 med3。
    - (5, 1, 2) → med3(5, 1, 2) = 2 Islamabad
    - (1, 2, 4) → med3(1, 2, 4) = 2。
    - (2, 4, 3) → med3(2, 4, 3) = 3。

- **结果**：

  - medians_twostep = {2, 2, 3}。

- **输出**：

  ```text
  2 2 3
  ```

- **与 adjacent_transform 的区别**：

  - adjacent_transform 直接传递元素，避免中间元组。
  - 两步方法更灵活，但稍复杂。

------

**部分 3：std::views::pairwise_transform**

cpp

```cpp
auto adjacent_difference = data |
    std::views::pairwise_transform(std::minus<>{});
// adjacent_difference == {4, -1, -2, 1}
```

- **std::views::pairwise_transform**：

  - 特化为 adjacent_transform<2>，对每对相邻元素应用二元函数。
  - 视图大小为 data.size() - 1（5 - 1 = 4）。

- **std::minus<>**：

  - 二元减法函数：a - b。

- **过程**：

  - data = {5, 1, 2, 4, 3}：
    - [0, 1]: 5 - 1 = 4。
    - [1, 2]: 1 - 2 = -1。
    - [2, 3]: 2 - 4 = -2。
    - [3, 4]: 4 - 3 = 1。

- **结果**：

  - adjacent_difference = {4, -1, -2, 1}。

- **输出**：

  ```text
  4 -1 -2 1
  ```

------

关键点分析

1. **adjacent_transform<N>**：
   - 直接对 N 个相邻元素应用函数，简洁高效。
2. **adjacent<N> + transform**：
   - 分两步，先生成元组，再变换，适合复杂逻辑。
3. **pairwise_transform**：
   - 专为两元素变换设计，模拟相邻差分。
4. **惰性求值**：
   - 所有视图都不修改 data，仅在迭代时计算。

------

时间复杂度

- **构造视图**：O(1)，惰性。
- **迭代**：
  - medians：O(n)，n = 3，每次 med3 是 O(1)。
  - medians_twostep：同上。
  - adjacent_difference：O(n)，n = 4。

------

使用场景

1. **滑动窗口分析**：
   - 计算中值或差分。
2. **数据处理**：
   - 处理相邻元素关系。
3. **算法模拟**：
   - 如差分计算。

------

注意事项

1. **C++20 要求**：
   - 需要 -std=c++20。
2. **范围大小**：
   - 元素数 < N 时，视图为空。
3. **std::apply**：
   - 需要 <tuple> 头文件（未在代码中显示，假设包含）。

------

总结

- **adjacent_transform<3>**：计算三元素中值，{2, 2, 3}。
- **adjacent<3> + transform**：等效实现，{2, 2, 3}。
- **pairwise_transform**：计算相邻差，{4, -1, -2, 1}。 这些视图展示了 C++20 Ranges 的灵活性和简洁性，适合处理相邻元素变换。如果你有具体问题或想扩展代码，请告诉我！