# std::recursive_mutex递归锁 

std::recursive_mutex 是 C++ 标准库中提供的一种互斥锁（mutex），定义在 <mutex> 头文件中。它是 C++11 引入的，用于多线程编程。与普通的 std::mutex 不同，std::recursive_mutex 允许同一个线程多次锁定它，而不会引发死锁。这种特性使其适用于需要递归调用的场景。

以下是对 std::recursive_mutex 的详细解释：

------

什么是 std::recursive_mutex？

std::recursive_mutex 是一种**递归互斥锁**，也称为**可重入互斥锁**。它的核心特点是：

- 同一个线程可以多次调用 lock() 或 try_lock() 来锁定它，而不会被阻塞。
- 每次锁定都会增加一个内部计数器。
- 线程必须调用相同次数的 unlock() 来完全释放锁，计数器才会归零，允许其他线程获取锁。

相比之下，普通的 std::mutex 不支持递归锁定，如果同一个线程试图多次锁定它，会导致未定义行为（通常是死锁）。

------

头文件和定义

cpp

```cpp
#include <mutex>

std::recursive_mutex mtx;
```

------

主要成员函数

1. **lock()**
   - 锁定互斥锁。如果锁已经被其他线程持有，当前线程会阻塞。
   - 如果当前线程已经持有锁，则计数器加 1，不会阻塞。
2. **unlock()**
   - 释放一次锁，计数器减 1。
   - 只有当计数器减到 0 时，锁才真正被释放，其他线程才能获取。
3. **try_lock()**
   - 尝试锁定互斥锁。如果锁可用，立即锁定并返回 true；如果锁不可用，返回 false。
   - 如果当前线程已经持有锁，则计数器加 1，返回 true。

------

示例代码

以下是一个使用 std::recursive_mutex 的例子，展示递归函数中的锁使用：

cpp

```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::recursive_mutex mtx;

void recursive_function(int depth) {
    std::lock_guard<std::recursive_mutex> lock(mtx); // 自动管理锁
    std::cout << "Depth: " << depth << std::endl;
    if (depth > 0) {
        recursive_function(depth - 1); // 递归调用
    }
}

int main() {
    std::thread t1(recursive_function, 3);
    std::thread t2(recursive_function, 2);

    t1.join();
    t2.join();

    return 0;
}
```

输出（可能顺序不同）

```text
Depth: 3
Depth: 2
Depth: 1
Depth: 0
Depth: 2
Depth: 1
Depth: 0
```

解释

- recursive_function 在每次调用时都使用 std::lock_guard 锁定 mtx。
- 因为 mtx 是 std::recursive_mutex，同一个线程可以在递归调用中多次锁定它。
- 如果使用普通的 std::mutex，程序会死锁，因为第二次锁定会阻塞当前线程。

------

与 std::mutex 的区别

| 特性               | std::mutex             | std::recursive_mutex |
| ------------------ | ---------------------- | -------------------- |
| 同一个线程多次锁定 | 未定义行为（通常死锁） | 允许，计数器递增     |
| 性能               | 更高（更轻量）         | 稍低（需维护计数器） |
| 使用场景           | 简单互斥               | 递归或复杂调用链     |

------

使用场景

1. **递归函数**：
   - 当一个函数可能直接或间接调用自身，并且需要保护共享资源时。
2. **复杂调用链**：
   - 当一个锁定的函数调用另一个需要相同锁的函数时，避免手动解锁和重新加锁的麻烦。
3. **调试或临时解决方案**：
   - 在不清楚代码调用路径的情况下，std::recursive_mutex 可以作为安全的选择（但应尽量优化为非递归设计）。

------

注意事项

1. **性能开销**：
   - 相比 std::mutex，std::recursive_mutex 维护计数器会带来额外的开销，应避免滥用。
2. **设计问题**：
   - 使用递归锁可能掩盖代码中的逻辑问题。理想情况下，应尽量设计非递归的线程安全代码。
3. **必须匹配解锁**：
   - 每次 lock() 必须对应一次 unlock()，否则会导致资源未释放。
4. **异常安全**：
   - 推荐使用 std::lock_guard 或 std::unique_lock 管理锁，确保异常情况下也能正确解锁。

------

结合 RAII 的示例

使用 std::unique_lock 提供更灵活的管理：

cpp

```cpp
#include <iostream>
#include <mutex>

std::recursive_mutex mtx;

void recursive_print(int n) {
    std::unique_lock<std::recursive_mutex> lock(mtx);
    std::cout << n << " ";
    if (n > 0) {
        recursive_print(n - 1);
    }
    // lock 在离开作用域时自动解锁
}

int main() {
    recursive_print(3);
    std::cout << std::endl;
    return 0;
}
```

输出

```text
3 2 1 0
```

------

总结

std::recursive_mutex 是 C++ 中处理递归锁需求的工具，适合在同一个线程需要多次获取同一锁的场景。虽然功能强大，但应谨慎使用，因为它可能隐藏设计问题并带来性能开销。在大多数情况下，优先考虑非递归的线程安全设计，并搭配普通的 std::mutex。

如果你有具体的使用场景或问题，欢迎进一步讨论！