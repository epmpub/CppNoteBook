# thread_local

这段代码展示了 C++20 中多线程编程的一些特性，包括 std::jthread（可加入和支持停止的线程）、thread_local 存储以及 std::stop_token 用于线程停止控制。以下是对代码的逐步解释。

------

**代码分解**

**1. task 函数**

cpp

```cpp
void task() {
    thread_local std::vector<int> data;
    data.clear(); // Drop left-over elements from previous iteration,
                  // without de-allocating memory.
  
    // read-some data and do some processing
}
```

- **thread_local std::vector<int> data**:
  - thread_local 是 C++11 引入的存储类说明符，表示每个线程拥有独立的 data 实例。
  - 在这里，每个调用 task() 的线程都有自己的 std::vector<int>，与其他线程隔离。
  - 该向量在第一次调用时构造，并在线程结束时销毁。
- **data.clear()**:
  - 清空向量中的元素，但保留已分配的内存（容量不变）。
  - 注释说明：避免释放内存，以便在下一次迭代中重用，提升性能。
- **作用**: 
  - task() 表示一个任务，可能读取数据并处理。
  - 使用 thread_local 确保每个线程独立操作自己的数据，避免竞争条件。

------

**2. runner 函数**

cpp

```cpp
void runner(std::stop_token stop_token) {
    while (!stop_token.stop_requested()) {
        task();
    }
}
```

- **std::stop_token stop_token**:
  - C++20 引入，用于线程停止控制。
  - stop_token 是 std::jthread 提供的对象，表示线程的停止状态。
- **stop_token.stop_requested()**:
  - 返回 true 表示线程被请求停止，返回 false 表示继续运行。
- **循环**:
  - 当线程未被请求停止时，反复调用 task()。
- **作用**: 
  - runner 是线程的主循环，持续执行任务直到收到停止信号。

------

**3. main 函数**

cpp

```cpp
int main() {
    std::jthread t1(runner);
    std::jthread t2(runner);
    using namespace std::literals::chrono_literals;
    std::this_thread::sleep_for(5s);
}
```

- **std::jthread t1(runner) 和 t2(runner)**:
  - std::jthread 是 C++20 引入的线程类，改进自 std::thread。
  - 特点：
    1. **自动加入**: 析构时自动调用 join()，无需手动管理。
    2. **停止支持**: 提供 std::stop_token，允许请求线程停止。
  - 这里启动两个线程，分别执行 runner 函数。
- **std::this_thread::sleep_for(5s)**:
  - 使用 C++14 的时间字面量（5s 表示 5 秒）。
  - 主线程休眠 5 秒，让两个线程运行。
- **隐式行为**:
  - 主函数结束时，t1 和 t2 析构，自动调用 request_stop()（停止请求）和 join()（等待线程结束）。

------

**运行流程**

1. **启动**:
   - 主线程创建 t1 和 t2，两个线程开始执行 runner。
2. **执行**:
   - 每个线程进入 runner，循环调用 task()。
   - 每个线程有自己的 thread_local data，互不干扰。
   - task() 清空 data 并执行（未实现的）处理逻辑。
3. **休眠**:
   - 主线程休眠 5 秒，线程 t1 和 t2 持续运行。
4. **结束**:
   - 5 秒后，main 结束，t1 和 t2 析构。
   - std::jthread 自动请求停止（stop_token.stop_requested() 变为 true）。
   - runner 循环退出，线程结束，主线程等待它们完成。

------

**关键特性**

1. **thread_local**:
   - 确保每个线程有独立的 data，避免数据竞争。
   - clear() 重用内存，提升性能。
2. **std::jthread**:
   - 比 std::thread 更安全，自动管理生命周期。
   - 支持停止机制，无需手动信号量或标志。
3. **std::stop_token**:
   - 提供优雅的线程停止方式，线程可主动检查。

------

**中文解释**

**功能**

- **多线程任务**: 启动两个线程，循环执行任务 5 秒。
- **数据隔离**: 用 thread_local 确保每个线程有独立的数据。
- **停止控制**: 用 std::jthread 和 std::stop_token 管理线程生命周期。

**代码部分**

- **task()**: 定义线程任务，清理并重用本地向量。
- **runner()**: 线程主循环，检查停止信号并调用任务。
- **main()**: 启动线程，休眠后自动结束。

**运行**

- 两个线程并行运行 task()，各自维护 data。
- 5 秒后，主线程退出，jthread 停止并加入线程。

------

**潜在改进**

1. **任务实现**: 当前 task() 未实现具体逻辑，可添加数据读取和处理。
2. **停止控制**: 可手动调用 t1.request_stop() 提前停止。
3. **异常处理**: 添加 try-catch 处理任务中的潜在异常。

------

**完整示例（带输出）**

cpp

```cpp
#include <vector>
#include <thread>
#include <iostream>

void task() {
    thread_local std::vector<int> data;
    data.clear();
    data.push_back(std::hash<std::thread::id>{}(std::this_thread::get_id()));
    std::cout << "Thread " << data[0] << " running\n";
}

void runner(std::stop_token stop_token) {
    while (!stop_token.stop_requested()) {
        task();
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
    }
}

int main() {
    std::jthread t1(runner);
    std::jthread t2(runner);
    std::this_thread::sleep_for(std::chrono::seconds(5));
}
```

- **输出（示例）**:

  ```text
  Thread 12345 running
  Thread 67890 running
  Thread 12345 running
  Thread 67890 running
  ...
  ```

- **解释**: 每个线程打印其 ID，循环运行约 5 秒。

------

**总结**

这段代码展示了 C++20 多线程编程的现代特性：

- thread_local 提供线程局部存储。
- std::jthread 简化线程管理和停止。
- std::stop_token 实现优雅的线程控制。

如果你有具体问题或想扩展功能，请告诉我！