# std::latch

代码:

```C++
#include <latch>
#include <thread>
#include <vector>
#include <algorithm>
#include <random>
#include <array>
#include <ranges>
#include <iostream>

int main() {
    // Simple and inefficient parallel sum
    constexpr uint64_t chunks = 4;
    constexpr uint64_t total = 1024*1024;
    constexpr uint64_t chunk_size = total/chunks;

    // Random, but consistent input
    std::vector<uint32_t> data;
    std::mt19937 rng(1);
    std::generate_n(std::back_inserter(data), total, std::ref(rng));

    std::latch parallel_done(chunks);
    std::array<uint64_t, chunks> sums;
    std::array<std::jthread, chunks> executors;

    for (auto [chunk, result, exec] : 
        std::views::zip(data | std::views::chunk(chunk_size), sums, executors)) {
        // Start a thread for each chunk, storing the result in the corresponding slot
        exec = std::jthread([&, chunk] {
            result = std::ranges::fold_left(chunk, 0uz, std::plus<uint64_t>{});
            // Atomically decrement the latch
            parallel_done.count_down();
        });
    }
    // Block until all threads have produced a sum
    parallel_done.wait();

    // Final reduction, sum up the partial sums
    uint64_t sum = std::ranges::fold_left(sums, 0uz, std::plus<>{});

    std::cout << "sum == " << sum << "\n";
}
```

这段代码实现了一个简单的并行求和算法，使用 C++20 的特性（如 <latch>、std::jthread、Ranges 库）将一个大向量分成多个块（chunks），在多个线程中并行计算部分和，最后合并结果。以下是逐步解释。

------

代码概览

- 生成一个随机填充的向量 data。
- 使用 std::latch 同步多个线程，计算每个块的部分和。
- 通过 Ranges 库和 std::jthread 并行处理，最后合并部分和。

------

关键组件

1. **头文件**

cpp

```cpp
#include <latch>
#include <thread>
#include <vector>
#include <algorithm>
#include <random>
#include <array>
#include <ranges>
```

- <latch>：提供 std::latch，用于一次性同步。
- <thread>：提供 std::jthread（C++20）。
- <vector>：提供 std::vector。
- <algorithm>：提供 std::generate_n。
- <random>：提供 std::mt19937（随机数生成器）。
- <array>：提供 std::array。
- <ranges>：提供 Ranges 功能（如 std::views::chunk、fold_left）。
- **常量定义**

cpp

```cpp
constexpr uint64_t chunks = 4;
constexpr uint64_t total = 1024*1024;
constexpr uint64_t chunk_size = total/chunks;
```

- chunks：分成 4 个块。
- total：向量大小为 1,048,576（2^20）。
- chunk_size：每个块大小为 262,144（total / chunks）。
- **数据生成**

cpp

```cpp
std::vector<uint32_t> data;
std::mt19937 rng(1);
std::generate_n(std::back_inserter(data), total, std::ref(rng));
```

- **data**：
  - 初始为空向量。
- **rng**：
  - Mersenne Twister 随机数生成器，种子为 1（确保一致性）。
- **std::generate_n**：
  - 生成 total 个随机数，通过 std::back_inserter 追加到 data。
  - std::ref(rng) 确保传递引用，避免拷贝。
- **结果**：
  - data 包含 1,048,576 个随机 uint32_t 值。
- **同步和存储**

cpp

```cpp
std::latch parallel_done(chunks);
std::array<uint64_t, chunks> sums;
std::array<std::jthread, chunks> executors;
```

- **std::latch parallel_done(chunks)**：
  - 初始化计数为 4，线程完成时递减，计数为 0 时解除阻塞。
- **sums**：
  - 存储 4 个部分和，结果为 uint64_t（避免溢出）。
- **executors**：
  - 存储 4 个 std::jthread，自动管理线程生命周期。
- **并行计算**

cpp

```cpp
for (auto [chunk, result, exec] : 
    std::views::zip(data | std::views::chunk(chunk_size), 
                    sums, executors)) {
    exec = std::jthread([&, chunk] {
        result = std::ranges::fold_left(
          chunk, 0uz, std::plus<uint64_t>{});
        parallel_done.count_down();
    });
}
```

- **std::views::chunk(chunk_size)**：
  - 将 data 分成 4 个大小为 chunk_size 的块。
- **std::views::zip**：
  - 将 chunk（数据块）、result（sums 中的元素）、exec（executors 中的线程）配对。
- **线程任务**：
  - exec = std::jthread(...)：
    - 为每个块启动线程。
  - Lambda 参数：
    - [&, chunk]：捕获所有外部变量（按引用），chunk 按值捕获。
  - **std::ranges::fold_left**：
    - 对 chunk 累加，初始值 0uz（无符号零），使用 std::plus<uint64_t>。
    - 计算部分和，存入 result。
  - **parallel_done.count_down()**：
    - 原子递减 latch 计数，标记线程完成。
- **等待完成**

cpp

```cpp
parallel_done.wait();
```

- **wait()**：
  - 阻塞主线程，直到 latch 计数为 0（所有线程完成）。
- **最终合并**

cpp

```cpp
uint64_t sum = std::ranges::fold_left(sums, 0uz, std::plus<>{});
```

- **std::ranges::fold_left**：
  - 将 sums 中的 4 个部分和累加，初始值 0uz。
  - 使用默认 std::plus<>（类型推导为 uint64_t）。
- **结果**：
  - sum 是 data 中所有元素的总和。

------

为什么这样工作？

1. **std::latch**：
   - 提供一次性同步，适合等待所有线程完成。
2. **std::jthread**：
   - 自动加入（join）线程，避免手动管理。
3. **Ranges 库**：
   - chunk 分割数据，zip 配对资源，fold_left 简化累加。
4. **并行性**：
   - 每个线程独立计算部分和，latch 确保同步。

------

输出

- 无显式输出，但 sum 包含最终结果。
- 结果依赖随机数，但种子固定（rng(1)），每次运行一致。

------

使用场景

- **并行计算**：
  - 分担大数组的处理任务。
- **同步**：
  - 使用 latch 等待多个任务完成。
- **数据处理**：
  - Ranges 库简化数据分割和累加。

------

总结

- 代码将 1M 个随机数分成 4 块并行求和。
- 使用 std::latch 同步，std::jthread 管理线程。
- Ranges 库（chunk、zip、fold_left）提供现代化实现。
- 最终 sum 是所有元素之和，展示了简单并行算法的实现。