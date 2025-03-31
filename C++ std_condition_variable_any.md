# std::condition_variable_any

std::condition_variable_any 是 C++ 标准库中提供的一种条件变量类，定义在 <condition_variable> 头文件中。它是 C++11 引入的，与 std::condition_variable 类似，用于线程间的同步，但它更通用，因为它可以与任何满足“基本锁”（BasicLockable）要求的锁类型一起使用，而不仅仅局限于 std::mutex。

以下是对 std::condition_variable_any 的详细解释：

------

定义

cpp

```cpp
#include <condition_variable>

std::condition_variable_any cv;
```

------

与 std::condition_variable 的区别

| 特性         | std::condition_variable              | std::condition_variable_any                                  |
| ------------ | ------------------------------------ | ------------------------------------------------------------ |
| 支持的锁类型 | 仅 std::mutex                        | 任何满足 BasicLockable 的锁（如 std::mutex, std::recursive_mutex, 自定义锁） |
| 性能         | 通常更高效（因专为 std::mutex 优化） | 稍低（因需要通用性）                                         |
| 使用场景     | 简单线程同步                         | 需要灵活锁类型的复杂场景                                     |

- **BasicLockable** 要求锁类型具有 lock() 和 unlock() 成员函数。

------

主要成员函数

1. **wait(Lock& lock)**  

   - 阻塞当前线程，等待条件变量被通知。
   - 在等待前自动释放 lock，在被唤醒后重新锁定 lock。
   - 需要搭配一个锁对象（如 std::unique_lock）。

2. **wait(Lock& lock, Predicate pred)**  

   - 等待直到 pred() 返回 true，避免虚假唤醒（spurious wakeup）。

   - 等价于：

     cpp

     ```cpp
     while (!pred()) wait(lock);
     ```

3. **notify_one()**  

   - 唤醒一个等待中的线程（如果有）。

4. **notify_all()**  

   - 唤醒所有等待中的线程。

5. **wait_for(Lock& lock, Duration dur)**  

   - 等待一段时间（dur），超时后返回。
   - 返回 std::cv_status::timeout（超时）或 std::cv_status::no_timeout（被通知）。

6. **wait_until(Lock& lock, TimePoint tp)**  

   - 等待直到某个绝对时间点（tp）。

这些函数都有带谓词的重载版本（如 wait_for(lock, dur, pred)）。

------

示例代码

示例 1：基本使用

cpp

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::condition_variable_any cv;
std::mutex mtx;
bool ready = false;

void worker() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return ready; }); // 等待 ready 变为 true
    std::cout << "Worker thread: 数据已准备好\n";
}

int main() {
    std::thread t(worker);

    std::this_thread::sleep_for(std::chrono::seconds(1)); // 模拟准备工作
    {
        std::unique_lock<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one(); // 通知等待线程

    t.join();
    return 0;
}
```

输出

```text
Worker thread: 数据已准备好
```

示例 2：使用 std::recursive_mutex

std::condition_variable_any 的优势在于支持非 std::mutex 的锁，例如 std::recursive_mutex：

cpp

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::condition_variable_any cv;
std::recursive_mutex mtx;
int counter = 0;

void recursive_wait(int depth) {
    std::unique_lock<std::recursive_mutex> lock(mtx);
    if (depth > 0) {
        recursive_wait(depth - 1); // 递归加锁
    } else {
        cv.wait(lock, [] { return counter >= 3; });
        std::cout << "Counter reached: " << counter << "\n";
    }
}

void incrementer() {
    for (int i = 0; i < 5; ++i) {
        {
            std::unique_lock<std::recursive_mutex> lock(mtx);
            ++counter;
        }
        cv.notify_one();
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

int main() {
    std::thread t1(recursive_wait, 2);
    std::thread t2(incrementer);

    t1.join();
    t2.join();
    return 0;
}
```

输出

```text
Counter reached: 3
```

解释

- std::recursive_mutex 允许多次锁定，std::condition_variable_any 与之兼容。
- 如果使用 std::condition_variable，会因 std::mutex 不支持递归而失败。

------

使用场景

1. **自定义锁类型**：
   - 当使用第三方库的锁或自定义锁类型时，std::condition_variable_any 是唯一选择。
2. **递归锁**：
   - 与 std::recursive_mutex 配合，用于递归调用中需要条件等待的场景。
3. **复杂同步**：
   - 在需要灵活锁机制的线程同步中。

------

注意事项

1. **锁要求**：
   - 锁必须满足 BasicLockable 接口（lock() 和 unlock()）。
   - 通常搭配 std::unique_lock 使用，因为它支持灵活的锁管理。
2. **虚假唤醒**：
   - 条件变量可能被意外唤醒，总是使用带谓词的 wait 版本以确保条件成立。
3. **性能**：
   - 比 std::condition_variable 稍慢，因为它需要支持通用锁类型。
4. **异常安全**：
   - 如果锁操作抛出异常，wait 会传播该异常。

------

与 std::condition_variable 的选择

- **优先使用 std::condition_variable**：
  - 如果只使用 std::mutex，它是更轻量和优化的选择。
- **选择 std::condition_variable_any**：
  - 当需要支持其他锁类型（如 std::recursive_mutex）或未来可能扩展锁类型时。

------

总结

std::condition_variable_any 是 C++ 中一个更通用的条件变量工具，支持任意 BasicLockable 锁类型。它在功能上与 std::condition_variable 类似，但提供了更大的灵活性，适用于复杂同步场景。搭配 std::unique_lock 和谓词使用，可以安全高效地实现线程间通信。

如果你有具体问题或使用场景，欢迎进一步讨论！