

# Implementing Custom memory resources

使用 C++17 内存资源时，您可能想要实现自定义资源。

幸运的是，这很简单。PMR 库提供了一个抽象基类*std::pmr::memory_resource*。

自定义内存资源必须从此类派生并实现私有方法*do_allocate*、*do_deallocate*和*do_is_equal*。

```C++
#include <memory_resource>
#include <string>
#include <print>
#include <list>
#include <ranges>

// A simple tracing memory resource that uses an upstream resource
// for allocation, but prints information about the allocated
// or deallocated memory.
struct Tracer : std::pmr::memory_resource {
    Tracer(std::string_view label, std::pmr::memory_resource* upstream) : label_(label), up_(upstream) {}
private:
    // allocation implementation
    void* do_allocate(std::size_t bytes, std::size_t alignment) override {
        void* ptr = up_->allocate(bytes, alignment);
        std::println("[{}] allocated {}, {} bytes with {} byte allignment",
            label_, ptr, bytes, alignment);
        return ptr;
    }
    // deallocation implementation
    void do_deallocate(void* p, std::size_t bytes, std::size_t alignment) override {
        std::println("[{}] deallocating {}, {} bytes with {} byte alignment",
            label_, p, bytes, alignment);
        up_->deallocate(p, bytes, alignment);
    }
    // used to determine whether the container using this resource can move
    // or has to fallback to a copy, we cannot move across resources
    bool do_is_equal(const std::pmr::memory_resource& other) const noexcept override {
        // Tracer is transparent, so a Tracer(&res) == res
        if (up_ == &other)
            return true;
        // Two different tracers are equivalent
        // if they point to the same backing resource
        if (auto* ptr = dynamic_cast<const Tracer*>(&other); ptr != nullptr) {
            return up_ == ptr->up_;
        }
        return false;
    }  

    std::string label_;
    std::pmr::memory_resource* up_;
};

int main() {
    Tracer system("system", std::pmr::new_delete_resource());
    std::pmr::unsynchronized_pool_resource pool_res(&system);
    Tracer pool("pool", &pool_res);

    std::pmr::list<std::pmr::string> list(&pool);
    for (auto _ : std::views::iota(0, 128)) {
        list.push_back("This is going to allocate because it doesn't fit into SSO.");
    }
}
```

这段代码展示了 C++17 中引入的 <memory_resource> 库，用于自定义内存分配器。代码定义了一个简单的跟踪内存资源 Tracer，它包装了一个上游资源（upstream resource），并在分配和释放内存时打印跟踪信息。以下是逐步解释。

------

代码概览

- 定义 Tracer 类，继承 std::pmr::memory_resource，实现自定义内存分配和释放。
- 在 main 中使用 Tracer 跟踪标准分配器和池分配器的行为。
- 使用 std::pmr::list 和 std::pmr::string 测试内存分配。

------

关键组件

1. **头文件**

cpp

```cpp
#include <memory_resource>
#include <string>
#include <print>
#include <list>
#include <ranges>
```

- <memory_resource>：提供 std::pmr::memory_resource 和相关类。
- <string>：提供 std::pmr::string。
- <print>：C++23 的 std::println 用于输出。
- <list>：提供 std::pmr::list。
- <ranges>：提供 std::views::iota。
- **Tracer 类定义**

cpp

```cpp
struct Tracer : std::pmr::memory_resource {
    Tracer(std::string_view label, std::pmr::memory_resource* upstream) 
        : label_(label), up_(upstream) {}
```

- **继承**：
  - 从 std::pmr::memory_resource 派生，实现自定义内存资源。
- **成员**：
  - label_：标识资源名称。
  - up_：上游资源指针，用于实际分配。

**分配实现**

cpp

```cpp
void* do_allocate(std::size_t bytes, std::size_t alignment) override {
    void* ptr = up_->allocate(bytes, alignment);
    std::println("[{}] allocated {}, {} bytes with {} byte allignment",
        label_, ptr, bytes, alignment);
    return ptr;
}
```

- **do_allocate**：
  - 分配 bytes 大小的内存，满足 alignment 对齐。
  - 委托给 up_ 执行分配，记录并返回指针。

**释放实现**

cpp

```cpp
void do_deallocate(void* p, std::size_t bytes, std::size_t alignment) override {
    std::println("[{}] deallocating {}, {} bytes with {} byte alignment",
        label_, p, bytes, alignment);
    up_->deallocate(p, bytes, alignment);
}
```

- **do_deallocate**：
  - 释放指针 p 的内存，记录信息后委托给 up_。

**相等性检查**

cpp

```cpp
bool do_is_equal(const std::pmr::memory_resource& other) const noexcept override {
    if (up_ == &other)
        return true;
    if (auto* ptr = dynamic_cast<const Tracer*>(&other); ptr != nullptr) {
        return up_ == ptr->up_;
    }
    return false;
}
```

- **do_is_equal**：
  - 检查两个内存资源是否等价，用于容器移动或比较。
  - 若 other 是 up_，返回 true（透明性）。
  - 若 other 是 Tracer，比较其 up_ 是否相同。
- **作用**：
  - 支持跨资源比较，影响容器移动行为。
- **main 函数**

cpp

```cpp
Tracer system("system", std::pmr::new_delete_resource());
std::pmr::unsynchronized_pool_resource pool_res(&system);
Tracer pool("pool", &pool_res);
std::pmr::list<std::pmr::string> list(&pool);
for (auto _ : std::views::iota(0, 128)) {
    list.push_back("This is going to allocate because it doesn't fit into SSO.");
}
```

- **system**：
  - Tracer 实例，包装 std::pmr::new_delete_resource()（使用 new 和 delete 的默认分配器）。
- **pool_res**：
  - 未同步池资源，基于 system 分配内存。
- **pool**：
  - Tracer 实例，包装 pool_res。
- **list**：
  - 使用 pool 作为内存资源的链表，存储 std::pmr::string。
- **循环**：
  - 使用 std::views::iota(0, 128) 迭代 128 次。
  - 每次插入长字符串，超出 SSO（Small String Optimization，小对象优化）阈值，触发动态分配。

------

执行流程

1. **分配**：

   - list.push_back 创建 std::pmr::string，请求内存。

   - pool 跟踪并委托给 pool_res，可能再委托给 system。

   - 输出分配信息，如：

     ```text
     [pool] allocated 0x..., 32 bytes with 8 byte alignment
     [system] allocated 0x..., 64 bytes with 8 byte alignment
     ```

2. **释放**：

   - 程序结束，list 析构，释放所有字符串内存。
   - 输出释放信息。

------

输出示例

假设每次分配 32 字节，8 字节对齐（实际大小依赖实现）：

```text
[pool] allocated 0x..., 32 bytes with 8 byte alignment
[system] allocated 0x..., 64 bytes with 8 byte alignment
...
[pool] deallocating 0x..., 32 bytes with 8 byte alignment
[system] deallocating 0x..., 64 bytes with 8 byte alignment
```

- 具体地址和大小因平台和实现而异。

------

为什么这样工作？

1. **std::pmr::memory_resource**：
   - 提供抽象接口，允许自定义分配逻辑。
2. **Tracer**：
   - 包装上游资源，添加跟踪功能。
3. **do_is_equal**：
   - 确保资源比较正确，支持容器操作。
4. **PMR 容器**：
   - std::pmr::list 和 std::pmr::string 使用指定的内存资源。

------

使用场景

- **内存调试**：
  - 跟踪分配和释放。
- **自定义分配**：
  - 测试池分配器性能。
- **资源管理**：
  - 在多线程或复杂系统中监控内存。

------

总结

- Tracer 包装内存资源，记录分配和释放。
- system 和 pool 形成资源链，供 list 使用。
- 128 次插入触发动态分配，展示跟踪功能。
- 代码展示了 PMR 的灵活性和调试能力。