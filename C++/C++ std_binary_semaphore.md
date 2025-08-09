# std::binary_semaphore

在 C++ 中，std::binary_semaphore 是 C++20 引入的一种同步原语，定义在 <semaphore> 头文件中。它是信号量（semaphore）的一种特殊形式，其计数器只能取值 0 或 1，因此得名“二元信号量”。它常用于线程同步，特别是在需要控制对共享资源的访问或协调线程执行顺序的场景中。

以下是对 std::binary_semaphore 的详细解释：

------

定义和基本概念

- **头文件**：<semaphore>
- **类模板**：std::binary_semaphore 实际上是 std::counting_semaphore<1> 的别名，即一个最大计数为 1 的计数信号量。
- **用途**：
  - 实现互斥（类似轻量级的 std::mutex）。
  - 线程间的简单信号传递（类似于条件变量的单次通知）。
- **计数器**：
  - 初始值可以是 0 或 1。
  - 值为 1 表示资源可用或信号已发送。
  - 值为 0 表示资源被占用或信号未发送。

------

主要成员函数

1. **构造函数**：

   cpp

   ```cpp
   std::binary_semaphore sem(1); // 初始值为 1
   ```

   - 参数指定初始计数（只能是 0 或 1）。
   - 抛出 std::system_error 如果初始化失败。

2. **acquire()**：

   - 阻塞地获取信号量，将计数从 1 减到 0。
   - 如果计数已经是 0，则线程会阻塞直到计数变为 1。
   - 用于“锁定”资源或等待信号。

3. **try_acquire()**：

   - 非阻塞地尝试获取信号量。
   - 返回 true 并将计数从 1 减到 0（成功）；返回 false（计数已是 0，失败）。

4. **try_acquire_for(duration)** 和 **try_acquire_until(time_point)**：

   - 带超时的尝试获取。
   - 在指定时间段内或直到指定时间点尝试获取，成功则返回 true，否则返回 false。

5. **release()**：

   - 释放信号量，将计数从 0 增到 1。
   - 如果计数已是 1，则行为未定义（通常会导致异常或错误）。
   - 用于“解锁”资源或发送信号。

------

示例代码

1. 互斥访问共享资源

## 例子 1:

```cpp
#include <iostream>
#include <thread>
#include <semaphore>

std::binary_semaphore sem(1); // 初始值为 1，表示资源可用
int shared_resource = 0;

void worker(int id) {
    sem.acquire(); // 获取信号量，锁定资源
    std::cout << "Thread " << id << " accessing resource\n";
    shared_resource += 1;
    std::cout << "Resource value: " << shared_resource << "\n";
    sem.release(); // 释放信号量，解锁资源
}

int main() {
    std::thread t1(worker, 1);
    std::thread t2(worker, 2);
    t1.join();
    t2.join();
    return 0;
}
```

- **说明**：

  - sem 确保同一时间只有一个线程能访问 shared_resource。

  - 输出是有序的，例如：

    ```text
    Thread 1 accessing resource
    Resource value: 1
    Thread 2 accessing resource
    Resource value: 2
    ```

- 线程间信号传递

cpp

```cpp
#include <iostream>
#include <thread>
#include <semaphore>

std::binary_semaphore signal(0); // 初始值为 0，表示信号未发送

void sender() {
    std::cout << "Sender preparing data...\n";
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Sender sending signal\n";
    signal.release(); // 发送信号
}

void receiver() {
    std::cout << "Receiver waiting for signal...\n";
    signal.acquire(); // 等待信号
    std::cout << "Receiver got signal!\n";
}

int main() {
    std::thread t1(sender);
    std::thread t2(receiver);
    t1.join();
    t2.join();
    return 0;
}
```

- **说明**：

  - receiver 等待 sender 发送信号。

  - 输出示例：

    ```text
    Receiver waiting for signal...
    Sender preparing data...
    Sender sending signal
    Receiver got signal!
    ```

------

与其他同步原语的比较

1. **与 std::mutex 的区别**：
   - std::binary_semaphore 是更轻量级的同步工具，没有所有权概念（不像 std::mutex 必须由锁定它的线程解锁）。
   - 可以跨线程释放（适合信号传递场景）。
   - 不支持递归锁定。
2. **与 std::condition_variable 的区别**：
   - 更简单直接，适合单次信号传递。
   - 不需要配合 std::mutex 使用。
   - 不支持等待谓词（predicate）。
3. **与 std::counting_semaphore 的关系**：
   - std::binary_semaphore 是 std::counting_semaphore<1> 的特化。
   - 如果需要计数大于 1 的信号量，可以使用 std::counting_semaphore<N>。

------

注意事项

- **线程安全**：acquire() 和 release() 是线程安全的，但同一个信号量的多次 release()（导致计数超过 1）是未定义行为。
- **性能**：相比 std::mutex，它更轻量，但功能较少。
- **适用场景**：适合简单的互斥或单次信号传递；复杂场景可能需要其他工具。

------

总结

std::binary_semaphore 是 C++20 提供的一个简单而高效的同步工具，特别适合需要轻量级互斥或线程间信号传递的场景。通过 acquire() 和 release()，它可以轻松实现资源保护或线程协调，是现代 C++ 多线程编程中的一个实用补充。



## 例子 2:

The C++20 *std::binary_semaphore* is a specialization of the more general *std::counting_semaphore* that only supports two values, 0 and 1.

The main use case of a binary semaphore is for simple signalling, where the alternative approach would be to use the combination of *std::mutex*, *std::condition_variable* and a boolean variable.

```C++
#include <thread>
#include <semaphore>
#include <condition_variable>

{
std::binary_semaphore signal(0);
auto t = std::jthread([&signal]() {
    // Wait until this thread is signaled
    signal.acquire();
    // Actual code...
});

// Signal the thread to run
signal.release();
}

{
// Example of how this would look with a condition variable:
std::mutex mux;
std::condition_variable cond;
bool received = false;

auto t = std::jthread([&mux, &cond, &received]() {
    // Wait until this thread is signaled
    std::unique_lock lock(mux);
    cond.wait(lock, [&received]{ return received; });
    // Actual code...
});

{ // Signal the thread to run
    std::unique_lock lock(mux);
    received = true;
}
cond.notify_one();
}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/444vWT94f)