# *std::sample* 

随机取样

The *std::sample* algorithm (C++17) is a stable (maintains relative order of elements) sampling algorithm that randomly copies the specified number of elements from the source range into the destination range (output iterator).



```c++
#include <algorithm>
#include <vector>
#include <random>
#include <iterator>
#include <iostream>

int main() {
    std::vector<int> data{1, 2, 3, 4, 5, 6, 7, 8, 9};
    std::vector<int> out;

    auto gen = std::mt19937(1); // fixed seed for deterministic result

    std::sample(data.begin(), data.end(),   // input range
        std::back_inserter(out),            // output iterator
        4,      // number of elements to sample
        gen);   // random number engine

    for (auto v : out)
        std::cout << v << " ";
    std::cout << "\n";

    // stdlibc++: out == {1, 6, 8, 9}
    // libc++: out == {2, 4, 5, 9}
}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/EWazzMbns)