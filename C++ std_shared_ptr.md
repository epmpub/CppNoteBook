# std *::shared_ptr*

std *::shared_ptr*是一个功能强大但重量较重的引用计数智能指针。

指向同一个块的*std::shared_ptr*的不同实例可以安全地处理，无需线程同步。

*std::shared_ptr*可用于多线程代码中的数据交接。生产者可以放弃所有权，同时确保消费者不会失去对数据的访问权。



```C++
#include <vector>
#include <thread>
#include <random>
#include <algorithm>
#include <memory>

int main() {
    std::vector<std::jthread> runners(4);

    {
    // some simple data generation (not that relevant)
    std::vector<uint32_t> data_in;
    std::mt19937 rng;
    std::generate_n(std::back_inserter(data_in), 
                    1024*1024, std::ref(rng));

    // Create a shared pointer from the vector.
    // Note the const, making the data immutable.
    auto shared =
        std::make_shared<const std::vector<uint32_t>>(std::move(data_in));

    // Start the threads, giving each thread a copy of the shared_ptr
    for (auto &thread : runners) {
        thread = std::jthread([shared]() {
            // accessing const-methods of STL containers is thread-safe
            for (auto v : *shared) { 
                // process data...
            }
        });
    }
    }

    // 4 runner threads are all running, but the local reference
    // to the data is no longer held.

    // The data will be released once all runners finish, thus releasing
    // their references to the data, i.e. the ref-count reaches zero.

    // std::jthread auto-joins on destruction, or we can do it explicitly:
    for (auto &thread : runners) {
        thread.join();
    }
}
```

