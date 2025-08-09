

#  *std::thread*

```C++
#include <thread>
#include <latch>
#include <print>

std::atomic<int> total{};

int main() {
    // start a thread
    auto t1 = std::thread([]{ ++total; });
    // block until the thread finishes running
    t1.join();
    // total == 1

    std::println("total == {}", total.load());

    auto fn = [](int arg1, double arg2) {
        total += arg1 + arg2;
    };
    // start fn in a new thread with the arguments (42, 3.14)
    auto t2 = std::thread(fn, 42, 3.14);
    // block until the thread finishes running
    t2.join();
    // total == 46

    std::println("total == {}", total.load());

    size_t t_count = 4;
    std::latch work_done(t_count);
    auto work_fn = [&work_done]{
        // do work
        ++total;
        // mark that we are done
        work_done.count_down();
    };

    // Start four detached threads
    for (size_t i = 0; i < t_count; ++i)
        std::thread(work_fn).detach();

    work_done.wait(); // Wait until all threads finish
    // total == 50

    std::println("total == {}", total.load());
}
```

我来详细解释这段 C++ 代码，它展示了多线程编程中 std::thread 和 std::latch 的用法，以及线程安全的原子变量 std::atomic。

------

代码结构

使用的头文件：

- <thread>：提供 std::thread 用于创建和管理线程。
- <latch>：提供 std::latch 用于线程同步（C++20 特性）。
- <print>：提供 std::println 用于格式化输出（C++23 特性）。

全局变量：

- std::atomic<int> total{}：线程安全的原子整数，初始值为 0。

------

1. 单线程操作

代码

cpp

```cpp
auto t1 = std::thread([]{ ++total; });
t1.join();
std::println("total == {}", total.load());
```

解释

- **创建线程**：
  - std::thread 使用 lambda 函数创建一个新线程，执行 ++total（原子自增）。
- **join()**：
  - 主线程等待 t1 完成执行。
  - 完成后，total 从 0 增加到 1。
- **输出**：
  - total.load() 获取原子变量的当前值。
  - 输出：total == 1。
- **特点**：
  - std::atomic 保证线程安全，无需显式锁。
  - join() 确保线程完成后再继续主线程。

------

2. 带参数的线程

代码

cpp

```cpp
auto fn = [](int arg1, double arg2) {
    total += arg1 + arg2;
};
auto t2 = std::thread(fn, 42, 3.14);
t2.join();
std::println("total == {}", total.load());
```

解释

- **定义函数**：
  - fn 是一个 lambda，接受两个参数（int 和 double），将它们相加并累加到 total。
- **创建线程**：
  - std::thread(fn, 42, 3.14) 创建新线程，调用 fn(42, 3.14)。
  - 计算：total += 42 + 3.14，即 1 + 45.14 = 46.14。
  - 由于 total 是 int，小数部分被截断，结果为 46。
- **join()**：
  - 主线程等待 t2 完成。
- **输出**：
  - 输出：total == 46。
- **特点**：
  - std::thread 支持传递任意参数给线程函数。
  - 原子操作确保累加安全。

------

3. 多线程同步（使用 std::latch）

代码

cpp

```cpp
size_t t_count = 4;
std::latch work_done(t_count);
auto work_fn = [&work_done]{
    ++total;
    work_done.count_down();
};
for (size_t i = 0; i < t_count; ++i)
    std::thread(work_fn).detach();
work_done.wait();
std::println("total == {}", total.load());
```

解释

- **初始化**：

  - t_count = 4：计划启动 4 个线程。
  - std::latch work_done(t_count)：创建一个计数值为 4 的闩锁（latch）。
    - std::latch 是 C++20 引入的一次性同步工具，初始计数表示需要等待的事件数。

- **线程函数**：

  - work_fn 是一个 lambda，捕获 work_done 的引用：
    - ++total：原子自增 total。
    - work_done.count_down()：将闩锁计数减 1，表示当前线程完成工作。

- **创建分离线程**：

  cpp

  ```cpp
  for (size_t i = 0; i < t_count; ++i)
      std::thread(work_fn).detach();
  ```

  - 启动 4 个线程执行 work_fn。
  - detach() 使线程与主线程分离，独立运行，无需 join()。
  - 每个线程将 total 增加 1 并调用 count_down()。

- **work_done.wait()**：

  - 主线程等待闩锁计数减到 0。
  - 当 4 个线程都调用 count_down() 后，计数从 4 减到 0，wait() 返回。
  - 此时，total 从 46 增加到 50（每个线程加 1，共加 4）。

- **输出**：

  - 输出：total == 50。

- **特点**：

  - std::latch 提供了一种轻量级的同步机制，适合等待多个线程完成一次性任务。
  - detach() 允许线程独立运行，配合 latch 确保任务完成。

------

输出总结

```text
total == 1
total == 46
total == 50
```

------

关键概念

1. **std::thread**：
   - 创建和管理线程，支持 lambda 和带参数的函数。
   - join() 阻塞等待线程结束，detach() 让线程独立运行。
2. **std::atomic**：
   - 提供线程安全的整数操作，避免数据竞争。
   - load() 获取当前值，+= 是原子操作。
3. **std::latch**（C++20）：
   - 一次性计数器，用于等待多个线程完成。
   - count_down() 减少计数，wait() 阻塞直到计数为 0。
4. **线程同步**：
   - join() 适合单线程等待，latch 适合多线程协调。

------

执行流程

1. t1 增加 total 到 1，主线程等待结束。
2. t2 将 total 从 1 增加到 46，主线程等待结束。
3. 4 个分离线程各增加 total 一次（共加 4），latch 确保主线程等到所有工作完成，total 最终为 50。

这段代码展示了 C++ 多线程编程的基础和高级同步工具，适用于并发任务管理。如果有具体问题或需要更深入分析，请告诉我！