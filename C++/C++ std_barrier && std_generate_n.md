# *std::barrier* && std::generate_n

*std::barrier* is a C++20 synchronization primitive that enables the creation of synchronized execution phases across multiple threads.

A *std::barrier* is initialized with a count (the number of threads). When a thread arrives at a barrier, it can block until all other threads arrive or drop, decreasing the counter.

Unlike ***std::latch*** (which provides similar functionality), the *std::barrier* is re-usable and will call an optional completion function before unblocking the waiting threads.

```cpp
#include <barrier>
#include <vector>
#include <thread>
#include <algorithm>
#include <random>
#include <print>

// Barrier with a completion function that prints
// the phase information:
std::barrier phase(4,[id = 1] mutable {
    std::println("Phase {} complete.", id);
    id++;
});

std::vector<std::jthread> runners;
// Start 4 threads:
std::generate_n(std::back_inserter(runners), 4, [&phase]{
    return std::jthread([&phase]{
        std::println("Running phase 1 for thread {}",
                     std::this_thread::get_id());

        std::this_thread::yield();
        // block until all threads arrive
        phase.arrive_and_wait();

        std::println("Running phase 2 for thread {}",
                     std::this_thread::get_id());

        std::this_thread::yield();
        // block until all threads arrive
        phase.arrive_and_wait();
    });
});
runners.clear(); // join the threads

// This is guaranteed to print:
// Running phase 1 for thread xxxxx (for each of the four threads)
// Phase 1 complete.
// Running phase 2 for thread xxxxx (for each of the four threads)
// Phase 2 complete.


// Barrier without a custom completion function:
std::barrier other(5);
std::vector<std::jthread> runners2;
// Start 5 threads:
std::generate_n(std::back_inserter(runners2), 5, [&other]{
    return std::jthread([&other]{
        // Use thread id hash as the seed to get different
        // behaviour for each thread
        size_t seed = 
            std::hash<std::thread::id>{}(std::this_thread::get_id());
        std::mt19937 gen(seed);

        // 30% chance to produce true, 70% false
        std::bernoulli_distribution done(0.3); 

        int id = 1;
        while (true) {
            std::println("Running phase {} for thread {}", 
                         id, std::this_thread::get_id());

            std::this_thread::yield();
            if (done(gen)) { // 30% chance for the thread to stop
                // decrease the barrier initial counter
                // so that it waits for n-1 threads
                other.arrive_and_drop();
                return;
            }

            // Otherwise block until all (still running)
            // threads arrive.
            other.arrive_and_wait();
            ++id;
        }
    });
});
// This will print consecutive phases, with the number of 
// threads in each phase randomly decreasing.
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/Mo6EqWEK3)