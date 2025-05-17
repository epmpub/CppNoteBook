# C++ std::shared_mutex 读写锁

在 C++ 中，std::shared_mutex 是一种用于并发编程的互斥量类型，定义在 <shared_mutex> 头文件中（C++14 引入，C++17 完善）。它支持 **共享锁（读锁）** 和 **独占锁（写锁）**，适用于读多写少的场景，能够提高并发性能。以下是对 std::shared_mutex 的详细解释，以及它与你之前提到的 std::timed_mutex 和代码上下文的对比。

------

std::shared_mutex 简介

std::shared_mutex 是一种 **读写锁**（reader-writer lock），允许多个线程同时获取共享锁（读操作）以访问共享资源，但写操作需要独占锁，确保同一时刻只有一个线程可以修改资源。这种机制非常适合 **读多写少** 的场景，比如数据库查询、缓存访问或配置管理。

关键特性

1. **两种锁模式**：
   - **共享锁（Shared Lock）**：允许多个线程同时持有，用于只读操作（线程安全地读取共享资源）。
   - **独占锁（Exclusive Lock）**：只有一个线程可以持有，用于写操作（修改共享资源）。
2. **高并发性**：
   - 允许多个读线程同时访问资源，提高读操作的并发性能。
   - 写操作独占资源，确保数据一致性。
3. **RAII 支持**：
   - 兼容 std::lock_guard、std::unique_lock（独占锁）和 std::shared_lock（共享锁，C++14 引入）。
   - 也支持 std::scoped_lock（C++17）用于独占锁。
4. **无超时功能**：
   - 与 std::timed_mutex 不同，std::shared_mutex 不支持超时锁（如 try_lock_for）。如果需要超时，需使用 std::shared_timed_mutex（见后文）。

主要方法

- **独占锁（写锁）**：
  - lock()：获取独占锁，阻塞直到锁可用。
  - try_lock()：尝试获取独占锁，失败立即返回 false。
  - unlock()：释放独占锁。
- **共享锁（读锁）**：
  - lock_shared()：获取共享锁，阻塞直到锁可用（允许多个线程同时持有）。
  - try_lock_shared()：尝试获取共享锁，失败立即返回 false。
  - unlock_shared()：释放共享锁。

------

与 std::timed_mutex 的对比

你的代码中使用了 std::timed_mutex（支持超时锁）和 std::mutex，但未使用 std::shared_mutex。以下是 std::shared_mutex 与 std::timed_mutex 的主要区别：

| 特性     | std::shared_mutex                                            | std::timed_mutex                                    |
| -------- | ------------------------------------------------------------ | --------------------------------------------------- |
| 锁类型   | 共享锁（读）+ 独占锁（写）                                   | 仅独占锁                                            |
| 并发性   | 允许多个读线程同时访问                                       | 一次只允许一个线程访问                              |
| 超时支持 | 无（需用 std::shared_timed_mutex）                           | 有（try_lock_for, try_lock_until）                  |
| 适用场景 | 读多写少（如缓存、配置）                                     | 需要超时控制的独占访问                              |
| 引入版本 | C++14（完善于 C++17）                                        | C++11                                               |
| RAII 锁  | std::lock_guard, std::unique_lock, std::shared_lock, std::scoped_lock | std::lock_guard, std::unique_lock, std::scoped_lock |

相关类型：std::shared_timed_mutex

- C++14 引入了 std::shared_timed_mutex，它是 std::shared_mutex 的超时版本，结合了共享锁和超时功能。
- 额外方法：
  - try_lock_for / try_lock_until（独占锁）。
  - try_lock_for_shared / try_lock_until_shared（共享锁）。
- 如果你需要在读写锁场景中支持超时，std::shared_timed_mutex 是更好的选择。

------

在你的代码中的潜在应用

你的代码使用了 std::mutex（allIssuesMx）和 std::timed_mutex（openIssuesMx）来保护两个 std::vector<std::string>（allIssues 和 openIssues），并通过 std::scoped_lock 同时锁定两者进行写操作：

cpp

```cpp
std::scoped_lock lg(allIssuesMx, openIssuesMx);
allIssues.emplace_back("case1");
openIssues.emplace_back("case2");
```

如何使用 std::shared_mutex

如果 allIssues 或 openIssues 经常被多个线程读取（例如，查询列表内容），但修改（写入）较少，可以用 std::shared_mutex 替换 std::mutex 或 std::timed_mutex，以提高读并发性能。

修改后的代码（使用 std::shared_mutex）

cpp

```cpp
#include <mutex>
#include <shared_mutex>
#include <vector>
#include <string>
#include <iostream>
#include <thread>

int main() {
    std::vector<std::string> allIssues;
    std::shared_mutex allIssuesMx; // 使用 shared_mutex
    std::vector<std::string> openIssues;
    std::shared_mutex openIssuesMx; // 使用 shared_mutex

    // 写操作：需要独占锁
    {
        std::scoped_lock lg(allIssuesMx, openIssuesMx); // 独占锁
        allIssues.emplace_back("case1");
        openIssues.emplace_back("case2");
    }

    // 读操作：多个线程使用共享锁
    std::thread reader1([&] {
        std::shared_lock<std::shared_mutex> sl(allIssuesMx); // 共享锁
        std::cout << "Reader 1: allIssues size = " << allIssues.size() << "\n";
    });

    std::thread reader2([&] {
        std::shared_lock<std::shared_mutex> sl(allIssuesMx); // 共享锁
        std::cout << "Reader 2: allIssues size = " << allIssues.size() << "\n";
    });

    reader1.join();
    reader2.join();

    return 0;
}
```

输出（可能）：

```text
Reader 1: allIssues size = 1
Reader 2: allIssues size = 1
```

代码说明

1. **写操作**：
   - 使用 std::scoped_lock 锁定 allIssuesMx 和 openIssuesMx（作为独占锁），确保写操作（emplace_back）是线程安全的。
   - std::scoped_lock 自动调用 lock()，获取独占锁。
2. **读操作**：
   - 使用 std::shared_lock 获取共享锁，允许多个线程同时读取 allIssues 或 openIssues。
   - std::shared_lock 调用 lock_shared()，允许多个读线程并发执行。
3. **优势**：
   - 读线程（reader1 和 reader2）可以同时访问 allIssues，无需等待彼此，提高了并发性能。
   - 写操作仍保持独占，确保数据一致性。

------

std::shared_mutex 的适用场景

1. **读多写少**：
   - 例如，缓存系统（多线程读取缓存，偶尔更新）、配置文件（频繁读取，少量修改）或数据库索引（多读少写）。
2. **高并发读需求**：
   - 在 Web 服务器中，多个请求线程需要读取共享数据（如用户会话信息），但更新较少。
3. **与容器结合**：
   - 在你的代码中，std::vector 存储字符串列表。如果列表频繁被查询（如检查问题状态），std::shared_mutex 允许多个线程并行读取。

不适用场景

- **写操作频繁**：如果写操作与读操作一样频繁，std::shared_mutex 的共享锁优势不明显，可能不如 std::mutex 简单高效。
- **需要超时**：std::shared_mutex 不支持超时锁。如果需要超时，考虑 std::shared_timed_mutex。
- **单线程或低并发**：如果并发需求低，std::mutex 实现更简单，开销更小。

------

注意事项

1. **性能开销**：
   - std::shared_mutex 比 std::mutex 更复杂，因为它需要管理共享和独占两种锁状态，会有略高的开销。
   - 在读操作远多于写操作时，std::shared_mutex 的并发优势才能体现。
2. **死锁风险**：
   - 如果多个 std::shared_mutex 被同时锁定（如你的代码中的 allIssuesMx 和 openIssuesMx），需确保锁顺序一致，避免死锁。
   - std::scoped_lock 已经帮你处理了多锁的死锁问题，但手动调用 lock() 或 lock_shared() 时需小心。
3. **C++ 版本**：
   - std::shared_mutex 需 C++14 以上，std::shared_lock 和 std::scoped_lock 需 C++17 以上。
   - 如果使用旧编译器，可能需要回退到第三方库（如 Boost）或 std::shared_timed_mutex（C++14）。
4. **与你的代码的兼容性**：
   - 如果 openIssues 和 allIssues 的访问模式是读多写少，替换为 std::shared_mutex 可以提高性能。
   - 如果需要超时功能（如 std::timed_mutex 提供的），则应使用 std::shared_timed_mutex。

------

示例：结合超时（std::shared_timed_mutex）

如果需要在读写锁场景中支持超时，可以使用 std::shared_timed_mutex：

cpp

```cpp
#include <shared_mutex>
#include <vector>
#include <string>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    std::vector<std::string> openIssues;
    std::shared_timed_mutex openIssuesMx;

    // 写线程：长时间持有独占锁
    std::thread writer([&] {
        std::unique_lock<std::shared_timed_mutex> ul(openIssuesMx);
        openIssues.emplace_back("case1");
        std::this_thread::sleep_for(std::chrono::seconds(2));
        std::cout << "Writer: Added case1\n";
    });

    // 读线程：尝试获取共享锁
    std::thread reader([&] {
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        std::shared_lock<std::shared_timed_mutex> sl(openIssuesMx, std::defer_lock);
        if (sl.try_lock_for(std::chrono::milliseconds(500))) {
            std::cout << "Reader: Size = " << openIssues.size() << "\n";
        } else {
            std::cout << "Reader: Timed out\n";
        }
    });

    writer.join();
    reader.join();

    return 0;
}
```

输出（可能）：

```text
Reader: Timed out
Writer: Added case1
```

解释：

- 写线程持有独占锁 2 秒，阻塞读线程。
- 读线程尝试用 try_lock_for 获取共享锁，等待 500ms 后超时，打印失败信息。
- 这展示了 std::shared_timed_mutex 的超时功能，适用于需要读写锁和超时的场景。

------

总结

- **std::shared_mutex** 是一种读写锁，支持共享锁（多线程读）和独占锁（单线程写），适合读多写少的并发场景。
- **与你的代码**：可以将 allIssuesMx 或 openIssuesMx 替换为 std::shared_mutex，以支持多线程并发读取 allIssues 或 openIssues，提高性能。
- **与 std::timed_mutex 的区别**：
  - std::shared_mutex 提供读写锁，但无超时功能。
  - 如果需要超时，需使用 std::shared_timed_mutex。
- **使用建议**：
  - 如果 openIssues 和 allIssues 频繁被读取，std::shared_mutex 是优于 std::timed_mutex 的选择。
  - 如果仍需超时功能，考虑 std::shared_timed_mutex。
  - 对于写频繁或简单场景，std::mutex 可能更合适。
- **关键点**：std::shared_mutex 通过 std::shared_lock（读）和 std::scoped_lock/std::unique_lock（写）管理锁，结合 RAII 确保线程安全和资源释放。