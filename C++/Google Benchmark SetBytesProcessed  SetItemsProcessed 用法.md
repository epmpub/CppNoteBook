# Google Benchmark SetBytesProcessed  SetItemsProcessed 用法

```C++
static void BM_memcpy(benchmark::State& state) {
    char* src = new char[state.range(0)];
    char* dst = new char[state.range(0)];
    memset(src, 'x', state.range(0));
    for (auto _ : state)
        memcpy(dst, src, state.range(0));
    state.SetBytesProcessed(int64_t(state.iterations()) * int64_t(state.range(0)));
    state.SetItemsProcessed(int64_t(state.iterations()) * int64_t(state.range(0)));
    state.SetLabel("size=" + std::to_string(state.range(0)));
    delete[] src;
    delete[] dst;
}
BENCHMARK(BM_memcpy)->Arg(8)->Arg(64)->Arg(512)->Arg(4 << 10)->Arg(8 << 10);
```

```C++
#include <benchmark/benchmark.h>
#include <cstring>
#include <cstdlib>

static void BM_memcpy(benchmark::State& state) {
    // 检查参数有效性
    if (state.range(0) <= 0) {
        state.SkipWithError("Invalid range(0): must be positive");
        return;
    }

    // 使用对齐分配
    char* src = static_cast<char*>(std::aligned_alloc(64, state.range(0)));
    char* dst = static_cast<char*>(std::aligned_alloc(64, state.range(0)));
    if (!src || !dst) {
        state.SkipWithError("Memory allocation failed");
        std::free(src);
        std::free(dst);
        return;
    }

    // 初始化源缓冲区
    memset(src, 'x', state.range(0));

    // 计时循环
    for (auto _ : state) {
        memcpy(dst, src, state.range(0));
    }

    // 记录处理的字节数
    state.SetBytesProcessed(int64_t(state.iterations()) * int64_t(state.range(0)));

    // 释放内存
    std::free(src);
    std::free(dst);
}

// 注册测试，覆盖多种大小
BENCHMARK(BM_memcpy)->Range(1024, 1024 * 1024);
BENCHMARK_MAIN();
```

```C++

### 总结

- **功能**：测量 `memcpy` 在不同字节数下的性能，使用动态分配的缓冲区和计时循环。
- **设计**：通过 `state.range(0)` 参数化字节数，隔离 `memcpy` 的性能，报告吞吐量。
- **关键点**：
  - `state.range(0)` 指定复制字节数。
  - 计时循环只包含 `memcpy`，确保测量准确。
  - `state.SetBytesProcessed` 计算吞吐量。
- **潜在问题**：参数有效性、内存分配失败、缓冲区对齐。
- **改进**：添加检查、使用对齐分配、支持范围测试、考虑缓存预热。

如果你有进一步需求（例如，测试特定大小、比较 `memcpy` 和 `memmove`、或添加多参数支持），请告诉我，我可以提供更具体的代码或分析！
```



```C++
2025-04-21T10:30:38+08:00
Running C:\Users\sheng\source\repos\benchmark-2\x64\Release\benchmark-2.exe
Run on (16 X 3686 MHz CPU s)
CPU Caches:
  L1 Data 48 KiB (x8)
  L1 Instruction 32 KiB (x8)
  L2 Unified 1280 KiB (x8)
  L3 Unified 20480 KiB (x1)
-------------------------------------------------------------------------
Benchmark               Time             CPU   Iterations UserCounters...
-------------------------------------------------------------------------
BM_memcpy/8         0.910 ns        0.921 ns    746666667 bytes_per_second=8.09178Gi/s items_per_second=8.68848G/s size=8
BM_memcpy/64         1.82 ns         1.84 ns    407272727 bytes_per_second=32.3671Gi/s items_per_second=34.7539G/s size=64
BM_memcpy/512        3.48 ns         3.45 ns    194782609 bytes_per_second=138.239Gi/s items_per_second=148.433G/s size=512
BM_memcpy/4096       23.9 ns         23.5 ns     29866667 bytes_per_second=162.037Gi/s items_per_second=173.986G/s size=4096
BM_memcpy/8192       46.8 ns         46.5 ns     14451613 bytes_per_second=164.104Gi/s items_per_second=176.205G/s size=8192

C:\Users\sheng\source\repos\benchmark-2\x64\Release\benchmark-2.exe (process 11700) exited with code 0 (0x0).
```

