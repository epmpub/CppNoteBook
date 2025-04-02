

# use C++20 views  instead of raw for-loops.

The range-based loop was introduced in C++11; 

however, only with the introduction of C++20 views can it replace almost all instances for raw for-loops.



```c++
#include <algorithm>
#include <vector>
#include <ranges>
#include <iostream>

int main() {
    // C++11: iterate over values in an initializer list
    for (auto v : {1, 1, 2, 3, 5, 8, 13})
        std::cout << v << " ";
    std::cout << "\n";

    // C++20: iterate over [1,10), values 1..9
    for (auto v : std::views::iota(1, 10))
        std::cout << v << " ";
    std::cout << "\n";

    // C++20: downward iteration 5..-5
    for (auto v : std::views::iota(-5, 6) | std::views::reverse)
        std::cout << v << " ";
    std::cout << "\n";

    std::vector<int> data{1, 2, 3, 4, 5, 6, 7, 8, 9};
    auto it = std::ranges::lower_bound(data, 5);

    // C++20: iterating over sub-range, 1..4
    for (auto v : std::ranges::subrange(data.begin(), it))
        std::cout << v << " ";
    std::cout << "\n";

    // C++20: iterating over sub-range, 5..9
    for (auto v : std::ranges::subrange(it, data.end()))
        std::cout << v << " ";
    std::cout << "\n";

    // C++23: nested loop replacement
    // iterate over i==[0,5), j==[0,5)
    for (auto [i,j] : std::views::cartesian_product(
                        std::views::iota(0,5),
                        std::views::iota(0,5)))
        std::cout << "i == " << i << ", j == " << j << "\n";
}
```

