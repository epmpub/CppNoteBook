# C++ Working with alignment requirements

内存中的每个对象都根据其对应类型的对齐要求而具有对齐要求。对齐要求始终是 2 的幂，并且具有该对齐要求的对象只能放置在为对齐要求倍数的内存地址上。

对齐的主要结果是连续的变量或类成员可能需要额外的空间来进行必要的填充。

当在位置构造对象时，可能需要手动对齐指针。



```C++
#include <cstdint>
#include <memory>
#include <print>

// For x86-64
struct Data { // sizeof == 48
    char a;        // sizeof == 1, alignof == 1
                   // padding 3
    int b;         // sizeof == 4, alignof == 4
                   // padding 8
    long double c; // sizeof == 16, alignof == 16
                   // no padding
    bool d;        // sizeof == 1, alignof == 1
                   // padding 7
    int64_t e;     // sizeof == 8, alignof == 8
};

size_t padding(size_t offset, size_t alignment) {
    if (offset % alignment == 0)
        return 0;
    return alignment - offset % alignment;
}

int main() {
    size_t offset = 0;

    std::println("sizeof(Data) == {} bytes", sizeof(Data));
    std::println("\tsizeof(Data::a): {}", sizeof(Data::a));
    offset += sizeof(Data::a);
    std::println("\tpadding: {}", padding(offset, alignof(decltype(Data::b))));
    offset += padding(offset, alignof(decltype(Data::b)));
    std::println("\tsizeof(Data::b): {}", sizeof(Data::b));
    offset += sizeof(Data::b);
    std::println("\tpadding: {}", padding(offset, alignof(decltype(Data::c))));
    offset += padding(offset, alignof(decltype(Data::c)));
    std::println("\tsizeof(Data::c): {}", sizeof(Data::c));
    offset += sizeof(Data::c);
    std::println("\tpadding: {}", padding(offset, alignof(decltype(Data::d))));
    offset += padding(offset, alignof(decltype(Data::d)));
    std::println("\tsizeof(Data::d): {}", sizeof(Data::d));
    offset += sizeof(Data::d);
    std::println("\tpadding: {}", padding(offset, alignof(decltype(Data::e))));
    offset += padding(offset, alignof(decltype(Data::e)));
    std::println("\tsizeof(Data::d): {}\n", sizeof(Data::e));
    offset += sizeof(Data::e);

    // C++11 std::alignment_of and C++17 std::alignment_of_v
    size_t align_of_int64_t = std::alignment_of<int64_t>::value;

    std::println("align_of_int64_t == {}", align_of_int64_t);

    // same as
    align_of_int64_t = std::alignment_of_v<int64_t>;  // C++17

    std::println("align_of_int64_t == {}\n", align_of_int64_t);

    char bad{}; *(char volatile*)&bad; // inject an anoying offset
    std::byte place1[sizeof(int64_t)]; // not properly aligned for int64_t
    // int64_t *ptr1 = new (place1) int64_t(42); // Undefined Behaviour

    std::println("&place1 % alignof(int64_t) == {}", std::bit_cast<uint64_t>(&place1) % alignof(int64_t));
    
    // Force alignment using alignas (also works for members)
    alignas(int64_t) std::byte place2[sizeof(int64_t)];
    int64_t* ptr2 = new (place2) int64_t(42); // OK
    ptr2->~int64_t();

    std::println("&place2 % alignof(int64_t) == {}\n", std::bit_cast<uint64_t>(&place2) % alignof(int64_t));

    // For pointers, std::align provides a runtime solution
    std::byte place3[32];
    void* ptr3 = place3 + 2; // not properly aligned for int64_t

    std::println("ptr3 % alignof(int64_t) == {}", std::bit_cast<uint64_t>(ptr3) % alignof(int64_t));

    size_t sz = 32-2; // remaining space in place3
    std::align(alignof(int64_t), sizeof(int64_t), ptr3, sz);
    // sz == 32-2-padding
    // ptr3 == place3 + 2 + padding
    
    std::println("place3 == {}, ptr3_aligned == {}, sz == {}", (void*)place3, ptr3, sz);

    // Returns the adjusted pointer or nullptr if the operation is not possible
    void* ptr4 = std::align(16, 17, ptr3, sz);
    // ptr4 == nullptr, can't fit 17 bytes after ptr3 
    // with 16 byte alignment, ptr3 and sz unchanged

    std::println("ptr4 == {}, ptr3 == {}, sz == {}", ptr4, ptr3, sz);

    // std::max_align_t has an alignment requirent at least as strict as scalar types
    alignas(std::max_align_t) char buffer[512]; // typical use
}
```

我来把这个关于 C++ 内存对齐和填充的代码解释翻译成中文，并保持技术准确性。

------

1. **结构体定义和内存布局**

cpp

```cpp
struct Data {    // 总大小：48 字节
    char a;      // 1 字节 + 3 字节填充（对齐下一个 int）
    int b;       // 4 字节 + 8 字节填充（对齐 long double）
    long double c; // 16 字节
    bool d;      // 1 字节 + 7 字节填充（对齐 int64_t）
    int64_t e;   // 8 字节
};
```

- 在 x86-64 架构上，每个成员必须按照其自然对齐（alignof）要求排列
- 总大小为 48 字节，因为：
  - 包含成员之间的填充字节以满足对齐要求
  - 最终大小必须是最大对齐要求（long double 的 16 字节）的倍数

1. **填充计算函数**

cpp

```cpp
size_t padding(size_t offset, size_t alignment) {
    if (offset % alignment == 0)
        return 0;
    return alignment - offset % alignment;
}
```

- 计算到达下一个对齐地址所需的字节数
- 示例：如果 offset=1，alignment=4，则返回 3 字节填充

1. **主函数 - 布局分析** 代码会打印：

- sizeof(Data) = 48 字节
- 偏移量进展：
  - a: 0-1（1 字节）
  - 填充：3 字节（到达 4 字节边界）
  - b: 4-8（4 字节）
  - 填充：8 字节（到达 16 字节边界）
  - c: 16-32（16 字节）
  - 填充：7 字节（到达 8 字节边界）
  - d: 32-33（1 字节）
  - 填充：7 字节
  - e: 40-48（8 字节）

1. **对齐工具**

- std::alignment_of<T> / std::alignment_of_v<T>：获取类型的对齐要求
  - 对于 int64_t，返回 8（字节）
- alignas：强制指定对齐
- std::align：运行时指针对齐

1. **对齐示例**

cpp

```cpp
std::byte place1[sizeof(int64_t)];  // 可能未对齐
alignas(int64_t) std::byte place2[sizeof(int64_t)];  // 保证对齐
```

- place1：如果用于 int64_t 可能导致未定义行为
- place2：强制 8 字节对齐，可安全用于 int64_t

1. **动态指针对齐**

cpp

```cpp
std::byte place3[32];
void* ptr3 = place3 + 2;  // 未对齐
std::align(alignof(int64_t), sizeof(int64_t), ptr3, sz);
```

- 将 ptr3 调整到下一个有效的对齐边界
- 更新 sz 为剩余空间
- 如果对齐无法实现则返回 nullptr

1. **max_align_t**

cpp

```cpp
alignas(std::max_align_t) char buffer[512];
```

- 确保满足任何基本类型的最大对齐要求
- 常用于通用缓冲区

**关键概念：**

- 对齐确保高效的内存访问
- 填充填充未对齐成员之间的空隙
- 未对齐的访问可能导致未定义行为或性能问题
- 像 alignas 和 std::align 这样的工具帮助显式管理对齐

这段代码展示了在 x86-64 系统上检查和控制 C++ 内存布局的实用方法。

