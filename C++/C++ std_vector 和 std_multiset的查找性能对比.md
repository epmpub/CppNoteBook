# std::vector 和 std::multiset的查找性能对比,随机填充



```cpp
#include <algorithm>
#include <vector>
#include <set>
#include <numeric>
#include <random>
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;
    {
    std::cout << "std::vector\n";
    std::mt19937 rnd(0);
    std::vector<unsigned long> data; // 我们也可以预分配空间
    
    auto t1 = high_resolution_clock::now();
    // 用随机数据填充并排序
    std::ranges::generate_n(std::back_inserter(data), 
        64*1024, [&rnd]() { return rnd(); });
    std::ranges::sort(data);

    auto t2 = high_resolution_clock::now();
    int miss = 0;
    for (auto i = 0; i < 10'000; i++) // 查找10,000个元素
        if (std::ranges::lower_bound(data, rnd()) == data.end())
            ++miss;
    // 统计未命中次数以防止优化器移除此调用
    
    auto t3 = high_resolution_clock::now();
    std::cout << "初始化 " << duration_cast<microseconds>(t2 - t1)
        << " 运行时 " << duration_cast<microseconds>(t3 - t2) 
        << " 总计 " << duration_cast<microseconds>(t3-t1) << "\n";

    std::cout << "未命中次数: " << miss << "\n";
    }
    std::cout << "\n";

    {
    std::cout << "std::multiset\n";
    std::mt19937 rnd(0);
    std::multiset<unsigned long> data;
    
    auto t1 = high_resolution_clock::now();
    // 用随机（但与上面相同）数据填充，无需手动排序
    std::ranges::generate_n(std::inserter(data, data.end()), 
        64*1024, [&rnd]() { return rnd(); });

    auto t2 = high_resolution_clock::now();
    int miss = 0;
    for (auto i = 0; i < 10'000; i++)	// 查找10,000个元素
        if (data.lower_bound(rnd()) == data.end())
            ++miss;
    // 统计未命中次数以防止优化器移除此调用
    
    auto t3 = high_resolution_clock::now();
    std::cout << "初始化 " << duration_cast<microseconds>(t2 - t1)
        << " 运行时 " << duration_cast<microseconds>(t3 - t2)
        << " 总计 " << duration_cast<microseconds>(t3-t1) << "\n";

    std::cout << "未命中次数: " << miss << "\n";
    }
}
```

------

**解释**

**概述**

这个C++程序比较了两种数据结构（std::vector 和 std::multiset）在执行以下任务时的性能：

1. 用随机数据初始化。
2. 查找元素。

程序：

- 使用 std::vector 和 std::multiset 存储 64 * 1024（65,536）个随机无符号长整数。
- 测量以下操作的时间：
  - 用随机数初始化数据结构。
  - 执行 10,000 次查找（使用类似二分查找的 lower_bound）。
- 输出初始化时间、查找时间和总时间（单位为微秒），以及“未命中”（未找到的值）的次数。

代码使用梅森旋转算法随机数生成器（std::mt19937），种子设为 0 以确保结果可重现，并使用 std::chrono 进行高精度计时。

------

**代码分解**

**1. std::vector 部分**

cpp

```cpp
std::cout << "std::vector\n";
std::mt19937 rnd(0);
std::vector<unsigned long> data;
```

- 创建一个 std::vector 来存储数据。它会动态调整大小（这里未预分配空间，尽管注释中提到可以这样做）。
- rnd 是随机数生成器，种子为 0。

cpp

```cpp
auto t1 = high_resolution_clock::now();
std::ranges::generate_n(std::back_inserter(data), 64*1024, [&rnd]() { return rnd(); });
std::ranges::sort(data);
auto t2 = high_resolution_clock::now();
```

- **初始化**：std::ranges::generate_n 用 65,536 个随机数填充向量，back_inserter 将元素追加到向量中。
- **排序**：由于 std::vector 不维护顺序，调用 std::ranges::sort 对数据排序，以便后续高效二分查找。
- t1 到 t2 的时间测量了初始化和排序。

cpp

```cpp
int miss = 0;
for (auto i = 0; i < 10'000; i++)
    if (std::ranges::lower_bound(data, rnd()) == data.end())
        ++miss;
auto t3 = high_resolution_clock::now();
```

- **查找**：生成 10,000 个随机数，std::ranges::lower_bound 执行二分查找以找到第一个不小于随机值的元素。
- 如果返回 data.end()，表示值不在向量中（未命中），miss 增加。
- t2 到 t3 的时间测量查找阶段。
- miss 计数器防止编译器优化掉循环。

cpp

```cpp
std::cout << "初始化 " << duration_cast<microseconds>(t2 - t1)
    << " 运行时 " << duration_cast<microseconds>(t3 - t2) 
    << " 总计 " << duration_cast<microseconds>(t3-t1) << "\n";
std::cout << "未命中次数: " << miss << "\n";
```

- 输出初始化时间、查找时间和总时间（单位为微秒），以及未命中次数。

------

**2. std::multiset 部分**

cpp

```cpp
std::cout << "std::multiset\n";
std::mt19937 rnd(0);
std::multiset<unsigned long> data;
```

- 创建一个 std::multiset，它会自动保持元素有序并允许重复。
- 随机数生成器以相同的种子（0）重置以保持一致性。

cpp

```cpp
auto t1 = high_resolution_clock::now();
std::ranges::generate_n(std::inserter(data, data.end()), 64*1024, [&rnd]() { return rnd(); });
auto t2 = high_resolution_clock::now();
```

- **初始化**：用 65,536 个随机数填充多重集，std::ranges::generate_n 和 inserter 用于插入。插入时自动排序。
- 无需显式排序（与 std::vector 不同）。
- t1 到 t2 的时间测量初始化。

cpp

```cpp
int miss = 0;
for (auto i = 0; i < 10'000; i++)
    if (data.lower_bound(rnd()) == data.end())
        ++miss;
auto t3 = high_resolution_clock::now();
```

- **查找**：与 std::vector 相同，查找 10,000 个随机数，data.lower_bound 利用多重集的内部排序（平衡二叉搜索树）。
- 未命中时增加 miss。
- t2 到 t3 的时间测量查找。

cpp

```cpp
std::cout << "初始化 " << duration_cast<microseconds>(t2 - t1)
    << " 运行时 " << duration_cast<microseconds>(t3 - t2)
    << " 总计 " << duration_cast<microseconds>(t3-t1) << "\n";
std::cout << "未命中次数: " << miss << "\n";
```

- 输出与 std::vector 部分相同的指标。

------

**主要区别**

1. **std::vector**：
   - **优点**：排序后使用 lower_bound 查找很快（O(log n)，二分查找）。
   - **缺点**：需要显式排序（O(n log n)），插入为 O(1)（均摊），但调整大小可能变慢。
   - 内存效率高（连续存储）。
2. **std::multiset**：
   - **优点**：自动保持排序，无需单独排序步骤。插入和查找均为 O(log n)。
   - **缺点**：内存开销较高（树结构存储指针），插入比 std::vector 慢。

------

**预期输出**

输出示例（时间因系统而异）：

```text
std::vector
初始化 5000 us 运行时 200 us 总计 5200 us
未命中次数: 1234

std::multiset
初始化 6000 us 运行时 300 us 总计 6300 us
未命中次数: 1234
```

- **初始化**：std::vector 填充快但因排序整体较慢；std::multiset 初始化较慢，因每次插入为 O(log n)。
- **运行时**：std::vector 查找更快（连续内存二分查找）；std::multiset 稍慢（树遍历）。
- **总计**：std::multiset 通常总时间更高，因插入较慢。
- **未命中次数**：因随机数和种子相同，结果一致。

------

**目的**

此代码用于比较：

- 手动排序的 std::vector（查找快，设置慢）。
- 自动排序的 std::multiset（设置和查找时间平衡）。 有助于根据工作负载（例如大量插入 vs. 大量查找）选择合适的数据结构。

##  

------



## high_resolution_clock::now() 和 duration_cast<microseconds>(t3 - t1) 的含义

让我为你解释 high_resolution_clock::now() 和 duration_cast<microseconds>(t3 - t1) 的含义和用法，这些是 C++ 中 chrono 库的一部分，用于高精度时间测量。

------

**1. high_resolution_clock::now()**

- **定义**: std::chrono::high_resolution_clock::now() 是 C++ 标准库 <chrono> 中的一个静态函数，用于获取当前时间点。
- **返回类型**: 返回一个 std::chrono::time_point 对象，表示调用时的当前时刻。
- **特点**:
  - high_resolution_clock 是 C++ 提供的高精度时钟，通常具有最高的可用分辨率（可能是纳秒级，具体取决于系统）。
  - 它常用于性能测量，因为它能捕捉非常短的时间间隔。
- **用途**: 在代码中，auto t1 = high_resolution_clock::now(); 记录一个时间点（如操作开始时），之后可以用另一个时间点（如 t2）减去它来计算时间差。

**示例**:

cpp

```cpp
auto t1 = std::chrono::high_resolution_clock::now();
// 一些操作
auto t2 = std::chrono::high_resolution_clock::now();
```

t1 和 t2 是两个时间点，之间的差值可以用来计算操作耗时。

------

**2. duration_cast<microseconds>(t3 - t1)**

- **定义**: duration_cast 是 <chrono> 中的一个模板函数，用于将时间间隔（duration）转换为指定单位（如微秒）。
- **输入**: t3 - t1 是两个 time_point 对象相减的结果，类型为 std::chrono::duration，表示从 t1 到 t3 的时间差。
- **模板参数 <microseconds>**: 指定目标时间单位为微秒（std::chrono::microseconds）。
- **返回类型**: 返回一个 std::chrono::microseconds 对象，表示以微秒为单位的时间间隔。
- **作用**: 将原始时间差（可能是纳秒或其他单位）转换为微秒，并丢弃更小单位的分辨率（向下取整）。

**计算过程**:

1. t3 - t1: 计算两个时间点之间的差，返回一个 duration 对象（具体精度取决于 high_resolution_clock）。
2. duration_cast<microseconds>(...): 将这个时间差转换为微秒单位的整数。

**示例**:

cpp

```cpp
auto t1 = std::chrono::high_resolution_clock::now();
// 假设经过 1.234567 秒
auto t3 = std::chrono::high_resolution_clock::now();
auto duration = std::chrono::duration_cast<std::chrono::microseconds>(t3 - t1);
std::cout << duration.count() << " us\n"; // 输出 1234567 us
```

- duration.count() 返回微秒数的整数值（这里是 1,234,567 微秒）。

------

**结合使用**

在你的代码中，这两个函数一起用于测量代码段的执行时间：

cpp

```cpp
auto t1 = std::chrono::high_resolution_clock::now();
// 操作1：初始化数据
auto t2 = std::chrono::high_resolution_clock::now();
// 操作2：查找数据
auto t3 = std::chrono::high_resolution_clock::now();

std::cout << "初始化 " << std::chrono::duration_cast<std::chrono::microseconds>(t2 - t1)
          << " 运行时 " << std::chrono::duration_cast<std::chrono::microseconds>(t3 - t2)
          << " 总计 " << std::chrono::duration_cast<std::chrono::microseconds>(t3 - t1) << "\n";
```

- t2 - t1: 测量初始化时间。
- t3 - t2: 测量查找时间。
- t3 - t1: 测量总时间。
- duration_cast<microseconds>: 将这些时间差转换为微秒并输出。

------

**中文解释**

- **high_resolution_clock::now()**: 获取当前时间点，像一个高精度秒表，用于记录操作开始或结束的时刻。
- **duration_cast<microseconds>(t3 - t1)**: 计算从 t1 到 t3 的时间差，并将其转换为微秒单位。它就像把秒表读数从“原始刻度”换算成“微秒”。

**实际意义**:

- 在你的程序中，这两个工具用来比较 std::vector 和 std::multiset 的性能。
- 比如，t3 - t1 表示从开始到结束的总耗时，duration_cast<microseconds> 确保结果以微秒为单位显示，便于阅读和比较。

------

**注意事项**

1. **精度**: high_resolution_clock 的实际分辨率取决于硬件和操作系统，可能达不到真正的纳秒级，但通常足够用于性能测试。
2. **单位选择**: 如果需要更高精度，可以用 nanoseconds 代替 microseconds；如果需要更粗略的时间，可以用 milliseconds 或 seconds。
3. **输出**: duration_cast 的结果是一个 duration 对象，要用 .count() 获取整数值才能打印。

希望这对你理解代码中的时间测量部分有帮助！有什么疑问可以继续问我。



#  std::mt19937 rnd(0) 的含义和作用

让我为你解释 std::mt19937 rnd(0) 的含义和作用。这是 C++ 标准库 <random> 中的一部分，用于生成高质量的伪随机数。

------

**分解解释**

**1. std::mt19937**

- **定义**: std::mt19937 是 C++ 标准库中的一种随机数生成器类，基于 **Mersenne Twister（梅森旋转算法）**。

- **特点**:

  - 它是一个 **伪随机数生成器（PRNG）**，通过数学算法生成看似随机的数字序列。

  - “19937”指的是它的周期长度为 

    `2^{19937} - 1`

    ，这是一个非常大的数，意味着生成的随机数序列在重复之前非常长。

  - 相比简单的随机数生成器（如 rand()），std::mt19937 提供了更高的随机性质量和更长的周期，适合需要高质量随机数的场景（如模拟、统计分析或性能测试）。

- **头文件**: 定义在 <random> 中。

**2. rnd(0)**

- **定义**: std::mt19937 rnd(0) 创建了一个名为 rnd 的 std::mt19937 对象，并用种子值 0 初始化它。
- **种子（Seed）**:
  - 随机数生成器需要一个初始值（种子）来开始生成序列。
  - 使用相同的种子（如这里的 0），每次运行程序时，rnd 生成的随机数序列将完全相同。这种可重现性在调试或性能测试中非常有用。
  - 如果不提供种子，可以用当前时间（如 std::chrono::system_clock::now()）来初始化，以获得不同的序列。

**完整含义**

- std::mt19937 rnd(0): 创建一个梅森旋转随机数生成器对象 rnd，并用种子 0 初始化它。之后，调用 rnd() 将生成一个伪随机无符号整数。

------

**用法示例**

在你的代码中，std::mt19937 rnd(0) 用于生成随机数来填充数据结构并进行查找测试：

cpp

```cpp
std::mt19937 rnd(0);
std::vector<unsigned long> data;
std::ranges::generate_n(std::back_inserter(data), 64*1024, [&rnd]() { return rnd(); });
```

- **rnd()**: 每次调用返回一个随机 unsigned int（32位版本）或 unsigned long（64位版本，取决于编译器）。
-  **[&rnd]() { return rnd(); }**: 一个 lambda 表达式，捕获 rnd 并调用它生成随机数。
- **种子为 0**: 确保每次运行程序时，生成的 65,536 个随机数序列相同，便于比较 std::vector 和 std::multiset 的性能。

------

**中文解释**

- **std::mt19937**: 一种基于梅森旋转算法的高质量随机数生成器，生成随机数的周期非常长，随机性很好。
- **rnd(0)**: 用种子 0 初始化一个随机数生成器对象 rnd，保证每次运行程序时生成的随机数序列一致。
- **作用**: 在你的程序中，rnd 用于生成随机数据来测试数据结构的性能。因为种子是固定的（0），std::vector 和 std::multiset 接收到的随机数完全相同，确保性能比较的公平性。

------

**扩展知识**

1. **为什么不用 rand()?**

   - 老式的 rand()（在 <cstdlib> 中）质量较低，周期短（通常是 `2^{15}`或`2^{31}`），

     不适合现代应用。

   - std::mt19937 是现代 C++ 的推荐选择。

2. **32位 vs 64位**:

   - std::mt19937 是 32 位版本，生成的随机数范围是 [0, 2^32-1]。
   - 如果需要 64 位随机数，可以用 std::mt19937_64，范围是 [0, 2^64-1]。

3. **改变种子**:

   - 如果想要不同的随机序列，可以用时间作为种子：

     cpp

     ```cpp
     std::mt19937 rnd(std::chrono::system_clock::now().time_since_epoch().count());
     ```

4. **调用方式**:

   - 直接调用 rnd() 获取一个随机数。

   - 如果需要特定范围的数，可以搭配 std::uniform_int_distribution：

     cpp

     ```cpp
     std::uniform_int_distribution<int> dist(1, 100); // 范围 [1, 100]
     int random_num = dist(rnd); // 生成 1 到 100 之间的随机数
     ```

------

**在你的代码中的意义**

- std::mt19937 rnd(0) 确保 std::vector 和 std::multiset 部分的随机数生成完全一致。
- 例如，rnd() 第一次调用可能生成 123456789，第二次调用生成 987654321，等等。因为种子是 0，这些值在两次测试中相同，保证公平性。
- 它为 65,536 个随机数填充和 10,000 次查找提供了数据来源。

希望这个解释清楚地解答了你的疑问！有什么进一步的问题欢迎继续问我。