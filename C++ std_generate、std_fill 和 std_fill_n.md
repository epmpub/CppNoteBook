

# std::generate、std::fill 和 std::fill_n

```C++
#include <algorithm>
#include <vector>
#include <print>

int main() {
    std::vector<int> data(10);

    // Order of invocation is guaranteed
    std::generate(data.begin(), data.end(),
        [iota = 1] mutable {
            return iota++;
        });
    // data == {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    std::println("data == {}", data);

    std::fill(data.begin(), data.end(), 42);
    // data == {42, 42, 42, 42, 42, 42, 42, 42, 42, 42}
    std::println("data == {}", data);

    std::vector<int> empty;

    // Both algorithms have a counted variant that takes 
    // an iterator and the number of elements.
    std::fill_n(std::back_inserter(empty), 5, 7);
    // empty == {7, 7, 7, 7, 7}
    std::println("empty == {}", empty);
}
```



这段代码展示了 C++ 标准库中 <algorithm> 提供的三个算法：std::generate、std::fill 和 std::fill_n，用于操作容器中的元素。代码通过示例展示了它们的用法和特性。以下是逐步解释。

------

代码概览

- 使用 std::generate 生成递增序列。
- 使用 std::fill 填充固定值。
- 使用 std::fill_n 向空容器追加指定数量的元素。

------

关键组件

1. **头文件**

cpp

```cpp
#include <algorithm>
#include <vector>
```

- <algorithm>：提供 std::generate、std::fill 和 std::fill_n。
- <vector>：提供 std::vector 和 std::back_inserter。
- **std::generate 示例**

cpp

```cpp
std::vector<int> data(10);

std::generate(data.begin(), data.end(),
    [iota = 1] mutable {
        return iota++;
    });
// data == {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
```

- **data**：
  - 初始化为大小为 10 的向量，默认值 0。
- **std::generate**：
  - 原型：generate(first, last, gen)。
  - 在 [first, last) 范围内用生成器 gen 的返回值填充元素。
  - 保证从左到右顺序调用生成器。
- **生成器**：
  - [iota = 1] mutable { return iota++; }：
    - lambda 捕获变量 iota，初始值为 1。
    - mutable 允许修改捕获的 iota。
    - 每次调用返回当前 iota 并递增。
- **过程**：
  - 第一次：iota = 1，返回 1，iota = 2。
  - 第二次：iota = 2，返回 2，iota = 3。
  - ...
  - 第十次：iota = 10，返回 10，iota = 11。
- **结果**：
  - data = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}。
- **std::fill 示例**

cpp

```cpp
std::fill(data.begin(), data.end(), 42);
// data == {42, 42, 42, 42, 42, 42, 42, 42, 42, 42}
```

- **std::fill**：
  - 原型：fill(first, last, value)。
  - 将 [first, last) 范围内的元素设置为 value。
- **过程**：
  - 将 data 的每个元素设置为 42。
- **结果**：
  - data = {42, 42, 42, 42, 42, 42,edas 42, 42, 42, 42}。
- **注意**：
  - 不保证顺序，但结果一致。
- **std::fill_n 示例**

cpp

```cpp
std::vector<int> empty;

std::fill_n(std::back_inserter(empty), 5, 7);
// empty == {7, 7, 7, 7, 7}
```

- **empty**：
  - 初始为空向量。
- **std::fill_n**：
  - 原型：fill_n(first, n, value)。
  - 从 first 开始，填充 n 个元素为 value。
  - 返回指向第 n+1 个位置的迭代器。
- **std::back_inserter(empty)**：
  - 返回一个插入迭代器，每次赋值调用 empty.push_back。
- **过程**：
  - 填充 5 个 7：
    - empty.push_back(7) 重复 5 次。
- **结果**：
  - empty = {7, 7, 7, 7, 7}。

------

为什么这样工作？

1. **std::generate**：
   - 使用生成器动态填充，保证顺序，便于生成序列。
2. **std::fill**：
   - 简单高效地设置固定值，适用于已有范围。
3. **std::fill_n**：
   - 提供计数版本，配合插入迭代器可扩展容器。
4. **迭代器**：
   - begin/end 指定范围，back_inserter 支持动态增长。

------

输出

- 无运行时输出，代码展示结果：
  - data：{1, 2, 3, 4, 5, 6, 7, 8, 9, 10} → {42, ...}。
  - empty：{7, 7, 7, 7, 7}。

------

使用场景

- **std::generate**：
  - 生成序列（如索引、随机数）。
- **std::fill**：
  - 初始化或重置容器为固定值。
- **std::fill_n**：
  - 向动态容器追加固定数量的元素。

------

总结

- std::generate 使用 lambda 生成递增序列。
- std::fill 将范围填充为固定值。
- std::fill_n 计数填充，支持容器扩展。
- 代码展示了这些算法的灵活性和实用性。