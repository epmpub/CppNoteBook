# std::osyncstream std::ostream 的 C++20 同步线程安全包装器

std::ostream 的 C++20 同步线程安全包装器：std::osyncstream。

C++20 引入了同步流。

只要对该流的所有访问都是通过同步流进行的，就可以使用多个同步流写入单个目标流，而不会引入数据竞争或交错。

```C++
	std::vector<std::thread> threads;
	std::mutex mtx; // Mutex for thread synchronization
	for (size_t i = 0; i < 10; i++)
	{
		threads.push_back(std::thread([i, &mtx]() {
			{
				std::lock_guard<std::mutex> lock(mtx); // Lock the mutex
				std::cout << "Thread " << i << " is executing.\n";
			}
			std::this_thread::sleep_for(std::chrono::milliseconds(1000)); // Simulate work
			{
				std::lock_guard<std::mutex> lock(mtx); // Lock the mutex
				std::cout << "Thread " << i << " has finished execution.\n";
			}

			}));
	}

	for (auto& thread : threads)
	{
		thread.join();
	}
	std::cout << "All threads have finished execution.\n";

```



```C++
	std::vector<std::jthread> threads;

	for (size_t i = 0; i < 10; i++)
	{
		threads.emplace_back(std::jthread([i]() {
			std::osyncstream(std::cout) << "Thread " << i << " is executing.\n";
			std::this_thread::sleep_for(std::chrono::milliseconds(1000)); // Simulate work
			std::osyncstream(std::cout) << "Thread " << i << " has finished execution.\n";
			}));
	}
```



```c++
#include <syncstream>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    // Safe output to the same stream (here it is std::cout)
    // without introducing data races.
    auto j1 = std::jthread([](){
        std::osyncstream(std::cout) << 
            "We can safely write to the same stream.\n";
    });
    auto j2 = std::jthread([](){
        std::osyncstream(std::cout) << 
            "We can safely write to the same stream.\n";
    });

    // No interleaving, data are sent to std::cout 
    // when osyncstream is destroyed.
    auto j3 = std::jthread([]() {
        {
            std::osyncstream out(std::cout);
            out << "This ";
            out << "will ";
            out << "not ";
            out << "be ";
            out << "interleaved.\n";
        }
        using namespace std::chrono_literals;
        std::this_thread::sleep_for(2000ms);
        {
            std::osyncstream(std::cout) <<
            "Maybe someone said something before me.\n";
        }
    });
    auto j4 = std::jthread([]() {
        std::osyncstream(std::cout) << "Hey!\n";
    });
}
```

这段代码展示了 C++20 中引入的 <syncstream> 头文件和 std::osyncstream 类，用于在多线程环境中安全地向同一输出流（例如 std::cout）写入数据，避免数据竞争和输出交错。以下是逐步解释。

------

代码概览

- 使用 std::jthread 创建多个线程，向 std::cout 输出文本。
- 使用 std::osyncstream 包装 std::cout，确保线程安全的输出。
- 展示了如何避免多线程输出时的交错（interleaving）问题。

------

关键组件

1. **头文件**

cpp

```cpp
#include <syncstream>
#include <iostream>
#include <thread>
#include <chrono>
```

- <syncstream>：提供 std::osyncstream，用于线程安全的流输出。
- <iostream>：提供 std::cout。
- <thread>：提供 std::jthread（C++20），支持线程管理。
- <chrono>：提供时间工具，如 std::chrono_literals 用于延时。
- **基本线程安全输出**

cpp

```cpp
auto j1 = std::jthread([](){
    std::osyncstream(std::cout) << 
        "We can safely write to the same stream.\n";
});
auto j2 = std::jthread([](){
    std::osyncstream(std::cout) << 
        "We can safely write to the same stream.\n";
});
```

- **std::jthread**：

  - C++20 引入的线程类，自动在析构时加入（join）线程。
  - 这里创建两个线程 j1 和 j2，分别执行 lambda 函数。

- **std::osyncstream**：

  - 构造函数接受一个输出流（如 std::cout），创建一个同步流对象。
  - 确保多个线程向同一流写入时不会发生数据竞争。

- **行为**：

  - 两个线程同时向 std::cout 输出相同文本。

  - 普通情况下，直接使用 std::cout << ... 可能导致输出交错（如 "We can saWe can safely..."）。

  - 使用 std::osyncstream，每个线程的输出是原子性的，完整输出为：

    ```text
    We can safely write to the same stream.
    We can safely write to the same stream.
    ```

  - 输出顺序不确定（取决于线程调度），但不会交错。

- **销毁时写入**：

  - std::osyncstream 在析构时（lambda 结束）将缓冲区内容一次性写入 std::cout。

- **避免交错的复杂输出**

cpp

```cpp
auto j3 = std::jthread([]() {
    {
        std::osyncstream out(std::cout);
        out << "This ";
        out << "will ";
        out << "not ";
        out << "be ";
        out << "interleaved.\n";
    }
    using namespace std::chrono_literals;
    std::this_thread::sleep_for(2000ms);
    {
        std::osyncstream(std::cout) <<
            "Maybe someone said something before me.\n";
    }
});
```

- **线程 j3**：
  - 分为两个部分：
    1. **第一部分**：
       - 创建 std::osyncstream out 对象，多次写入文本。
       - 每次 out << 将数据追加到内部缓冲区。
       - 作用域结束时，out 析构，缓冲区内容一次性写入 std::cout。
       - 输出：This will not be interleaved.（完整、无交错）。
    2. **延时**：
       - std::this_thread::sleep_for(2000ms) 暂停 2 秒。
    3. **第二部分**：
       - 创建另一个 std::osyncstream，输出新文本。
       - 输出：Maybe someone said something before me.。
- **关键点**：
  - std::osyncstream 确保每个对象生命周期内的输出是原子的。
  - 两次输出的顺序取决于线程调度，但每条消息完整。
- **另一个线程**

cpp

```cpp
auto j4 = std::jthread([]() {
    std::osyncstream(std::cout) << "Hey!\n";
});
```

- **线程 j4**：
  - 简单地输出 "Hey!"。
  - 同样使用 std::osyncstream，保证原子性。

------

为什么这样工作？

1. **std::osyncstream 的机制**：
   - 内部维护一个缓冲区，所有写入操作（如 <<）都追加到缓冲区。
   - 在对象析构时，缓冲区内容一次性写入目标流（std::cout），并通过同步机制（可能是互斥锁）确保线程安全。
   - 这避免了直接操作 std::cout 时可能的数据竞争。
2. **无交错**：
   - 普通 std::cout 在多线程中可能导致字符级别的交错。
   - std::osyncstream 将输出延迟到析构，确保整块输出完整。
3. **std::jthread**：
   - 自动管理线程生命周期，主线程结束时会等待所有线程完成。

------

输出示例

运行时可能的输出（顺序随机，但每行完整）：

```text
We can safely write to the same stream.
We can safely write to the same stream.
This will not be interleaved.
Hey!
Maybe someone said something before me.
```

- **顺序不确定**：线程调度决定哪行先输出。
- **无交错**：每行作为一个整体输出。

------

使用场景

- **多线程日志**：
  - 在多线程程序中向控制台或文件安全写入日志。
- **调试输出**：
  - 确保调试信息在并发环境中清晰可读。
- **性能考虑**：
  - std::osyncstream 引入同步开销，适合低频输出；高频场景可能需要其他机制（如缓冲队列）。

------

总结

- std::osyncstream 是 C++20 的工具，用于线程安全的流输出。
- 它通过缓冲和析构时写入，避免多线程输出交错。
- 代码通过 std::jthread 展示了其在多线程环境中的用法。
- 每个 std::osyncstream 对象的输出是原子的，保证完整性。