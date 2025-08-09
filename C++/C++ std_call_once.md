# std::call_once

std::call_once 是 C++ 标准库中提供的一个线程同步工具，定义在 <mutex> 头文件中。它与 std::once_flag 一起使用，确保某个函数在多线程环境中只被调用一次。C++11 引入了这个功能，用于实现线程安全的单次初始化（lazy initialization）。

以下是对 std::call_once 的详细解释：

------

定义

cpp

```cpp
#include <mutex>

void call_once(std::once_flag& flag, Callable&& f, Args&&... args);
```

- **flag**: 一个 std::once_flag 对象，用于跟踪函数是否已被调用。
- **f**: 要执行的函数或可调用对象。
- **args**: 传递给 f 的参数。
- **返回值**: 无（void）。

------

std::once_flag

- std::once_flag 是一个轻量级的、非拷贝的标志类，表示某个操作是否已经执行。
- 它必须在调用 std::call_once 前定义，并且通常是静态或全局变量。

------

行为

- std::call_once 保证在多个线程同时调用时，f 只会被执行一次。
- 第一个调用 std::call_once 的线程执行 f，其他线程会阻塞，直到该调用完成。
- 如果 f 在执行时抛出异常，异常会传播给调用线程，且 flag 保持未完成状态，允许后续调用再次尝试。

------

示例代码

示例 1：基本使用

cpp

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::once_flag flag;

void init() {
    std::cout << "初始化只执行一次\n";
}

void worker() {
    std::call_once(flag, init);
}

int main() {
    std::thread t1(worker);
    std::thread t2(worker);
    std::thread t3(worker);

    t1.join();
    t2.join();
    t3.join();
    return 0;
}
```

输出

```text
初始化只执行一次
```

示例 2：带参数的初始化

cpp

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::once_flag flag;
int shared_value;

void init_value(int x) {
    shared_value = x;
    std::cout << "设置 shared_value 为 " << x << "\n";
}

void worker(int value) {
    std::call_once(flag, init_value, value);
    std::cout << "线程读取 shared_value: " << shared_value << "\n";
}

int main() {
    std::thread t1(worker, 42);
    std::thread t2(worker, 99);
    std::thread t3(worker, 7);

    t1.join();
    t2.join();
    t3.join();
    return 0;
}
```

输出（示例）

```text
设置 shared_value 为 42
线程读取 shared_value: 42
线程读取 shared_value: 42
线程读取 shared_value: 42
```

- 只有第一个线程的参数（例如 42）生效。

**示例 3：异常处理**

cpp

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::once_flag flag;

void risky_init() {
    std::cout << "尝试初始化\n";
    throw std::runtime_error("初始化失败");
}

void worker() {
    try {
        std::call_once(flag, risky_init);
    } catch (const std::exception& e) {
        std::cout << "捕获异常: " << e.what() << "\n";
    }
}

int main() {
    std::thread t1(worker);
    std::thread t2(worker);

    t1.join();
    t2.join();
    return 0;
}
```

输出（示例）

```text
尝试初始化
捕获异常: 初始化失败
尝试初始化
捕获异常: 初始化失败
```

- 异常导致初始化失败，后续线程会再次尝试。

------

使用场景

1. **单例模式（Singleton）**：

   - 确保单例对象只初始化一次。

   cpp

   ```cpp
   class Singleton {
   public:
       static Singleton& getInstance() {
           std::call_once(flag, [] { instance = new Singleton(); });
           return *instance;
       }
   private:
       Singleton() {}
       static std::once_flag flag;
       static Singleton* instance;
   };
   std::once_flag Singleton::flag;
   Singleton* Singleton::instance = nullptr;
   ```

2. **延迟初始化**：

   - 在多线程环境中懒惰地初始化共享资源。

   cpp

   ```cpp
   std::once_flag flag;
   std::vector<int> data;
   void init_data() { data = {1, 2, 3}; }
   void use_data() { std::call_once(flag, init_data); }
   ```

3. **一次性配置**：

   - 确保全局配置只设置一次。

------

时间复杂度与性能

- **初次调用**：取决于 f 的复杂度，可能涉及锁开销。
- **后续调用**：接近 O(1)，只需检查 flag 状态。
- 内部实现通常使用原子操作或互斥锁来保证线程安全。

------

注意事项

1. **线程安全**：
   - std::call_once 本身是线程安全的，但 f 的执行需要自行确保安全。
2. **异常处理**：
   - 如果 f 抛出异常，flag 保持未完成状态，后续调用会再次尝试。
3. **不可重用**：
   - 一个 std::once_flag 一旦完成（f 成功执行），无法重置。
4. **与静态初始化比较**：
   - 对于简单常量，static 变量可能是更好的选择（如 static const int x = 42;），但 std::call_once 适用于动态初始化。

------

与其他同步工具的对比

| 工具           | 用途       | 可重用性 | 复杂度       |
| -------------- | ---------- | -------- | ------------ |
| std::call_once | 一次性执行 | 否       | 简单         |
| std::mutex     | 通用同步   | 是       | 较复杂       |
| std::atomic    | 原子操作   | 是       | 轻量但有限制 |

------

总结

std::call_once 是一个强大且简单的工具，用于确保函数在多线程环境中只执行一次。它与 std::once_flag 配合，广泛应用于单例模式、延迟初始化等场景。它的设计避免了手动锁管理的复杂性，同时提供了异常安全的保证。如果你有具体问题或使用案例，欢迎进一步讨论！