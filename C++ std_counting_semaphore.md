# std::counting_semaphore

```C++
#include <deque>
#include <mutex>
#include <semaphore>
#include <concepts>
#include <utility>
#include <thread>
#include <chrono>
#include <print>

struct Data {};

// Simple unbounded thread-safe many<->many producer/consumer queue
struct WorkQueue {
    WorkQueue() : queue_{}, mux_{}, sem_{0} {}
    void push(std::convertible_to<Data> auto&& data) {
        // Push a new element into the queue
        {
            auto _ = std::lock_guard(mux_);
            queue_.push_back(std::forward<decltype(data)>(data));
        }
        // Atomically increase the counter in the semaphore.
        // If any threads are blocked on acquire, they will be notified.
        sem_.release();
    }
    Data pop() {
        // Try to atomically decrease the counter in the semaphore.
        // If the counter is already 0, blocks.
        sem_.acquire();

        // At this point we are guaranteed available data,
        // still need to synchronize against other consumers.
        auto _ = std::lock_guard(mux_);
        Data result = std::move(queue_.front());
        queue_.pop_front();
        return result;
    }
private:
    std::deque<Data> queue_;
    std::mutex mux_;
    std::counting_semaphore<> sem_;
};

int main() {
    using namespace std::chrono;
    WorkQueue q;

    auto producer = std::jthread{[&q]{
        std::this_thread::sleep_for(200ms);
        std::println("Producer: publishing data");
        q.push(Data{});
        std::this_thread::sleep_for(200ms);
        std::println("Producer: publishing data");
        q.push(Data{});
        std::println("Producer: publishing data");
        q.push(Data{});
        std::println("Producer: publishing data");
        q.push(Data{});
    }};
    auto consumer1 = std::jthread{[&q]{
        std::println("Consumer1: attempting to read data");
        auto _ = q.pop();
        std::println("Consumer1: succeeded in reading data");
        std::println("Consumer1: attempting to read data");
        auto _ = q.pop();
        std::println("Consumer1: succeeded in reading data");
    }};
    auto consumer2 = std::jthread{[&q]{
        std::println("Consumer2: attempting to read data");
        auto _ = q.pop();
        std::println("Consumer2: succeeded in reading data");
        std::println("Consumer2: attempting to read data");
        auto _ = q.pop();
        std::println("Consumer2: succeeded in reading data");
    }};
}
```

The C++20 synchronization primitive for limiting concurrent access to a resource: std::counting_semaphore.



The C++20 *std::counting_semaphore* is a synchronization primitive that can limit the number of concurrent threads accessing a shared resource.

The typical use cases overlap with *std::condition_variable*; however, *std::counting_semaphore* offers a more straightforward interface and potentially better performance.



这段代码实现了一个线程安全的无界多生产者-多消费者队列（WorkQueue），使用 C++20 的新特性（如 std::jthread 和 std::counting_semaphore）以及标准库组件（如 std::deque 和 std::mutex）。它展示了如何在多线程环境中安全地生产和消费数据。以下是对代码的逐步解释：

------

1. Data 结构体

cpp

```cpp
struct Data {};
```

- **用途**：一个简单的空结构体，表示队列中传递的数据。
- **说明**：这里为了演示而定义为空，实际使用中可以包含具体数据。

------

2. WorkQueue 类

cpp

```cpp
struct WorkQueue {
    WorkQueue() : queue_{}, mux_{}, sem_{0} {}
    // ... 其他成员函数 ...
private:
    std::deque<Data> queue_;
    std::mutex mux_;
    std::counting_semaphore<> sem_;
};
```

- **用途**：实现一个线程安全的队列，支持多个生产者和消费者。
- **成员**：
  - queue_：std::deque<Data>，存储数据的双端队列，支持从前端弹出（pop_front）和后端插入（push_back）。
  - mux_：std::mutex，保护队列的访问，确保线程安全。
  - sem_：std::counting_semaphore<>，初始值为 0，用于同步生产者和消费者，跟踪队列中可用数据的数量。

------

push 方法

cpp

```cpp
void push(std::convertible_to<Data> auto&& data) {
    {
        auto _ = std::lock_guard(mux_);
        queue_.push_back(std::forward<decltype(data)>(data));
    }
    sem_.release();
}
```

- **用途**：向队列中添加一个新元素。
- **参数**：
  - std::convertible_to<Data> auto&& data：使用 C++20 的概念（concept），接受任何可转换为 Data 类型的右值或左值。
  - std::forward：完美转发参数，保留其值类别（左值或右值）。
- **执行步骤**：
  1. 使用 std::lock_guard 锁定互斥锁 mux_，确保对 queue_ 的访问是线程安全的。
  2. 将数据移动或复制到队列尾部（push_back）。
  3. 锁释放后，调用 sem_.release()，将信号量的计数器原子性加 1。
- **信号量作用**：
  - 增加计数器表示队列中多了一个可用元素。
  - 如果有消费者线程在 sem_.acquire() 上阻塞，它们会被唤醒。

------

pop 方法

cpp

```cpp
Data pop() {
    sem_.acquire();
    auto _ = std::lock_guard(mux_);
    Data result = std::move(queue_.front());
    queue_.pop_front();
    return result;
}
```

- **用途**：从队列中取出一个元素。
- **执行步骤**：
  1. sem_.acquire()：尝试将信号量计数器减 1。
     - 如果计数器为 0，则阻塞，直到生产者通过 release() 增加计数器。
     - 成功减 1 表示队列中至少有一个元素可用。
  2. 使用 std::lock_guard 锁定 mux_，防止其他消费者同时访问队列。
  3. 将队列前端的元素移动到 result（std::move(queue_.front())）。
  4. 从队列前端移除该元素（pop_front）。
  5. 返回 result。
- **线程安全**：
  - 信号量确保消费者只有在数据可用时才继续。
  - 互斥锁确保多个消费者不会同时操作队列。

------

3. main 函数

cpp

```cpp
int main() {
    using namespace std::chrono;
    WorkQueue q;

    auto producer = std::jthread{[&q]{
        std::this_thread::sleep_for(200ms);
        std::println("Producer: publishing data");
        q.push(Data{});
        std::this_thread::sleep_for(200ms);
        std::println("Producer: publishing data");
        q.push(Data{});
        std::println("Producer: publishing data");
        q.push(Data{});
        std::println("Producer: publishing data");
        q.push(Data{});
    }};
    auto consumer1 = std::jthread{[&q]{
        std::println("Consumer1: attempting to read data");
        auto _ = q.pop();
        std::println("Consumer1: succeeded in reading data");
        std::println("Consumer1: attempting to read data");
        auto _ = q.pop();
        std::println("Consumer1: succeeded in reading data");
    }};
    auto consumer2 = std::jthread{[&q]{
        std::println("Consumer2: attempting to read data");
        auto _ = q.pop();
        std::println("Consumer2: succeeded in reading data");
        std::println("Consumer2: attempting to read data");
        auto _ = q.pop();
        std::println("Consumer2: succeeded in reading data");
    }};
}
```

- **用途**：创建并测试 WorkQueue，模拟一个生产者和两个消费者的场景。

- **线程**：

  - producer：一个生产者线程（std::jthread），每隔 200 毫秒生产一个 Data 对象，总共生产 4 个。
  - consumer1 和 consumer2：两个消费者线程，各消费 2 个数据。

- **执行流程**：

  1. 生产者启动后等待 200ms，消费者可能先尝试读取但会阻塞（因队列为空，信号量为 0）。
  2. 生产者推送第一个数据，信号量增至 1，一个消费者被唤醒并消费数据。
  3. 此过程重复，生产者总共推送 4 个数据，两个消费者各消费 2 个。

- **输出示例**（可能因线程调度而异）：

  ```text
  Consumer1: attempting to read data
  Consumer2: attempting to read data
  Producer: publishing data
  Consumer1: succeeded in reading data
  Consumer1: attempting to read data
  Producer: publishing data
  Consumer2: succeeded in reading data
  Consumer2: attempting to read data
  Producer: publishing data
  Consumer1: succeeded in reading data
  Producer: publishing data
  Consumer2: succeeded in reading data
  ```

- **特性**：

  - std::jthread：C++20 引入，自动在析构时加入（join）线程，无需手动管理。
  - std::println：C++23 提议的格式化输出函数，这里假设可用。

------

关键点

1. **线程安全**：
   - std::mutex 保护队列的并发访问。
   - std::counting_semaphore 同步生产者和消费者，避免消费者在队列为空时访问。
2. **无界队列**：
   - 使用 std::deque，理论上容量无上限（受内存限制）。
3. **多生产者-多消费者**：
   - 设计支持多个线程同时调用 push 和 pop，通过互斥锁和信号量协调。
4. **性能**：
   - 信号量提供高效的等待机制，避免忙等待（busy waiting）。
   - std::deque 适合双端操作，但若只需单端操作，std::queue 也可替代。

------

总结

WorkQueue 是一个简单但功能完整的线程安全队列实现，结合了 C++20/23 的现代特性。它展示了如何使用信号量和互斥锁协调多线程数据传递，适用于任务调度、消息队列等场景。代码中的生产者-消费者示例清晰地演示了其工作原理和线程同步机制。