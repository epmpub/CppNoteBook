# C++ 26 硬件缓存特性（std::hardware_destructive_interference_size 和 std::hardware_constructive_interference_size）



```C++
#include <new>
#include <atomic>

struct NoShare {
  // Ensures that flag1 and flag2 do not share a (L1) cache line
  alignas(std::hardware_destructive_interference_size) 
    std::atomic<bool> flag1;
  alignas(std::hardware_destructive_interference_size) 
    std::atomic<bool> flag2;
};

struct Shared {
    std::atomic<bool> flag;
    int data;
    int also_data;
};
// Ensure that flag, data and other_data will be in the same cache line
static_assert(sizeof(Shared) <= std::hardware_constructive_interference_size);

int main() {
    // Align using destructive size to prevent false sharing
    // with other local variables
    alignas(sizeof(std::hardware_destructive_interference_size))
        Shared shared;
}
```

这段代码展示了 C++ 中如何利用内存对齐（alignas）和硬件缓存特性（std::hardware_destructive_interference_size 和 std::hardware_constructive_interference_size）来优化多线程程序的性能，避免伪共享（false sharing）问题，并确保数据在缓存行中的布局符合预期。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <new>：提供 alignas 和缓存行大小常量（如 std::hardware_destructive_interference_size）。
   - <atomic>：提供 std::atomic，用于线程安全的原子操作。
2. **主要内容**：
   - 定义 NoShare 结构体，避免两个原子变量共享缓存行。
   - 定义 Shared 结构体，确保多个变量在同一缓存行内。
   - 在 main 中使用对齐来实例化 Shared。

------

**关键概念：缓存行和伪共享**

- **缓存行（Cache Line）**：
  - 现代 CPU 使用缓存来加速内存访问，缓存以固定大小的块（通常 64 字节）加载和存储数据，这些块称为缓存行。
- **伪共享（False Sharing）**：
  - 当多个线程同时访问同一缓存行中的不同变量，且至少一个线程写入时，会导致缓存失效和同步开销，即使这些变量逻辑上独立。
  - 例如，两个线程分别操作 flag1 和 flag2，如果它们在同一缓存行，修改 flag1 会导致 flag2 的缓存失效。

------

**代码逐步解释**

**1. NoShare 结构体**

cpp

```cpp
struct NoShare {
  alignas(std::hardware_destructive_interference_size) 
    std::atomic<bool> flag1;
  alignas(std::hardware_destructive_interference_size) 
    std::atomic<bool> flag2;
};
```

- **std::hardware_destructive_interference_size**：
  - C++17 引入的常量，表示避免伪共享的最小对齐大小。
  - 通常等于缓存行大小（例如 64 字节），具体值依赖硬件架构。
- **alignas**：
  - 指定变量的对齐方式，确保 flag1 和 flag2 的内存地址按 std::hardware_destructive_interference_size 对齐。
  - 效果：flag1 和 flag2 被强制分开放在不同的缓存行中。
- **std::atomic<bool>**：
  - 线程安全的布尔值，适用于多线程环境。
- **目的**：
  - 防止 flag1 和 flag2 共享同一缓存行，避免伪共享。
  - 例如，一个线程修改 flag1，不会影响另一个线程访问 flag2 的缓存。

------

**2. Shared 结构体**

cpp

```cpp
struct Shared {
    std::atomic<bool> flag;
    int data;
    int also_data;
};
static_assert(sizeof(Shared) <= std::hardware_constructive_interference_size);
```

- **成员**：
  - flag（std::atomic<bool>）：1 字节（通常对齐到 4 或 8 字节）。
  - data 和 also_data（int）：各 4 字节。
- **sizeof(Shared)**：
  - 假设 flag 对齐到 4 字节，data 和 also_data 各 4 字节，总大小约为 12 字节（可能因填充对齐到 16 字节）。
- **std::hardware_constructive_interference_size**：
  - C++17 引入的常量，表示缓存行中可以共享的最大大小（通常 64 字节）。
  - 表示这些变量应该放在同一缓存行以利用空间局部性。
- **static_assert**：
  - 静态断言，确保 Shared 的大小不超过缓存行大小。
  - 目的：保证 flag、data 和 also_data 在同一缓存行内，避免跨缓存行访问。
- **目的**：
  - 当这些变量总是被一起访问时，将它们放在同一缓存行可以减少缓存未命中，提高性能。

------

**3. main 函数**

cpp

```cpp
int main() {
    alignas(sizeof(std::hardware_destructive_interference_size))
        Shared shared;
}
```

- **alignas**：
  - 将 shared 对象的起始地址对齐到 std::hardware_destructive_interference_size（例如 64 字节）。
  - 注意：这里使用 sizeof(std::hardware_destructive_interference_size) 是语法错误，应直接写 alignas(std::hardware_destructive_interference_size)。
    - sizeof 返回常量的大小（通常 8 字节，size_t 大小），而非对齐值。
    - 正确的意图是对齐到缓存行大小。
- **效果**：
  - 确保 shared 与栈上的其他变量（如局部变量）不会共享缓存行，避免伪共享。
- **Shared shared**：
  - 创建 Shared 类型的实例，其内部成员已在同一缓存行内。

------

**关键技术点**

1. **缓存行对齐**：
   - std::hardware_destructive_interference_size：避免伪共享的最小间距。
   - std::hardware_constructive_interference_size：利用缓存行共享的最大范围。
   - 典型值均为 64 字节（视架构而定，例如 x86-64）。
2. **alignas**：
   - 控制变量的内存对齐，确保符合硬件优化需求。
   - 可用于结构体成员或整个对象。
3. **伪共享优化**：
   - NoShare：通过对齐分隔 flag1 和 flag2，适合独立访问的变量。
   - Shared：通过限制大小聚合同一缓存行，适合相关数据的访问。
4. **静态断言**：
   - 在编译期验证设计假设，确保 Shared 符合缓存行要求。

------

**可能的改进或注意事项**

1. **语法错误**：
   - alignas(sizeof(std::hardware_destructive_interference_size)) 应改为 alignas(std::hardware_destructive_interference_size)。
   - 当前代码可能导致意外的对齐（例如 8 字节而非 64 字节）。
2. **实际效果**：
   - 对齐优化在多线程程序中更明显，单线程示例中难以观察。
   - 可添加多线程测试（例如两个线程分别修改 flag1 和 flag2）验证伪共享避免。
3. **平台依赖**：
   - std::hardware_destructive_interference_size 和 std::hardware_constructive_interference_size 的值由实现定义，通常为 64 字节，但应检查具体编译器文档。

------

**运行流程总结**

- 定义 NoShare：确保 flag1 和 flag2 在不同缓存行。
- 定义 Shared：确保 flag、data 和 also_data 在同一缓存行。
- 在 main 中创建对齐的 shared 对象，避免与栈上其他变量的伪共享。

------

**总结**

这段代码展示了 C++ 中如何通过内存对齐优化缓存行为：

- **避免伪共享**：NoShare 将独立变量分隔到不同缓存行。
- **利用空间局部性**：Shared 将相关变量聚合同一缓存行。
- **硬件特性**：利用 C++17 的缓存行大小常量和 alignas。

如果你有具体问题（例如伪共享的实际测试），欢迎进一步提问！