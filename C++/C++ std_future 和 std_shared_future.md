# std::future 和 std::shared_future 

在 C++ 中，std::future 和 std::shared_future 是标准库中用于异步操作的工具，定义在 <future> 头文件中。它们都表示一个异步计算的结果，可以用来获取线程或任务的返回值或异常。两者的主要区别在于所有权和共享性：std::future 是独占的，而 std::shared_future 是可共享的。以下是对两者的详细解释：

------

std::future

定义

- **头文件**：<future>

- **类模板**：

  cpp

  ```cpp
  template<class T> class future;
  ```

  - T：异步操作返回值的类型（可以是 void）。

- **来源**：

  - 通常由 std::async、std::packaged_task 或 std::promise 创建。

- **行为**：

  - 表示一个独占的未来值，只能由一个对象持有。
  - 一旦通过 .get() 获取值或异常，future 变为无效状态（valid = false）。
  - 不支持拷贝，只支持移动。

- **主要成员函数**：

  1. **get()**：
     - 阻塞等待异步结果，返回值或抛出异常。
     - 调用后 future 失效。
  2. **wait()**：
     - 阻塞等待异步操作完成，不获取结果。
  3. **valid()**：
     - 检查 future 是否有效（是否关联结果）。
  4. **wait_for(duration)** 和 **wait_until(time_point)**：
     - 等待一段时间或直到某个时间点，返回状态（std::future_status）。

示例

cpp

```cpp
#include <iostream>
#include <future>
#include <thread>

int compute() {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return 42;
}

int main() {
    std::future<int> fut = std::async(std::launch::async, compute);
    std::cout << "Waiting...\n";
    int result = fut.get(); // 阻塞直到结果可用
    std::cout << "Result: " << result << "\n";
    // fut.get(); // 再次调用：未定义行为（future已失效）
    return 0;
}
```

- **输出**：

  ```text
  Waiting...
  Result: 42
  ```

- **说明**：fut 是独占的，get() 后无法再次使用。

------

std::shared_future

定义

- **头文件**：<future>

- **类模板**：

  cpp

  ```cpp
  template<class T> class shared_future;
  ```

  - T：异步操作返回值的类型。

- **来源**：

  - 从 std::future 转换而来（通过 .share() 或构造函数）。
  - 可以直接从 std::promise 或 std::packaged_task 创建。

- **行为**：

  - 表示一个可共享的未来值，允许多个对象持有同一结果。
  - 支持拷贝，多个 shared_future 实例共享同一状态。
  - .get() 可以多次调用，返回相同的结果。

- **主要成员函数**：

  - 与 std::future 相同（get(), wait(), valid(), wait_for(), wait_until()）。
  - 区别在于 get() 不使对象失效，且返回的是共享状态的副本。

示例

cpp

```cpp
#include <iostream>
#include <future>
#include <thread>

int compute() {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return 42;
}

int main() {
    std::future<int> fut = std::async(std::launch::async, compute);
    std::shared_future<int> shared_fut = fut.share(); // 从 future 转换为 shared_future

    std::thread t1([shared_fut]() {
        std::cout << "Thread 1: " << shared_fut.get() << "\n";
    });
    std::thread t2([shared_fut]() {
        std::cout << "Thread 2: " << shared_fut.get() << "\n";
    });

    t1.join();
    t2.join();
    std::cout << "Main: " << shared_fut.get() << "\n";
    return 0;
}
```

- **输出**（顺序可能不同）：

  ```text
  Thread 1: 42
  Thread 2: 42
  Main: 42
  ```

- **说明**：多个线程通过 shared_fut 共享同一结果，get() 可多次调用。

------

主要区别

| 特性       | std::future                | std::shared_future       |
| ---------- | -------------------------- | ------------------------ |
| 所有权     | 独占（不可拷贝，只可移动） | 可共享（支持拷贝）       |
| get() 行为 | 调用后失效                 | 可多次调用，返回共享结果 |
| 使用场景   | 单次获取结果               | 多个消费者需要同一结果   |
| 构造方式   | 直接从异步操作生成         | 从 future 转换或直接生成 |
| 线程安全   | 单个对象线程安全           | 多个副本独立使用线程安全 |

------

创建方式对比

1. **std::future**：

   cpp

   ```cpp
   std::future<int> fut = std::async(compute);
   ```

2. **std::shared_future**：

   cpp

   ```cpp
   std::future<int> fut = std::async(compute);
   std::shared_future<int> shared_fut = fut.share(); // 转移所有权
   ```

   或：

   cpp

   ```cpp
   std::promise<int> prom;
   std::shared_future<int> shared_fut = prom.get_future();
   ```

------

使用场景

- **std::future**：
  - 单线程或单消费者需要异步结果。
  - 例如：启动一个任务并等待其完成。
- **std::shared_future**：
  - 多个线程或消费者需要访问同一结果。
  - 例如：广播异步计算结果给多个处理单元。

示例：异常处理

cpp

```cpp
#include <iostream>
#include <future>

void risky_task() {
    throw std::runtime_error("Oops!");
}

int main() {
    std::future<void> fut = std::async(risky_task);
    std::shared_future<void> shared_fut = fut.share();

    try {
        shared_fut.get(); // 抛出异常
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << "\n";
    }

    try {
        shared_fut.get(); // 再次抛出同一异常
    } catch (const std::exception& e) {
        std::cout << "Caught again: " << e.what() << "\n";
    }
    return 0;
}
```

- **输出**：

  ```text
  Caught: Oops!
  Caught again: Oops!
  ```

- **说明**：shared_fut 保留异常状态，可多次访问。

------

注意事项

- **阻塞**：get() 和 wait() 会阻塞直到结果可用。
- **有效性**：
  - std::future 调用 get() 后失效。
  - std::shared_future 始终有效（只要共享状态存在）。
- **异常**：如果异步操作抛出异常，get() 会重新抛出。
- **性能**：
  - std::future 更轻量，适合单次使用。
  - std::shared_future 有共享状态管理开销。

------

总结

- **std::future**：独占的异步结果持有者，适合单消费者场景，简单高效。
- **std::shared_future**：可共享的异步结果，适合多消费者场景，支持多次访问。 两者都是 C++ 异步编程的核心工具，选择取决于结果是否需要共享以及使用模式。