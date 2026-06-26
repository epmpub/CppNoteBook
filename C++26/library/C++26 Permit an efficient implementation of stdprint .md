

## C++26 Permit an efficient implementation of std::print 

```c
#include <format>
#include <print>
#include <chrono>
#include <vector>
#include <iostream>
#include <cstdint>

struct Timestamp {
    int64_t value;
};

// ==================== 自定义 formatter ====================
template <>
struct std::formatter<Timestamp> {
    constexpr auto parse(std::format_parse_context& ctx) {
        return ctx.begin();
    }

    auto format(const Timestamp& ts, std::format_context& ctx) const {
        // 使用最简单的格式字符串，保证 consteval 友好
        return std::format_to(ctx.out(), "[{}]", ts.value);
    }
};

// ==================== 启用 C++26 高效 non-locking 优化 ====================
template <>
constexpr bool std::enable_nonlocking_formatter_optimization<Timestamp> = true;
// ===========================================================

int main() {
    const int N = 100'000;
    std::vector<Timestamp> data(N);
    for (int i = 0; i < N; ++i) {
        data[i].value = 1720000000000LL + i * 1000;   // 模拟时间戳
    }

    // ==================== C++26 高效路径 (直接写入) ====================
    auto start1 = std::chrono::high_resolution_clock::now();
    for (const auto& ts : data) {
        std::print("{}", ts);
    }
    auto end1 = std::chrono::high_resolution_clock::now();

    // ==================== C++23 风格 (临时 string) ====================
    auto start2 = std::chrono::high_resolution_clock::now();
    for (const auto& ts : data) {
        std::string s = std::format("{}", ts);
        std::cout << s;
    }
    auto end2 = std::chrono::high_resolution_clock::now();

    auto duration1 = std::chrono::duration<double, std::milli>(end1 - start1).count();
    auto duration2 = std::chrono::duration<double, std::milli>(end2 - start2).count();

    std::println("\n\n=== 性能对比 ({} 次) ===", N);
    std::println("std::print (C++26 efficient) : {:.2f} ms", duration1);
    std::println("std::format + cout           : {:.2f} ms", duration2);
    std::println("加速比: {:.2f}x", duration2 / duration1);

    return 0;
}
```

