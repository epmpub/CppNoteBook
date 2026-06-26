C++26 constexpr std::shared_ptr 及相关智能指针支持

（提案 P3037R6，已被采纳）。这是 C++26 中 constexpr 扩展的重要一环，让智能指针在编译期（constant expression）中能像运行时一样安全、方便地使用动态内存管理。1. 背景与演进

- C++20：引入 constexpr 动态内存分配（new/delete）。
- C++23：std::unique_ptr 支持 constexpr（P2273R3）。
- C++26：std::shared_ptr、std::weak_ptr 及其配套设施全面 constexpr 支持。

核心依赖：P2738R1（constexpr void* 转换），解决了类型擦除（type erasure）和 control block 的实现难题。 

open-std.org

2. 支持范围（“and friends”）

- std::shared_ptr<T>：几乎所有构造函数、成员函数、操作。
- std::weak_ptr<T>：相应支持。
- std::make_shared / allocate_shared 系列（包括 _for_overwrite）。
- 指针转换：static_pointer_cast、dynamic_pointer_cast、const_pointer_cast。
- 比较运算符：==、!=、<=> 等（shared_ptr 和 unique_ptr）。
- std::atomic<std::shared_ptr<T>> 和 std::atomic<std::weak_ptr<T>> 的偏特化（C++26 新增）。
- std::out_ptr / inout_ptr 适配器。
- std::bad_weak_ptr 异常。

不支持（推迟到未来）：

- reinterpret_pointer_cast（依赖 reinterpret_cast）。
- std::hash<shared_ptr<T>> 等哈希（指针到 size_t 的转换问题）。
- I/O 重载（operator<<）。

特性测试宏：__cpp_lib_constexpr_memory（值更新）。 

cppreference.com

3. 使用示例

cpp

```cpp
#include <memory>
#include <cassert>

constexpr int test_shared() {
    auto p1 = std::make_shared<int>(42);
    auto p2 = p1;                    // 共享所有权
    auto p3 = std::make_shared<int[]>(10); // 数组支持

    *p1 = 100;
    assert(*p2 == 100);

    std::weak_ptr<int> w = p1;
    auto sp = w.lock();              // weak_ptr 支持
    assert(sp && *sp == 100);

    auto p4 = std::static_pointer_cast<double>(
        std::make_shared<double>(3.14));

    return *p1 + static_cast<int>(*p4);  // 142
}

static_assert(test_shared() == 142);
```

SYCL buffer 等实际场景（提案中的动机示例）：

cpp

```cpp
struct buffer {
    std::shared_ptr<int[]> data;
    // ...
    constexpr buffer(size_t n) 
        : data(std::make_shared<int[]>(n)) {}
};
```

编译期能自动管理内存，避免原始指针 + 预处理器宏的丑陋代码。4. 实现注意事项

- Control Block：运行时通常单次分配（control block + 对象），constexpr 下可能两次分配（因为 reinterpret_cast 限制），但这仅影响编译期，不影响运行时行为。
- Atomic 操作：常量求值是单线程，库实现用 std::is_constant_evaluated() 绕过原子 intrinsics（或直接依赖 C++26 constexpr std::atomic）。
- 性能：编译期开销可接受，运行时零开销（与之前实现兼容）。
- 与 unique_ptr 对比

| 特性              | unique_ptr (C++23) | shared_ptr (C++26) |
| ----------------- | ------------------ | ------------------ |
| 所有权            | 独占               | 共享               |
| make_* 支持       | 是                 | 是                 |
| 控制块 / 类型擦除 | 简单               | 复杂（已解决）     |
| weak_ptr 关联     | -                  | 支持               |
| 原子偏特化        | -                  | 支持               |

总结C++26 的 constexpr std::shared_ptr 让编译期安全动态内存管理成为现实，极大提升了元编程、编译期计算、测试和嵌入式/性能敏感代码的表达力。你现在可以在 static_assert、constexpr 函数中放心使用共享所有权，而无需回退到裸指针。这体现了 C++ “同一份代码，编译期/运行时一致”的哲学。推荐在新代码中优先使用智能指针，即使在 constexpr 上下文中。需要具体实现细节、与 unique_ptr 混合使用的例子，或某个成员函数的 constexpr 行为吗？