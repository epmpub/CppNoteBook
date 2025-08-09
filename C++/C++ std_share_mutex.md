# std::share_mutex

```c++
#include <iostream>
#include <mutex>
#include <shared_mutex>
#include <syncstream>
#include <thread>
 
class ThreadSafeCounter
{
public:
    ThreadSafeCounter() = default;
 
    // Multiple threads/readers can read the counter's value at the same time.
    unsigned int get() const
    {
        std::shared_lock lock(mutex_);
        return value_;
    }
 
    // Only one thread/writer can increment/write the counter's value.
    void increment()
    {
        std::unique_lock lock(mutex_);
        ++value_;
    }
 
    // Only one thread/writer can reset/write the counter's value.
    void reset()
    {
        std::unique_lock lock(mutex_);
        value_ = 0;
    }
 
private:
    mutable std::shared_mutex mutex_;
    unsigned int value_{};
};
 
int main()
{
    ThreadSafeCounter counter;
 
    auto increment_and_print = [&counter]()
    {
        for (int i{}; i != 3; ++i)
        {
            counter.increment();
            std::osyncstream(std::cout)
                << std::this_thread::get_id() << ' ' << counter.get() << '\n';
        }
    };
 
    std::thread thread1(increment_and_print);
    std::thread thread2(increment_and_print);
 
    thread1.join();
    thread2.join();
}
```

## 注意(osyncstream):

 std::osyncstream(std::cout) << std::this_thread::get_id() << ' ' << counter.get() << '\n';

C++20 standard



> The class template `std::basic_osyncstream` is a convenience wrapper for [`std::basic_syncbuf`](http://127.0.0.1:57176/c__/en.cppreference.com/w/cpp/io/basic_syncbuf.html). It provides a mechanism to synchronize threads writing to the same stream.

It can be used with a named variable:

```
{
    std::osyncstream synced_out(std::cout); // synchronized wrapper for std::cout
    synced_out << "Hello, ";
    synced_out << "World!";
    synced_out << std::endl; // flush is noted, but not yet performed
    synced_out << "and more!\n";
} // characters are transferred and std::cout is flushed
```

as well as with a temporary:

```
std::osyncstream(std::cout) << "Hello, " << "World!" << '\n';
```