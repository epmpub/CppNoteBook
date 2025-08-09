# atomic compare_exchange_weak 与 compare_exchange_strong

在 C++ 中，std::atomic 提供了原子操作机制，包括 compare_exchange_weak 和 compare_exchange_strong，用于实现比较并交换（CAS）操作。以下是对这些函数以及 std::atomic<std::shared_ptr<T>> 的中文解释。

------

1. compare_exchange_weak 与 compare_exchange_strong

compare_exchange_weak 和 compare_exchange_strong 是 std::atomic<T> 的成员函数，用于执行比较并交换（CAS）操作。它们会原子地比较原子对象的值与 *期望值*（expected），如果相等，则将其替换为 *目标值*（desired）。两者的主要区别在于行为和使用场景。

compare_exchange_weak

- **签名**：

  cpp

  ```cpp
  bool compare_exchange_weak(T& expected, T desired,
                            std::memory_order success_order,
                            std::memory_order failure_order) noexcept;
  ```

- **行为**：

  - 比较原子对象的值与 expected。
  - 如果相等，将原子对象的值替换为 desired。
  - 如果不相等，将 expected 更新为当前原子对象的值。
  - 返回 true 表示交换成功，false 表示失败。
  - **可能出现伪失败**：即使比较成功，操作也可能失败（例如由于硬件限制）。因此适合在循环中重试。

- **使用场景**：

  - 适用于需要重试的循环场景，例如无锁算法，性能通常较高。

- **示例**：

  cpp

  ```cpp
  std::atomic<int> x{0};
  int expected = 0;
  int desired = 1;
  while (!x.compare_exchange_weak(expected, desired)) {
      expected = 0; // 比较失败时重置 expected
  }
  // x 现在为 1
  ```

compare_exchange_strong

- **签名**：

  cpp

  ```cpp
  bool compare_exchange_strong(T& expected, T desired,
                              std::memory_order success_order,
                              std::memory_order failure_order) noexcept;
  ```

- **行为**：

  - 与 compare_exchange_weak 类似，但 **不会出现伪失败**。
  - 如果比较失败，仅因为原子对象的值与 expected 不匹配。
  - 返回 true 表示交换成功，false 表示失败。

- **使用场景**：

  - 适用于需要保证比较成功时立即完成，或重试代价较高的场景。
  - 适合单次 CAS 操作，不需要循环。

- **示例**：

  cpp

  ```cpp
  std::atomic<int> x{0};
  int expected = 0;
  int desired = 1;
  if (x.compare_exchange_strong(expected, desired)) {
      // 交换成功，x 现在为 1
  } else {
      // 交换失败，expected 包含 x 的当前值
  }
  ```

主要区别

| 特性     | compare_exchange_weak  | compare_exchange_strong |
| -------- | ---------------------- | ----------------------- |
| 伪失败   | 可能即使比较成功也失败 | 不会出现伪失败          |
| 性能     | 某些平台上通常更快     | 由于更强保证，略慢      |
| 使用场景 | 循环、无锁算法         | 单次 CAS 或重试代价高   |

内存序

两个函数都接受 std::memory_order 参数来控制内存同步：

- success_order：交换成功时的内存序（如 std::memory_order_seq_cst 表示顺序一致性）。
- failure_order：交换失败时的内存序（如 std::memory_order_relaxed 表示最小同步）。
- 常见选择：两者都使用 std::memory_order_seq_cst（默认，最强保证）。

------

2. std::atomic<std::shared_ptr<T>>

从 C++20 开始，std::atomic 针对 std::shared_ptr<T> 进行了特化，支持对共享指针的原子操作。这在多线程场景中非常有用，例如原子地更新共享资源指针。

关键点

- std::atomic<std::shared_ptr<T>> 支持原子操作，如 load、store、exchange、compare_exchange_weak 和 compare_exchange_strong。
- 在大多数现代平台上，这些操作是无锁的（取决于实现，可用 is_lock_free() 检查）。
- 特化确保 shared_ptr 的控制块和指针被原子更新，保证线程安全。

示例代码

cpp

```cpp
#include <atomic>
#include <memory>
#include <iostream>

int main() {
    std::atomic<std::shared_ptr<int>> atomic_ptr{std::make_shared<int>(0)};
    
    // 创建用于比较和交换的 shared_ptr
    std::shared_ptr<int> expected = atomic_ptr.load();
    std::shared_ptr<int> desired = std::make_shared<int>(42);
    
    // 使用 compare_exchange_strong 原子替换 shared_ptr
    if (atomic_ptr.compare_exchange_strong(expected, desired)) {
        std::cout << "交换成功，新值：" << *atomic_ptr.load() << "\n";
    } else {
        std::cout << "交换失败，当前值：" << *atomic_ptr.load() << "\n";
    }
    
    return 0;
}
```

注意事项

- **使用场景**：适用于无锁数据结构，多个线程需要更新共享资源指针（例如配置文件对象）。
- **线程安全**：原子操作确保 shared_ptr 的引用计数和指针更新是安全的，避免数据竞争。
- **性能**：使用 atomic_ptr.is_lock_free() 检查实现是否无锁。如果不是无锁，可能使用内部锁，影响性能。
- **C++20 要求**：此特化仅在 C++20 或更高版本可用。需包含 <memory> 头文件并使用 -std=c++20 编译。

使用 compare_exchange_weak

cpp

```cpp
std::atomic<std::shared_ptr<int>> atomic_ptr{std::make_shared<int>(0)};
std::shared_ptr<int> expected = atomic_ptr.load();
std::shared_ptr<int> desired = std::make_shared<int>(42);

while (!atomic_ptr.compare_exchange_weak(expected, desired)) {
    expected = atomic_ptr.load(); // 比较失败时更新 expected
}
// atomic_ptr 现在指向值为 42 的 shared_ptr<int>
```

使用 compare_exchange_strong

cpp

```cpp
std::atomic<std::shared_ptr<int>> atomic_ptr{std::make_shared<int>(0)};
std::shared_ptr<int> expected = atomic_ptr.load();
std::shared_ptr<int> desired = std::make_shared<int>(42);

if (atomic_ptr.compare_exchange_strong(expected, desired)) {
    std::cout << "交换 succès\n";
} else {
    std::cout << "交换失败\n";
}
```

------

总结

- **compare_exchange_weak**：更快，可能出现伪失败，适合循环。
- **compare_exchange_strong**：可靠，无伪失败，适合单次操作或重试代价高的场景。
- **std::atomic<std::shared_ptr<T>>**：C++20 特性，支持共享指针的原子操作，适用于线程安全的指针更新。
- **最佳实践**：需要保证成功或重试代价高时使用 compare_exchange_strong；性能关键的循环中使用 compare_exchange_weak。对于 std::atomic<std::shared_ptr<T>>，在性能敏感代码中检查 is_lock_free()。

如果有具体使用场景或需要进一步说明，请告诉我！