C++26 undeprecated std::polymorphic_allocator::destory 解释**

```cpp
#include <memory_resource>   // polymorphic_allocator, monotonic_buffer_resource
#include <iostream>
#include <array>

int main()
{
    // 1. 准备一块内存缓冲区（推荐使用 monotonic_buffer_resource）
    std::array<std::byte, 256> buffer{};   // 256 字节栈上缓冲区

    std::pmr::monotonic_buffer_resource mr(buffer.data(), buffer.size());

    // 2. 使用 polymorphic_allocator
    std::pmr::polymorphic_allocator<int> alloc{&mr};

    // 3. 分配 + 构造 + 销毁 + 释放
    int* p = alloc.allocate(1);           // 分配 1 个 int 的空间
    alloc.construct(p, 42);               // 构造对象，初始化为 42

    std::cout << "Value: " << *p << '\n';

    alloc.destroy(p);                     // 销毁对象（C++26 已 undeprecated）
    alloc.deallocate(p, 1);               // 释放内存

    return 0;
}
```

### 编译指令（推荐）

```bash
# 使用 C++20 或 C++26
g++ -std=c++20 -Wall -Wextra example.cpp -o example
# 或
clang++ -std=c++20 -Wall -Wextra example.cpp -o example
```

### 其他可选的内存资源方式

1. **最简单（使用全局 new/delete）**：
   ```cpp
   std::pmr::polymorphic_allocator<int> alloc{std::pmr::new_delete_resource()};
   ```

2. **使用 `std::pmr::unsynchronized_pool_resource`**（适合多次小对象分配）：
   ```cpp
   std::pmr::unsynchronized_pool_resource pool;
   std::pmr::polymorphic_allocator<int> alloc{&pool};
   ```

---

**关键点说明**：

- `monotonic_buffer_resource` 是最常用的演示用内存资源。
- `allocate` / `deallocate` 的第二个参数是**对象个数**，不是字节数。
- `construct` / `destroy` 是对称操作，`destroy` 在 C++26 中已正式取消弃用。
- 栈上的 `std::array<std::byte, N>` 是最安全的演示方式，避免内存泄漏。

需要我再加上错误处理、分配多个对象、或者配合 `std::pmr::vector` 的例子吗？