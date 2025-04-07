# *std::reduce* (C++17)

*std::reduce* (C++17) 是一种广义的约简算法，是左折叠*std::accumulate 的*对应算法。

*std::reduce*只能进行结合和交换运算，因为归约不是按照严格的从左到右的顺序进行评估的。

不过，接下来该算法还通过*std::execution 支持并行执行*。

```c++
#include <vector>
#include <algorithm>
#include <execution>
#include <random>
#include <iostream>

int main() {
    std::vector<int> data{1, 2, 3, 4, 5, 6, 7};

    // Basic reduce with int{} (0) as starting value 
    // and std::plus<int>{} as the reduction operation
    int sum = std::reduce(data.begin(), data.end());
    // sum == 28

    std::cout << "sum == " << sum << "\n";

    // Reduce with custom init value and reduction operation
    int product = std::reduce(data.begin(), data.end(),
                            1, std::multiplies<>{});
    // product == 5040

    std::cout << "product == " << product << '\n';

    std::mt19937 gen(0);
    std::vector<unsigned> large_data;
    std::generate_n(std::back_inserter(large_data), 1'000'000,
                    std::ref(gen));

    // Automatically parallelized sum
    int64_t big_sum = std::reduce(std::execution::par,
                                large_data.cbegin(), large_data.cend(), 0z);
    // stdlibc++: big_sum == 2147988759967286

    std::cout << "big_sum == " << big_sum << '\n';
}
```



这段代码展示了 C++17 中引入的 <algorithm> 库中的 std::reduce 函数，用于对容器中的元素进行归约（reduction）操作。它还结合了 C++17 的并行执行策略（<execution>）来加速计算。以下是对代码的逐步解释。

------

代码概览

- 代码使用 std::reduce 对 std::vector 中的元素进行求和和求积操作。
- 展示了三种用法：
  1. 默认求和（使用默认初始值和默认加法操作）。
  2. 自定义初始值和操作（求乘积）。
  3. 并行归约（对大数据集求和）。
- 使用了 <random> 和 <vector> 来生成测试数据。

------

关键组件

1. **头文件和依赖**

cpp

```cpp
#include <vector>
#include <algorithm>
#include <execution>
#include <random>
#include <iostream>
```

- <vector>：提供 std::vector。
- <algorithm>：提供 std::reduce 和 std::generate_n。
- <execution>：提供并行执行策略（如 std::execution::par）。
- <random>：提供随机数生成器 std::mt19937。
- <iostream>：用于输出。
- **基本求和**

cpp

```cpp
std::vector<int> data{1, 2, 3, 4, 5, 6, 7};
int sum = std::reduce(data.begin(), data.end());
// sum == 28
std::cout << "sum == " << sum << "\n";
```

- **std::reduce**：
  - 定义在 <algorithm> 中，类似于 std::accumulate，但支持并行执行，且不保证操作顺序。
  - 基本形式：std::reduce(first, last, init, binary_op)。
  - 如果不提供 init 和 binary_op，默认使用 0 作为初始值和 std::plus<>{}（加法）作为归约操作。
- **这里**：
  - data.begin() 和 data.end() 指定范围。
  - 未提供初始值和操作，因此使用默认值：初始值 0，操作 std::plus<int>{}。
  - 计算：0 + 1 + 2 + 3 + 4 + 5 + 6 + 7 = 28。
- **输出**：sum == 28。
- **自定义初始值和操作（求乘积）**

cpp

```cpp
int product = std::reduce(data.begin(), data.end(), 1, std::multiplies<>{});
// product == 5040
std::cout << "product == " << product << '\n';
```

- **参数**：
  - data.begin(), data.end()：与之前相同，指定范围。
  - 1：自定义初始值（对于乘法，初始值通常是 1，因为 0 会使结果始终为 0）。
  - std::multiplies<>{}：一个函数对象，表示乘法操作。
- **计算**：
  - 1 * 1 * 2 * 3 * 4 * 5 * 6 * 7 = 5040。
- **输出**：product == 5040。
- **注意**：std::multiplies<> 是 <functional> 中的模板，默认推导为 std::multiplies<int>，与 int 类型匹配。
- **生成大数据集**

cpp

```cpp
std::mt19937 gen(0);
std::vector<unsigned> large_data;
std::generate_n(std::back_inserter(large_data), 1'000'000, std::ref(gen));
```

- **std::mt19937 gen(0)**：
  - 创建一个 Mersenne Twister 随机数生成器，种子为 0（固定种子确保结果可重现）。
- **std::generate_n**：
  - 生成 1,000,000 个随机数，填充到 large_data 中。
  - std::back_inserter(large_data)：通过 push_back 向向量追加元素。
  - std::ref(gen)：传递 gen 的引用（避免拷贝），每次调用 gen() 生成一个随机 unsigned int。
- **结果**：large_data 是一个包含 1,000,000 个随机 unsigned int 的向量。
- **并行归约**

cpp

```cpp
int64_t big_sum = std::reduce(std::execution::par, large_data.cbegin(), large_data.cend(), 0z);
// stdlibc++: big_sum == 2147988759967286
std::cout << "big_sum == " << big_sum << '\n';
```

- **std::reduce 的并行版本**：
  - 第一个参数 std::execution::par 表示使用并行执行策略（需要 <execution> 支持）。
  - large_data.cbegin(), large_data.cend()：指定常量迭代器范围。
  - 0z：初始值为 0，后缀 z 表示 std::size_t 类型（这里会被提升为 int64_t）。
  - 默认操作仍是 std::plus<>{}。
- **并行性**：
  - std::reduce 在并行模式下会将数据分成多个块，分别计算部分和，然后合并结果。
  - 这需要操作（如加法）满足交换律和结合律（std::plus<> 满足）。
- **结果**：
  - big_sum 是所有 1,000,000 个随机数的和。
  - 使用 std::mt19937 和种子 0，stdlibc++ 的实现得到 2147988759967286（具体值依赖实现和随机数序列）。
- **输出**：big_sum == 2147988759967286。
- **类型**：使用 int64_t 避免溢出，因为 unsigned int 的和可能超过 32 位范围。

------

为什么这样工作？

1. **std::reduce vs std::accumulate**：
   - std::accumulate 是顺序执行的，保证从左到右的顺序。
   - std::reduce 不保证顺序，适合并行化，因此更快，但要求操作是可交换和可结合的。
2. **并行执行**：
   - std::execution::par 利用多线程（依赖实现，如 TBB 或线程池）。
   - 对于大向量（如 large_data），并行化显著提升性能。
3. **默认值**：
   - 未指定操作时，std::reduce 使用 std::plus<>{}，未指定初始值时使用元素类型的默认值（int 为 0）。

------

注意事项

- **编译器支持**：
  - 需要 C++17 或更高版本，并链接并行库（如 -ltbb 用于 Intel TBB）。
  - 示例中的 std::execution::par 可能在某些实现中不可用，除非启用特定支持。
- **溢出风险**：
  - 对大数据集求和时，使用 int64_t 或更大的类型是必要的。
- **可重现性**：
  - 种子 0 确保 large_data 的内容固定，但 big_sum 的具体值可能因随机数生成器的实现而异。

------

输出

运行代码（假设使用 stdlibc++ 和种子 0）：

```text
sum == 28
product == 5040
big_sum == 2147988759967286
```

------

总结

- std::reduce 是一个强大的工具，支持顺序和并行归约。
- 默认行为是求和，自定义操作（如 std::multiplies<>）扩展了其用途。
- 并行模式（std::execution::par）适用于大数据集，显著提高效率。
- 这段代码展示了 std::reduce 的灵活性和现代 C++ 的特性（如随机数生成和容器操作）。如果需要进一步澄清某个部分，请告诉我！