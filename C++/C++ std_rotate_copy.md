# std::rotate_copy

The *std::rotate_copy* algorithm will write a rotated version of the input range to the provided output iterator.

A range rotated around a pivot is the subrange *[pivot, end)* followed by *[begin, pivot)*.

The algorithm does have a parallel variant.



```C++
#include <algorithm>
#include <vector>
#include <iostream>
#include <execution>

int main() {
    std::vector<int> data{1,2,3,4,5,6,7,8,9};
    std::vector<int> out;

    // The element pointed to by the pivot will be the new first element
    auto pivot = data.begin() + 2;
    // *pivot == 3

    std::rotate_copy(
        data.begin(), pivot, data.end(),
        std::back_inserter(out));
    // out == {3, 4, 5, 6, 7, 8, 9, 1, 2}

    for (auto v : out)
        std::cout << v << " ";
    std::cout << '\n';

    auto pivot2 = data.begin() + 5;
    // *pivot2 == 6

    // Range version
    std::ranges::rotate_copy(data, pivot2, out.begin());
    // out == {6, 7, 8, 9, 1, 2, 3, 4, 5}

    for (auto v : out)
        std::cout << v << " ";
    std::cout << '\n';

    // Parallel version
    std::rotate_copy(std::execution::par_unseq,
        data.begin(), pivot, data.end(),
        out.begin());
    // out == {3, 4, 5, 6, 7, 8, 9, 1, 2}

    for (auto v : out)
        std::cout << v << " ";
    std::cout << '\n';
}
```

这段代码展示了 C++ 中 std::rotate_copy 和 std::ranges::rotate_copy 的用法，用于对容器中的元素进行旋转并复制到目标容器。代码包含三种不同方式的调用：标准版本、范围版本和并行版本。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <algorithm>：提供 std::rotate_copy 等算法。
   - <vector>：提供 std::vector 容器。
   - <iostream>：用于标准输出。
   - <execution>：C++17 中引入的并行执行策略（如 std::execution::par_unseq）。
2. **三个功能块**：
   - 使用经典的 std::rotate_copy。
   - 使用 C++20 的范围版本 std::ranges::rotate_copy。
   - 使用并行版本的 std::rotate_copy。

------

**第一部分：经典 std::rotate_copy**

cpp

```cpp
std::vector<int> data{1, 2, 3, 4, 5, 6, 7, 8, 9};
std::vector<int> out;

auto pivot = data.begin() + 2; // *pivot == 3
std::rotate_copy(data.begin(), pivot, data.end(), std::back_inserter(out));
// out == {3, 4, 5, 6, 7, 8, 9, 1, 2}

for (auto v : out)
    std::cout << v << " ";
std::cout << '\n';
```

**逐步解释**

1. **初始化**：
   - data 是一个包含 {1, 2, 3, 4, 5, 6, 7, 8, 9} 的向量。
   - out 是一个空的向量，用于存储旋转后的结果。
2. **定义 pivot**：
   - pivot = data.begin() + 2 指向第 3 个元素（索引 2），值为 3。
   - pivot 是旋转的“新起点”，即旋转后它将成为序列的第一个元素。
3. **std::rotate_copy**：
   - 签名：std::rotate_copy(first, middle, last, output)。
     - first：输入范围的起始迭代器（data.begin()）。
     - middle：旋转的“新起点”（pivot）。
     - last：输入范围的结束迭代器（data.end()）。
     - output：输出迭代器（std::back_inserter(out)）。
   - 功能：将 [first, last) 的元素旋转，使得 middle 成为新起点，并将结果复制到 output。
   - 逻辑：
     - 原始序列：{1, 2, 3, 4, 5, 6, 7, 8, 9}。
     - pivot 指向 3，旋转后 3 成为起点，之前的元素 {1, 2} 移到末尾。
     - 结果：{3, 4, 5, 6, 7, 8, 9, 1, 2}。
   - std::back_inserter(out)：动态地将元素插入 out，无需预先分配空间。
4. **输出**：
   - 遍历 out，打印：3 4 5 6 7 8 9 1 2。

------

**第二部分：范围版本 std::ranges::rotate_copy**

cpp

```cpp
auto pivot2 = data.begin() + 5; // *pivot2 == 6
std::ranges::rotate_copy(data, pivot2, out.begin());
// out == {6, 7, 8, 9, 1, 2, 3, 4, 5}

for (auto v : out)
    std::cout << v << " ";
std::cout << '\n';
```

**逐步解释**

1. **定义 pivot2**：
   - pivot2 = data.begin() + 5 指向第 6 个元素（索引 5），值为 6。
2. **std::ranges::rotate_copy**：
   - 签名：std::ranges::rotate_copy(range, middle, output)。
     - range：输入范围（这里是整个 data）。
     - middle：旋转的“新起点”（pivot2）。
     - output：输出迭代器（out.begin()）。
   - 功能：与经典版本类似，但使用 C++20 的范围接口，接受整个容器而非迭代器对。
   - 逻辑：
     - 原始序列：{1, 2, 3, 4, 5, 6, 7, 8, 9}。
     - pivot2 指向 6，旋转后 6 成为起点，之前的元素 {1, 2, 3, 4, 5} 移到末尾。
     - 结果：{6, 7, 8, 9, 1, 2, 3, 4, 5}。
   - 注意：这里直接使用 out.begin()，需要确保 out 已分配足够空间（之前通过 back_inserter 填充后大小为 9）。
3. **输出**：
   - 遍历 out，打印：6 7 8 9 1 2 3 4 5。

------

**第三部分：并行版本 std::rotate_copy**

cpp

```cpp
std::rotate_copy(std::execution::par_unseq,
    data.begin(), pivot, data.end(), out.begin());
// out == {3, 4, 5, 6, 7, 8, 9, 1, 2}

for (auto v : out)
    std::cout << v << " ";
std::cout << '\n';
```

**逐步解释**

1. **std::execution::par_unseq**：
   - C++17 引入的并行执行策略：
     - par：允许并行执行。
     - unseq：允许无序执行（线程内指令可以乱序）。
   - 适合大规模数据处理，但这里的数据较小，可能不会有明显性能提升。
2. **std::rotate_copy 并行版本**：
   - 签名：std::rotate_copy(execution_policy, first, middle, last, output)。
     - 多了一个执行策略参数 std::execution::par_unseq。
   - 功能：与经典版本相同，但可能利用多线程并行处理。
   - 逻辑：
     - pivot 仍指向 3，结果与第一部分相同：{3, 4, 5, 6, 7, 8, 9, 1, 2}。
   - 注意：并行版本需要目标容器有足够空间，这里 out 已分配。
3. **输出**：
   - 遍历 out，打印：3 4 5 6 7 8 9 1 2。

------

**关键技术点**

1. **std::rotate_copy vs std::rotate**：
   - std::rotate 修改原容器，而 std::rotate_copy 只复制结果，不改变原数据。
   - 这里 data 始终保持 {1, 2, 3, 4, 5, 6, 7, 8, 9} 不变。
2. **输出迭代器**：
   - std::back_inserter 动态扩展容器，适合初始为空的 out。
   - out.begin() 需要预分配空间，适合覆盖已有数据。
3. **范围接口**：
   - std::ranges::rotate_copy 是 C++20 的现代化接口，更简洁，直接操作整个容器。
4. **并行执行**：
   - std::execution::par_unseq 提供并行优化，但对于小数据集（如这里 9 个元素），开销可能大于收益。

------

**输出总结**

```text
3 4 5 6 7 8 9 1 2
6 7 8 9 1 2 3 4 5
3 4 5 6 7 8 9 1 2
```

- **第一次**：以 3 为起点旋转。
- **第二次**：以 6 为起点旋转。
- **第三次**：以 3 为起点旋转（并行版本）。

------

**总结**

- **经典版本**：传统迭代器接口，灵活但稍显繁琐。
- **范围版本**：C++20 的现代化接口，更简洁易读。
- **并行版本**：C++17 的并行支持，适合大数据处理。
- 这段代码展示了 C++ 算法库的演进和多样的使用方式。

如果有具体问题（例如并行策略的细节），欢迎进一步提问！