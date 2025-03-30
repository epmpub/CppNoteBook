spdlog

以下是一个使用 C++ spdlog 库的多线程日志记录示例。该示例展示了如何在多线程环境中安全地共享同一个日志对象：

```cpp
#include <spdlog/spdlog.h>
#include <spdlog/sinks/basic_file_sink.h>
#include <vector>
#include <thread>
#include <memory>

// 线程函数
void thread_worker(int thread_id, std::shared_ptr<spdlog::logger> logger)
{
    for (int i = 0; i < 3; ++i) {
        // 使用格式字符串记录日志
        logger->info("Thread {} - Message {}", thread_id, i);
        
        // 也可以使用流式语法
        logger->warn() << "Thread " << thread_id << " - Warning " << i;
    }
}

int main()
{
    try {
        // 创建线程安全的文件日志器（自动处理多线程同步）
        auto file_logger = spdlog::basic_logger_mt("basic_logger", "logs/multithread.log");
        
        // 设置全局日志级别为 info
        spdlog::set_level(spdlog::level::info);

        const int num_threads = 5;
        std::vector<std::thread> threads;

        // 创建多个工作线程
        for (int i = 0; i < num_threads; ++i) {
            threads.emplace_back(thread_worker, i + 1, file_logger);
        }

        // 等待所有线程完成
        for (auto& t : threads) {
            t.join();
        }

        // 程序结束时自动 flush 并关闭所有日志器
        spdlog::shutdown();
    }
    catch (const spdlog::spdlog_ex& ex) {
        std::cerr << "Log initialization failed: " << ex.what() << std::endl;
        return 1;
    }

    return 0;
}
```

关键点说明：

1. **线程安全**：
   - 使用 `basic_logger_mt` 创建多线程安全的日志器（`_mt` 后缀表示 multi-thread）
   - spdlog 内部使用互斥锁保证线程安全，无需额外同步

2. **日志器共享**：
   - 通过 `std::shared_ptr` 共享日志器对象
   - 所有线程使用同一个日志器实例

3. **输出示例**：
```log
[2023-10-01 12:34:56.789] [basic_logger] [info] Thread 1 - Message 0
[2023-10-01 12:34:56.790] [basic_logger] [warning] Thread 2 - Warning 0
[2023-10-01 12:34:56.791] [basic_logger] [info] Thread 3 - Message 0
...
```

4. **异步日志（高性能场景）**：
```cpp
// 添加这行头文件
#include <spdlog/async.h>

// 在 main 函数开始处初始化异步日志
spdlog::init_thread_pool(8192, 1);  // 队列大小 8192，1 个后台线程
auto async_file = std::make_shared<spdlog::sinks::basic_file_sink_mt>("logs/async.log");
auto async_logger = std::make_shared<spdlog::async_logger>(
    "async_logger", async_file, spdlog::thread_pool());
```

5. **编译注意事项**：
   - 需要 C++11 或更高版本
   - 链接 spdlog 库和其依赖（如 fmt）
   - 示例使用头文件版本时需要将 spdlog 添加到包含路径

这个示例展示了：
- 多线程环境下的安全日志记录
- 两种日志格式（格式化字符串和流式语法）
- 异常处理
- 日志器的生命周期管理
- 异步日志配置选项

可以根据实际需求调整日志格式、输出目标（文件/控制台）和日志级别。spdlog 会自动处理不同线程日志消息的排队和顺序问题。