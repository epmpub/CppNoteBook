C++17 timed_mutex scoped_lock

```C++
#include <mutex>
#include <vector>
#include <string>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    std::vector<std::string> openIssues;
    std::timed_mutex openIssuesMx;

    // 模拟一个线程长时间持有 openIssuesMx
    std::thread t1([&] {
        std::unique_lock lg(openIssuesMx);
        std::cout << "Thread 1: Holding lock for 2 seconds\n";
        std::this_thread::sleep_for(std::chrono::seconds(2));
        openIssues.emplace_back("case1");
        });

    // 另一个线程尝试获取 openIssuesMx
    std::thread t2([&] {
        std::this_thread::sleep_for(std::chrono::milliseconds(100)); // 稍后尝试
        std::unique_lock<std::timed_mutex> ul(openIssuesMx, std::defer_lock);
        if (ul.try_lock_for(std::chrono::seconds(2))) {
            std::cout << "Thread 2: Acquired lock\n";
            openIssues.emplace_back("case2");
        }
        else {
            std::cout << "Thread 2: Timed out, could not acquire lock\n";
        }
        });

    t1.join();
    t2.join();

    // 打印结果
    std::cout << "Open issues: ";
    for (const auto& issue : openIssues) {
        std::cout << issue << " ";
    }
    std::cout << std::endl;

    return 0;
}
```



```C++
#include <mutex>
...
std::vector<std::string> allIssues;
std::mutex allIssuesMx;
std::vector<std::string> openIssues;
std::timed_mutex openIssuesMx;

// 同时锁住两个issue列表：
{
    std::scoped_lock lg(allIssuesMx, openIssuesMx);
    ... // 操作allIssues和openIssues
}
```

注意根据类模板参数推导，声明`lg`时你不需要指明互斥量的类型。

这个示例等价于下面的C++11代码：

```C++
// 锁住两个issue列表：
{
    std::lock(allIssuesMx, openIssuesMx);   // 以避免死锁的方式上锁
    std::lock_guard<std::mutex> lg1(allIssuesMx, std::adopt_lock);
    std::lock_guard<std::mutex> lg2(openIssuesMx, std::adopt_lock);
    ... // 操作allIssues和openIssues
}
```





