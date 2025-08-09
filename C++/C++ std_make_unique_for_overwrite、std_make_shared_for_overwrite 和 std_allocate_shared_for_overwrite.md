# std::make_unique_for_overwrite、std::make_shared_for_overwrite 和 std::allocate_shared_for_overwrite

```C++
#include <memory>
#include <memory_resource>

int main() {
    auto p1 = std::make_unique_for_overwrite<int>();
    // decltype(p1) == std::unique_ptr<int>, *p1 == indeterminate value
    static_assert(std::is_same_v<decltype(p1), std::unique_ptr<int>>);

    auto p2 = std::make_shared_for_overwrite<int>();
    // decltype(p2) == std::shared_ptr<int>, *p2 == indeterminate value
    static_assert(std::is_same_v<decltype(p2), std::shared_ptr<int>>);

    std::pmr::monotonic_buffer_resource mr;
    std::pmr::polymorphic_allocator<int> alloc{&mr};
    auto p3 = std::allocate_shared_for_overwrite<int>(alloc);
    // decltype(p3) == std::shared_ptr<int>, *p3 == indeterminate value
    static_assert(std::is_same_v<decltype(p3), std::shared_ptr<int>>);

    // Overloads also support default-initialized arrays
    auto p4 = std::make_unique_for_overwrite<int[]>(7); // array with 7 elements
    // decltype(p4) == std::unique_ptr<int[]>, p4[0] == indeterminate value
    static_assert(std::is_same_v<decltype(p4), std::unique_ptr<int[]>>);

    auto p5 = std::make_shared_for_overwrite<int[]>(7);
    // decltype(p5) == std::shared_ptr<int[]>, p5[0] == indeterminate value
    static_assert(std::is_same_v<decltype(p5), std::shared_ptr<int[]>>);

    auto p6 = std::allocate_shared_for_overwrite<int[]>(alloc, 7);
    // decltype(p6) == std::shared_ptr<int[]>, p6[0] == indeterminate value
    static_assert(std::is_same_v<decltype(p6), std::shared_ptr<int[]>>);
}
```

这段代码展示了 C++20 中引入的几个新函数：std::make_unique_for_overwrite、std::make_shared_for_overwrite 和 std::allocate_shared_for_overwrite，

它们用于创建智能指针（std::unique_ptr 和 std::shared_ptr），并支持覆盖初始化（overwrite initialization）。

这些函数与传统的 make_unique 和 make_shared 的区别在于，它们不会对分配的对象进行默认初始化，而是保留未定义值（indeterminate value），从而提高性能。

以下是对代码的逐步解释：

------

1. 单对象智能指针

std::make_unique_for_overwrite

cpp

```cpp
auto p1 = std::make_unique_for_overwrite<int>();
static_assert(std::is_same_v<decltype(p1), std::unique_ptr<int>>);
```

- **用途**：创建一个 std::unique_ptr<int>，管理单个 int 对象的动态内存。
- **特点**：
  - 分配内存但不执行默认初始化，*p1 的值是未定义的（indeterminate value）。
  - 与 std::make_unique<int>() 的区别在于，后者会将 int 初始化为 0，而这里不会。
- **类型**：decltype(p1) 是 std::unique_ptr<int>，通过 static_assert 验证。
- **场景**：适用于后续会立即覆盖对象值的场景，避免初始化开销。

std::make_shared_for_overwrite

cpp

```cpp
auto p2 = std::make_shared_for_overwrite<int>();
static_assert(std::is_same_v<decltype(p2), std::shared_ptr<int>>);
```

- **用途**：创建一个 std::shared_ptr<int>，管理单个 int 对象的共享所有权。
- **特点**：
  - 分配内存但不初始化，*p2 的值是未定义的。
  - 与 std::make_shared<int>() 不同，后者会初始化为 0。
- **类型**：decltype(p2) 是 std::shared_ptr<int>。
- **优势**：与 make_unique 类似，但使用共享所有权模型。

std::allocate_shared_for_overwrite

cpp

```cpp
std::pmr::monotonic_buffer_resource mr;
std::pmr::polymorphic_allocator<int> alloc{&mr};
auto p3 = std::allocate_shared_for_overwrite<int>(alloc);
static_assert(std::is_same_v<decltype(p3), std::shared_ptr<int>>);
```

- **用途**：使用自定义分配器（polymorphic_allocator）创建一个 std::shared_ptr<int>。
- **特点**：
  - 使用 std::pmr::monotonic_buffer_resource 作为内存资源，提供连续的、非释放的内存分配。
  - *p3 的值是未定义的。
  - 与 std::make_shared_for_overwrite 的区别在于，它允许指定分配器。
- **类型**：decltype(p3) 是 std::shared_ptr<int>。
- **场景**：适用于需要自定义内存分配策略的场景（如内存池）。

------

2. 数组智能指针

std::make_unique_for_overwrite（数组）

cpp

```cpp
auto p4 = std::make_unique_for_overwrite<int[]>(7);
static_assert(std::is_same_v<decltype(p4), std::unique_ptr<int[]>>);
```

- **用途**：创建一个 std::unique_ptr<int[]>，管理包含 7 个 int 元素的动态数组。
- **特点**：
  - 分配内存但不对数组元素进行初始化，p4[0] 等元素的值是未定义的。
  - 与 std::make_unique<int[]>(7) 不同，后者会将所有元素初始化为 0。
- **类型**：decltype(p4) 是 std::unique_ptr<int[]>。
- **注意**：unique_ptr<int[]> 使用 delete[] 释放内存，适合管理动态数组。

std::make_shared_for_overwrite（数组）

cpp

```cpp
auto p5 = std::make_shared_for_overwrite<int[]>(7);
static_assert(std::is_same_v<decltype(p5), std::shared_ptr<int[]>>);
```

- **用途**：创建一个 std::shared_ptr<int[]>，管理包含 7 个 int 元素的共享所有权数组。
- **特点**：
  - 数组元素未初始化，p5[0] 是未定义值。
- **类型**：decltype(p5) 是 std::shared_ptr<int[]>。
- **优势**：支持共享所有权，与 unique_ptr 版本类似但允许多个拥有者。

std::allocate_shared_for_overwrite（数组）

cpp

```cpp
auto p6 = std::allocate_shared_for_overwrite<int[]>(alloc, 7);
static_assert(std::is_same_v<decltype(p6), std::shared_ptr<int[]>>);
```

- **用途**：使用自定义分配器创建一个 std::shared_ptr<int[]>，管理包含 7 个元素的数组。
- **特点**：
  - 使用 alloc（基于 monotonic_buffer_resource）分配内存。
  - 数组元素未初始化，p6[0] 是未定义值。
- **类型**：decltype(p6) 是 std::shared_ptr<int[]>。
- **场景**：结合多态分配器和共享所有权，用于高级内存管理。

------

关键点

1. **与传统函数的区别**：
   - 传统 make_unique 和 make_shared 会执行默认初始化（值初始化为 0）。
   - *_for_overwrite 版本跳过初始化，适用于后续会覆盖值的场景，减少性能开销。
2. **未定义值（indeterminate value）**：
   - 未初始化的对象在访问时可能导致未定义行为（UB），因此这些函数假定用户会立即覆盖值。
3. **支持的类型**：
   - 单对象：unique_ptr<T> 和 shared_ptr<T>。
   - 数组：unique_ptr<T[]> 和 shared_ptr<T[]>。
4. **多态分配器**：
   - std::pmr::monotonic_buffer_resource 是一种只增长、不释放的内存资源，适合临时或连续分配。
   - std::allocate_shared_for_overwrite 允许自定义内存管理。

------

总结

这段代码展示了 C++20 中用于创建智能指针的新工具，强调了性能优化（避免不必要的初始化）和灵活性（支持自定义分配器）。通过 static_assert，验证了返回类型的正确性。这些函数特别适用于需要高效内存分配且明确会覆盖初始值的场景，例如缓冲区管理或对象池。