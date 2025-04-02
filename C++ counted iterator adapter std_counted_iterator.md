

# The C++ counted iterator adapter: std::counted_iterator

Only a few algorithms in the standard library offer counted variants that operate using a begin iterator and the number of elements.

C++20 introduced the std::counted_iterator adapter that can turn any range version of an algorithm into a counted variant.

------

完整代码

cpp

```cpp
std::vector<int> out2;
std::ranges::copy(
    // Adapt iterator, and specify number of elements.
    std::counted_iterator(data.begin(), 5), 
    // counted_iterator == default_sentinel if count is zero
    std::default_sentinel,
    std::back_inserter(out2));
// out2 == {1, 2, 3, 4, 5} 
```

前提假设

- **data** 是一个 std::vector<int>，至少包含 5 个元素，例如 {1, 2, 3, 4, 5, ...}。

- 必要的头文件已包含：

  cpp

  ```cpp
  #include <vector>
  #include <ranges>
  #include <iterator>
  ```

------

逐步解释

1. **目标容器定义**

cpp

```cpp
std::vector<int> out2;
```

- **out2** 是一个空的 std::vector<int>，用于存储复制的结果。
- 初始状态：out2.size() == 0，out2.capacity() 可能为 0（取决于实现）。

------

2. **std::ranges::copy**

cpp

```cpp
std::ranges::copy(
    std::counted_iterator(data.begin(), 5), 
    std::default_sentinel,
    std::back_inserter(out2));
```

- **std::ranges::copy** 是 C++20 Ranges 库中的算法，用于将一个范围的元素复制到目标位置。

- **签名**：

  cpp

  ```cpp
  template <std::input_iterator I, std::sentinel_for<I> S, std::weakly_incrementable O>
  std::ranges::copy_result<I, O> copy(I first, S last, O result);
  ```

  - **I first**: 输入范围的起始迭代器。
  - **S last**: 输入范围的结束标记（sentinel）。
  - **O result**: 输出迭代器。

- **功能**：

  - 从 first 到 last 的元素被复制到 result 指定的位置。

------

3. **输入范围起始：std::counted_iterator**

cpp

```cpp
std::counted_iterator(data.begin(), 5)
```

- **std::counted_iterator** 是 C++20 引入的迭代器适配器，定义在 <iterator> 中。
- **作用**：
  - 包装一个基础迭代器（如 data.begin()），并限制迭代次数。
- **构造**：
  - **data.begin()**：data 的起始迭代器，指向第一个元素（例如 1）。
  - **5**：计数，表示最多迭代 5 次。
- **行为**：
  - 从 data.begin() 开始，允许访问最多 5 个元素。
  - 内部维护一个计数器，每次递增迭代器时计数减 1，当计数为 0 时停止。

------

4. **输入范围结束：std::default_sentinel**

cpp

```cpp
std::default_sentinel
```

- **std::default_sentinel** 是 C++20 引入的通用哨兵类型，定义在 <iterator> 中。
- **作用**：
  - 作为 std::counted_iterator 的结束标记。
  - 当 std::counted_iterator 的计数减到 0 时，它等于 std::default_sentinel，表示范围结束。
- **与 counted_iterator 的配合**：
  - std::counted_iterator 和 std::default_sentinel 一起定义了一个有限范围，无需显式指定结束迭代器（如 data.begin() + 5）。

------

5. **输出迭代器：std::back_inserter**

cpp

```cpp
std::back_inserter(out2)
```

- **std::back_inserter** 是 <iterator> 中的函数，返回一个 std::back_insert_iterator。
- **作用**：
  - 将元素追加到容器（这里是 out2）的末尾。
  - 通过调用 out2.push_back() 实现。
- **行为**：
  - 每次复制一个元素时，out2 会动态增长，将元素插入末尾。
- **优点**：
  - 不需要预先分配空间，适合从空容器开始。

------

6. **执行过程**

假设 data = {1, 2, 3, 4, 5, 6, 7}：

1. **初始状态**：
   - out2 = {}（空）。
   - std::counted_iterator(data.begin(), 5) 指向 1，计数为 5。
2. **copy 操作**：
   - 复制第 1 个元素：1 → out2.push_back(1)，out2 = {1}，计数减到 4。
   - 复制第 2 个元素：2 → out2.push_back(2)，out2 = {1, 2}，计数减到 3。
   - 复制第 3 个元素：3 → out2.push_back(3)，out2 = {1, 2, 3}，计数减到 2。
   - 复制第 4 个元素：4 → out2.push_back(4)，out2 = {1, 2, 3, 4}，计数减到 1。
   - 复制第 5 个元素：5 → out2.push_back(5)，out2 = {1, 2, 3, 4, 5}，计数减到 0。
   - 计数为 0，counted_iterator 等于 default_sentinel，停止复制。
3. **结果**：
   - out2 = {1, 2, 3, 4, 5}。

------

输出结果

cpp

```cpp
// out2 == {1, 2, 3, 4, 5}
```

- **out2** 包含 data 的前 5 个元素。
- 如果 data 少于 5 个元素（例如 {1, 2, 3}），则复制所有元素，out2 = {1, 2, 3}。

------

时间复杂度

- **std::ranges::copy**：O(n)，n 是复制的元素数（这里是 5）。
- **std::back_inserter**：每次插入是 O(1)（均摊），可能触发 vector 扩容（O(n)，但均摊后仍为 O(1)）。
- **总复杂度**：O(n)，这里 n = 5。

------

使用场景

1. **复制部分范围**：
   - 从大容器中提取前 N 个元素。
2. **动态构建容器**：
   - 使用 back_inserter 从空容器开始填充。
3. **Ranges 编程**：
   - 配合其他视图或迭代器适配器处理数据。

------

注意事项

1. **C++20 要求**：
   - 需要 -std=c++20 和支持 C++20 的编译器。
2. **范围安全性**：
   - 如果 data.size() < 5，只复制可用元素，不会越界。
3. **std::counted_iterator 的灵活性**：
   - 计数为 0 时，范围为空，out2 不变。

------

等价传统代码

cpp

```cpp
std::vector<int> out2;
for (int i = 0; i < 5 && i < data.size(); ++i) {
    out2.push_back(data[i]);
}
```

- **std::ranges::copy 版本**更简洁，类型安全，且支持泛型编程。

------

验证代码

cpp

```cpp
#include <vector>
#include <ranges>
#include <iterator>
#include <iostream>

int main() {
    std::vector<int> data = {1, 2, 3, 4, 5, 6, 7};
    std::vector<int> out2;
    std::ranges::copy(
        std::counted_iterator(data.begin(), 5),
        std::default_sentinel,
        std::back_inserter(out2)
    );

    for (int x : out2) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
1 2 3 4 5
```

------

总结

这段代码使用 std::ranges::copy 将 data 的前 5 个元素复制到 out2：

- **std::counted_iterator** 限制复制 5 个元素。
- **std::default_sentinel** 作为结束标记。
- **std::back_inserter** 动态填充 out2。 结果是 out2 = {1, 2, 3, 4, 5}。如果你有具体问题或想扩展用法，请告诉我！