# !!! C++ std::accumulate 和 std::partial_sum 

这段代码展示了 C++ 标准库中 <numeric> 头文件提供的两个函数：std::accumulate 和 std::partial_sum，它们用于对序列进行累积计算（fold 操作）。代码还通过自定义操作和反向迭代展示了左折叠（left fold）和右折叠（right fold）的实现。以下是对代码的逐步解释：

------

1. **std::accumulate**

cpp

```cpp
int res = std::accumulate(data.begin(), data.end(),
    0, // 初始累加器值
    [](int acc, int el) {
        return acc / 2 + el;
    });
// res == 12
```

- **功能**：从左到右对序列进行累积计算，返回最终结果。
- **参数**：
  - data.begin(), data.end()：输入范围 [1, 2, 3, 4, 5, 6, 7]。
  - 0：初始累加器值（acc），也决定了累加器的类型（这里是 int）。
  - Lambda 函数：自定义操作 acc / 2 + el。
- **计算过程**：
  1. acc = 0, el = 1 → 0 / 2 + 1 = 1
  2. acc = 1, el = 2 → 1 / 2 + 2 = 2
  3. acc = 2, el = 3 → 2 / 2 + 3 = 4
  4. acc = 4, el = 4 → 4 / 2 + 4 = 6
  5. acc = 6, el = 5 → 6 / 2 + 5 = 8
  6. acc = 8, el = 6 → 8 / 2 + 6 = 10
  7. acc = 10, el = 7 → 10 / 2 + 7 = 12
- **结果**：res = 12。
- **特点**：只返回最终结果，不保留中间状态。

------

2. **std::partial_sum（左折叠）**

cpp

```cpp
std::vector<int> left_fold;
std::partial_sum(data.begin(), data.end(),
    std::back_inserter(left_fold),
    [](int acc, int el) {
        return acc / 2 + el;
    });
// left_fold == {1, 2, 4, 6, 8, 10, 12}
```

- **功能**：从左到右计算部分和（partial sums），将每个中间结果存储到输出范围。
- **参数**：
  - data.begin(), data.end()：输入范围 [1, 2, 3, 4, 5, 6, 7]。
  - std::back_inserter(left_fold)：输出迭代器，将结果追加到 left_fold。
  - Lambda 函数：自定义操作 acc / 2 + el。
- **计算过程**：
  - 初始值是第一个元素 1，直接输出。
  - 然后依次计算：
    1. acc = 1, el = 2 → 1 / 2 + 2 = 2
    2. acc = 2, el = 3 → 2 / 2 + 3 = 4
    3. acc = 4, el = 4 → 4 / 2 + 4 = 6
    4. acc = 6, el = 5 → 6 / 2 + 5 = 8
    5. acc = 8, el = 6 → 8 / 2 + 6 = 10
    6. acc = 10, el = 7 → 10 / 2 + 7 = 12
- **结果**：left_fold = {1, 2, 4, 6, 8, 10, 12}。
- **特点**：
  - 初始累加器值是输入的第一个元素。
  - 输出包含所有中间结果，形成左折叠。

------

3. **std::partial_sum（单元素情况）**

cpp

```cpp
std::vector<int> single{1};
std::vector<int> out;
std::partial_sum(single.begin(), single.end(),
    std::back_inserter(out),
    std::plus<>{});
// out == {1}
```

- **功能**：验证只有一个元素时的行为。
- **参数**：
  - single.begin(), single.end()：输入范围 [1]。
  - std::plus<>{}：默认加法操作。
- **计算过程**：
  - 只有一个元素 1，直接作为初始值输出。
- **结果**：out = {1}。
- **特点**：std::partial_sum 的输出总是至少包含第一个元素。

------

4. **std::partial_sum（右折叠）**

cpp

```cpp
std::vector<int> right_fold;
std::partial_sum(data.rbegin(), data.rend(),
    std::back_inserter(right_fold),
    [](int acc, int el) {
        return acc / 2 + el;
    });
// right_fold == {7, 9, 9, 8, 7, 5, 3}
```

- **功能**：通过反向迭代实现右折叠。
- **参数**：
  - data.rbegin(), data.rend()：反向范围 [7, 6, 5, 4, 3, 2, 1]。
  - std::back_inserter(right_fold)：输出迭代器。
  - Lambda 函数：acc / 2 + el。
- **计算过程**：
  - 初始值是第一个元素 7（反向的第一个）。
  - 然后依次计算：
    1. acc = 7, el = 6 → 7 / 2 + 6 = 9
    2. acc = 9, el = 5 → 9 / 2 + 5 = 9
    3. acc = 9, el = 4 → 9 / 2 + 4 = 8
    4. acc = 8, el = 3 → 8 / 2 + 3 = 7
    5. acc = 7, el = 2 → 7 / 2 + 2 = 5
    6. acc = 5, el = 1 → 5 / 2 + 1 = 3
- **结果**：right_fold = {7, 9, 9, 8, 7, 5, 3}。
- **特点**：
  - 使用反向迭代器（rbegin(), rend()）实现从右到左的折叠。
  - 需要双向迭代器支持（如 std::vector 提供）。

------

关键区别

| 特性   | std::accumulate    | std::partial_sum                 |
| ------ | ------------------ | -------------------------------- |
| 输出   | 单一最终结果       | 所有中间结果                     |
| 初始值 | 由用户指定（如 0） | 输入的第一个元素                 |
| 方向   | 总是左折叠         | 默认左折叠，可通过反向迭代右折叠 |
| 用途   | 计算总和或单一值   | 生成累积序列                     |

------

总结

- **std::accumulate**：
  - 从左到右累积计算，返回最终结果。
  - 初始值由用户提供，灵活性高。
- **std::partial_sum**：
  - 从左到右计算部分和，输出所有中间结果。
  - 初始值是第一个元素，适合生成累积序列。
- **右折叠**：
  - 通过反向迭代器（如 rbegin(), rend()）实现。
- **自定义操作**：
  - 两个函数都支持自定义二元操作（如 acc / 2 + el），扩展了应用场景。

这段代码清晰地展示了如何使用这两个函数实现折叠操作，并通过例子说明了它们的计算过程和结果。