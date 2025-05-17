# C++17 std::shared_timed_mutex 结合超时

```C++
#include <shared_mutex>
#include <vector>
#include <string>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    std::vector<std::string> openIssues;
    std::shared_timed_mutex openIssuesMx;

    // 写线程：长时间持有独占锁
    std::thread writer([&] {
        std::unique_lock<std::shared_timed_mutex> ul(openIssuesMx);
        openIssues.emplace_back("case1");
        std::this_thread::sleep_for(std::chrono::seconds(2));
        std::cout << "Writer: Added case1\n";
        });

    // 读线程：尝试获取共享锁
    std::thread reader([&] {
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        std::shared_lock<std::shared_timed_mutex> sl(openIssuesMx, std::defer_lock);
        if (sl.try_lock_for(std::chrono::milliseconds(2000))) {
            std::cout << "Reader: Size = " << openIssues.size() << "\n";
        }
        else {
            std::cout << "Reader: Timed out\n";
        }
        });

    writer.join();
    reader.join();

    return 0;
}
```

