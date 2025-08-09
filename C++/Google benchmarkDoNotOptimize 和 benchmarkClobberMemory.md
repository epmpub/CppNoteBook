# Google benchmark::DoNotOptimize 和 benchmark::ClobberMemory 

```C++
static void BM_DenseRange(benchmark::State& state) {
    for (auto _ : state) {
        std::vector<int> v(state.range(0), state.range(0));
        auto data = v.data();
        benchmark::DoNotOptimize(data);
        benchmark::ClobberMemory();
    }
}
BENCHMARK(BM_DenseRange)->DenseRange(0, 1024, 128);
```



这段代码定义了一个 Google Benchmark 测试函数 BM_DenseRange，用于测量在不同大小的 std::vector<int> 上执行某些操作的性能。测试通过 DenseRange 指定一系列参数，创建并初始化向量，获取其底层数据指针，并使用 benchmark::DoNotOptimize 和 benchmark::ClobberMemory 防止编译器优化影响结果。

以下是对代码的逐行解释、功能分析、设计意图和潜在改进建议。

------

代码逐行解释

cpp

```cpp
static void BM_DenseRange(benchmark::State& state) {
```

- 定义一个静态函数 BM_DenseRange，作为 Google Benchmark 的测试函数。
- 参数 benchmark::State& state 用于控制测试运行、访问参数（通过 state.range(0)）和记录结果。
- 函数名称以 BM_ 开头，符合 Google Benchmark 的命名惯例。

cpp

```cpp
    for (auto _ : state) {
```

- for (auto _ : state) 是 Google Benchmark 的计时循环，代码在此循环内会被重复执行。
- Google Benchmark 自动控制迭代次数以确保性能测量的统计稳定性。
- 变量 _ 是一个占位符，表示不使用循环变量（C++17 语法）。

cpp

```cpp
        std::vector<int> v(state.range(0), state.range(0));
```

- 创建一个 std::vector<int> 对象 v，使用构造函数 std::vector(size_type n, const T& value)。
- 参数：
  - state.range(0)（作为 n）：指定向量的元素数量。
  - state.range(0)（作为 value）：每个元素初始化为 state.range(0) 的值（类型转换为 int）。
- 效果：v 包含 state.range(0) 个元素，每个元素的值为 state.range(0)。
- 例如，如果 state.range(0) = 128：
  - v 是一个包含 128 个元素的向量，每个元素的值为 128（v = {128, 128, ..., 128}）。
- 内存分配：
  - 向量动态分配内存，大小为 state.range(0) * sizeof(int) 字节。
  - 假设 sizeof(int) = 4，则 state.range(0) = 128 时，分配 128 × 4 = 512 字节。

cpp

```cpp
        auto data = v.data();
```

- 调用 std::vector::data()，返回指向向量底层数组的指针。
- data 是一个 int*，指向 v 的第一个元素。
- 目的：可能为后续操作（例如内存访问或复制）准备数据指针，尽管当前代码未直接使用 data。

cpp

```cpp
        benchmark::DoNotOptimize(data);
```

- benchmark::DoNotOptimize 是 Google Benchmark 提供的工具，防止编译器优化掉 data 或相关操作。
- 作用：
  - 确保 v.data() 的调用和 data 的存储不会被编译器优化消除。
  - 告诉编译器 data 的值可能在外部使用，保留相关内存访问。
- 为什么需要？
  - 没有 DoNotOptimize，编译器可能发现 data 未被使用，优化掉 v.data() 的调用甚至整个向量的创建，导致测试结果不反映实际性能。

cpp

```cpp
        benchmark::ClobberMemory();
```

- benchmark::ClobberMemory 是 Google Benchmark 的另一个工具，强制编译器假设所有内存内容可能被修改。
- 作用：
  - 防止编译器基于内存不变的假设优化代码。
  - 模拟真实场景中内存可能被外部代码修改的情况。
- 为什么需要？
  - 确保 v 的内存分配和初始化不会被编译器合并或省略。
  - 保证测试测量的是实际的内存操作成本。

cpp

```cpp
    }
}
```

- 计时循环和函数结束。
- 每次迭代都会创建一个新的 std::vector<int>，初始化其元素，获取数据指针，并执行优化控制操作。
- 向量 v 在每次迭代结束时自动销毁（超出作用域，调用析构函数释放内存）。

cpp

```cpp
BENCHMARK(BM_DenseRange)->DenseRange(0, 1024, 128);
```

- 注册基准测试 BM_DenseRange，使用 DenseRange(start, end, step) 指定参数范围。

- 参数：

  - start = 0：起始参数值。
  - end = 1024：结束参数值（包含）。
  - step = 128：步长。

- 效果：测试将运行以下参数值：

  - ( 0, 128, 256, 384, 512, 640, 768, 896, 1024 )

  - 计算方式：从 0 开始，每次增加 128，直到达到或超过 1024（包含 1024）。

  - 总共 

    `\lfloor (1024 - 0) / 128 \rfloor + 1 = 8 + 1 = 9`

     个测试用例。

- 每个测试用例对应一个 state.range(0) 值，生成形如 BM_DenseRange/0, BM_DenseRange/128, ..., BM_DenseRange/1024 的测试。

------

功能和设计意图

功能

- **测量向量创建和初始化的性能**：
  - 测试创建并初始化一个 std::vector<int>（大小和元素值由 state.range(0) 指定）的性能。
  - 每次迭代包括：
    - 分配内存（std::vector 的动态分配）。
    - 初始化元素（将每个元素设为 state.range(0)）。
    - 获取数据指针（v.data()）。
    - 销毁向量（释放内存）。
- **参数化测试**：
  - 使用 DenseRange(0, 1024, 128) 测试不同向量大小（0 到 1024，步长 128）。
  - 每个大小对应一个测试用例，测量性能随向量大小的变化。

设计意图

- **测试内存分配和初始化**：
  - 主要测量 std::vector 的构造（分配 + 初始化）和析构（释放）的开销。
  - state.range(0) 同时作为大小和初始化值，可能是一个简化选择，模拟某种特定场景（例如，初始化为固定值的向量）。
- **防止编译器优化**：
  - benchmark::DoNotOptimize 和 benchmark::ClobberMemory 确保编译器不会优化掉向量的创建、初始化或内存访问，反映真实性能。
- **探索性能趋势**：
  - DenseRange 允许测试一系列大小，观察性能如何随向量大小变化（例如，缓存效应、分配器行为）。
- **通用框架**：
  - 代码结构可扩展，可以修改为测试其他操作（例如，访问 data、执行计算、或比较不同容器）。

运行示例

假设代码完整如下：

cpp

```cpp
#include <benchmark/benchmark.h>
#include <vector>

static void BM_DenseRange(benchmark::State& state) {
    for (auto _ : state) {
        std::vector<int> v(state.range(0), state.range(0));
        auto data = v.data();
        benchmark::DoNotOptimize(data);
        benchmark::ClobberMemory();
    }
}
BENCHMARK(BM_DenseRange)->DenseRange(0, 1024, 128);
BENCHMARK_MAIN();
```





- **解释**：
  - `BM_DenseRange/0`：空向量（大小 0），时间最短（~10.2 ns），仅涉及构造和析构开销。
  - `BM_DenseRange/128`：128 个元素（512 字节），时间增加到 ~45.6 ns。
  - 时间随大小线性增加，反映分配和初始化的开销。
  - 吞吐量未设置（未调用 `state.SetBytesProcessed`），因此未显示 bytes/s。

---

### 关键点分析

1. **为什么用 `DenseRange`？**
   - `DenseRange(start, end, step)` 生成等间隔的参数序列，适合测试连续的输入范围（这里是 0 到 1024，步长 128）。
   - 相比 `Range`（测试 2 的幂次），`DenseRange` 提供更细粒度的控制，适合分析性能随大小的平滑变化。
   - 示例：测试 128、256 等大小，观察缓存效应（例如，L1 缓存 48 KiB、L2 1280 KiB）。

2. **为什么初始化为 `state.range(0)`？**
   - 使用 `state.range(0)` 作为元素值是一个简化选择，可能模拟某种场景（例如，初始化为固定值的数组）。
   - 实际值对性能影响不大，因为 `std::vector` 的初始化是逐元素赋值，值本身不影响内存访问模式。
   - 替代：可以初始化为 0 或其他值，结果类似。

3. **为什么用 `DoNotOptimize` 和 `ClobberMemory`？**
   - `DoNotOptimize(data)` 防止编译器优化掉 `v.data()` 的调用或整个向量的创建。
   - `ClobberMemory()` 确保内存操作（如分配和初始化）不会被编译器缓存或合并。
   - 没有这些，编译器可能优化掉大部分操作，导致时间测量为 0 或不准确。

4. **为什么在循环内创建向量？**
   - 每次迭代创建和销毁 `std::vector` 测量完整的构造和析构开销。
   - 放在循环外会导致向量只创建一次，测试结果仅反映访问 `data()` 的开销（可能被优化掉）。

5. **性能影响因素**：
   - **内存分配**：`std::vector` 使用分配器（通常 `new`），分配和释放的开销随大小增加。
   - **初始化**：逐元素赋值涉及循环，线性增加时间。
   - **缓存效应**：
     - 小向量（例如 128 元素，512 字节）完全在 L1 缓存（48 KiB）中。
     - 大向量（例如 1024 元素，4096 字节）可能部分在 L2 缓存（1280 KiB）。
     - 不会触发 L3 缓存（20480 KiB）或主内存访问。
   - **CPU 频率**：3686 MHz（周期 ~0.2714 ns），初始化 1024 个元素需要 ~1024 周期（~278 ns），实际时间（280.1 ns）接近理论值，说明开销主要来自初始化。

---

### 潜在问题和改进建议

#### 1. **潜在问题**
- **零大小向量**：
  - `state.range(0) = 0` 创建空向量，可能不是有意义的测试用例（分配开销最小）。
  - 修复：从非零值开始，例如 `DenseRange(128, 1024, 128)`。
- **初始化值的意义**：
  - 初始化为 `state.range(0)` 可能不直观（例如，128 个元素值为 128），且值本身对性能无影响。
  - 修复：初始化为固定值（如 0）或随机值（模拟真实数据）。
- **缺少吞吐量报告**：
  - 未调用 `state.SetBytesProcessed`，输出不显示 bytes/s，难以比较内存操作效率。
  - 修复：添加字节处理量（见下文）。
- **编译器优化风险**：
  - 尽管使用了 `DoNotOptimize` 和 `ClobberMemory`，仍需确保编译器不会优化掉其他部分（例如，向量构造）。
  - 检查：使用 `-O3` 编译并验证结果。

#### 2. **改进建议**
- **添加吞吐量报告**：
  - 计算处理的字节数，显示吞吐量：
    ```cpp
    state.SetBytesProcessed(int64_t(state.iterations()) * int64_t(state.range(0)) * sizeof(int));
    ```
    - 每次迭代处理 `state.range(0) * sizeof(int)` 字节。
    - 输出将包含 bytes/s，例如：
      ```
      BM_DenseRange/128    45.6 ns    45.6 ns   5000000   10.7MB/s
      ```
- **测试不同初始化方式**：
  - 比较不同初始化方法的性能：
    - 默认构造：`std::vector<int> v(state.range(0))`（未定义值）。
    - 零初始化：`std::vector<int> v(state.range(0), 0)`.
    - 随机值：使用 `std::generate` 填充随机数。
    ```cpp
    #include <random>
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<> dis(0, 1000);
    std::generate(v.begin(), v.end(), [&]() { return dis(gen); });
    ```
- **测试数据访问**：
  - 当前仅创建向量和获取指针，未实际使用 `data`。
  - 添加数据操作（例如，求和或复制），测量内存访问性能：
    ```cpp
    int sum = 0;
    for (int i = 0; i < state.range(0); ++i) {
        sum += data[i];
    }
    benchmark::DoNotOptimize(sum);
    ```
- **测试不同容器**：
  - 比较 `std::vector` 与其他容器（如 `std::array` 或原始数组）的性能：
    ```cpp
    std::unique_ptr<int[]> arr(new int[state.range(0)]());
    ```
- **缓存预热**：
  - 在计时循环前访问向量，确保数据在缓存中：
    ```cpp
    volatile int dummy = v[0];
    ```
- **多参数测试**：
  - 使用 `Args` 测试更多维度，例如初始化方式或数据模式：
    ```cpp
    BENCHMARK(BM_DenseRange)->Args({128, 0})->Args({128, 1}); // 0: zero, 1: random
    ```

#### 3. **优化输出**：
  - 添加参数打印，增强可读性：
    ```cpp
    std::cout << "Testing vector size: " << state.range(0) << "\n";
    ```
    - 放在循环外，避免影响计时。

