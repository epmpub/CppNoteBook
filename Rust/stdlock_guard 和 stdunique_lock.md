# std::lock_guard 和 std::unique_lock

在 C++ 中，std::lock_guard 和 std::unique_lock 都是用于管理 std::mutex（或其他互斥量）的 RAII 类，确保锁在作用域结束时自动释放。它们的主要区别在于灵活性、功能和性能开销。以下是两者的详细对比：1. 核心区别

| 特性     | std::lock_guard                        | std::unique_lock                                         |
| -------- | -------------------------------------- | -------------------------------------------------------- |
| 功能     | 简单的 RAII 锁管理，锁定后不可手动解锁 | 更灵活的锁管理，支持手动加锁/解锁、延迟锁定、条件变量等  |
| 性能开销 | 较低，无额外状态管理                   | 略高，因支持额外功能（如状态标志）                       |
| 手动解锁 | 不支持，作用域结束自动解锁             | 支持手动解锁（unlock()）                                 |
| 延迟锁定 | 不支持，必须在构造时锁定               | 支持延迟锁定（std::defer_lock）                          |
| 条件变量 | 不支持与 std::condition_variable 配合  | 支持与 std::condition_variable 配合                      |
| 可转移   | 不支持，锁不可转移                     | 支持，锁的所有权可转移                                   |
| 使用场景 | 简单、确定需要锁整个作用域的场景       | 需要灵活锁管理（如条件变量、部分作用域锁、转移锁）的场景 |

2. 详细说明std::lock_guard

- 定义：std::lock_guard 是一个轻量级的 RAII 类，在构造时锁定互斥量，在析构时自动解锁。
- 特点：
  - 简单性：无需管理锁状态，构造时锁定，作用域结束时解锁。
  - 无额外开销：不维护任何状态，性能最高。
  - 不可手动解锁：一旦锁定，必须等到作用域结束。
  - 不可转移：不能将锁的所有权转移给另一个 std::lock_guard。
- 适用场景：
  - 需要在整个作用域内保护数据的简单场景。
  - 不需要复杂的锁管理逻辑（如手动解锁或条件变量）。
- 局限性：
  - 缺乏灵活性，无法在作用域内提前解锁或延迟锁定。
  - 不支持与 std::condition_variable 配合使用。

示例：

cpp

```cpp
#include <mutex>
#include <thread>
#include <iostream>

std::mutex mtx;
int counter = 0;

void increment() {
    std::lock_guard<std::mutex> lock(mtx); // 构造时锁定
    ++counter;
    std::cout << "Counter: " << counter << std::endl;
    // 作用域结束时自动解锁
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
    return 0;
}
```

- std::lock_guard 确保 counter 的修改是线程安全的，作用域结束时自动解锁。

std::unique_lock

- 定义：std::unique_lock 是一个更灵活的 RAII 类，支持手动加锁/解锁、延迟锁定、锁转移以及与条件变量配合。
- 特点：
  - 灵活性：支持手动解锁（unlock()）、延迟锁定（std::defer_lock）和尝试锁定（try_lock()）。
  - 状态管理：维护锁的状态（是否持有锁），因此有轻微性能开销。
  - 可转移：锁的所有权可以通过 std::move 转移到另一个 std::unique_lock。
  - 条件变量：可以与 std::condition_variable 配合，用于线程同步。
- 适用场景：
  - 需要在作用域内部分时间持有锁（手动解锁）。
  - 使用条件变量（如等待某个条件成立）。
  - 需要转移锁的所有权或尝试非阻塞锁定。
- 局限性：
  - 相比 std::lock_guard，有轻微性能开销（因维护状态）。
  - 使用更复杂，需小心管理锁状态。

示例：

cpp

```cpp
#include <mutex>
#include <thread>
#include <iostream>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
int counter = 0;
bool ready = false;

void increment() {
    std::unique_lock<std::mutex> lock(mtx, std::defer_lock); // 延迟锁定
    lock.lock(); // 手动锁定
    ++counter;
    std::cout << "Counter: " << counter << std::endl;
    lock.unlock(); // 手动解锁
    ready = true;
    cv.notify_one(); // 通知条件变量
}

void wait_for_increment() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return ready; }); // 等待 ready 为 true
    std::cout << "Counter after wait: " << counter << std::endl;
}

int main() {
    std::thread t1(increment);
    std::thread t2(wait_for_increment);
    t1.join();
    t2.join();
    return 0;
}
```

- std::unique_lock 支持延迟锁定、手动解锁和条件变量，适合复杂同步场景。
- 性能对比

- std::lock_guard：
  - 仅管理锁的获取和释放，无额外状态开销。
  - 适合简单场景，性能最佳。
- std::unique_lock：
  - 维护锁状态（是否持有锁），有轻微内存和性能开销。
  - 额外功能（如手动解锁、条件变量）增加了灵活性，但可能影响性能。
- 如何选择

- 用 std::lock_guard：
  - 当你需要在整个作用域内持有锁，且逻辑简单。
  - 不需要与条件变量交互或手动管理锁。
  - 追求最大性能。
- 用 std::unique_lock：
  - 需要手动解锁（例如在作用域内只保护部分代码）。
  - 需要与 std::condition_variable 配合。
  - 需要延迟锁定、尝试锁定或转移锁。
  - 愿意接受轻微性能开销换取灵活性。
- 与 Rust 的对比

- Rust 的 **MutexGuard**（通过 Mutex::lock 返回）类似于 C++ 的 std::lock_guard，但 Rust 的 MutexGuard 自动解引用，提供更便捷的数据访问。
- C++ 的 std::unique_lock 提供了比 Rust MutexGuard 更丰富的功能（如延迟锁定和转移），但需要手动管理。
- Rust 的 RwLock 读写锁场景在 C++ 中对应 std::shared_mutex（C++17），而 std::unique_lock 可以与 std::shared_mutex 配合实现读写锁。
- 注意事项

- 死锁：无论是 std::lock_guard 还是 std::unique_lock，都需要避免死锁（例如多个线程以不同顺序锁定多个互斥量）。
- 异常安全：两者都通过 RAII 确保异常安全，作用域结束时自动解锁。
- C++ 版本：std::lock_guard 和 std::unique_lock 在 C++11 引入，std::shared_mutex（读写锁）需要 C++17。
- 性能优化：对于高性能场景，优先选择 std::lock_guard；对于复杂同步逻辑，选择 std::unique_lock。
- 总结

- std::lock_guard：简单、高效，适合整个作用域需要锁的场景，类似于 Rust 的 MutexGuard。
- std::unique_lock：灵活、功能丰富，适合需要手动管理锁、条件变量或转移锁的场景，性能略低于 std::lock_guard。
- 选择依据：根据是否需要灵活性（手动解锁、条件变量、延迟锁定）选择；简单场景用 std::lock_guard，复杂场景用 std::unique_lock。

如果你有具体场景（例如与条件变量配合或复杂锁管理），可以提供代码或细节，我可以帮你进一步分析或优化！