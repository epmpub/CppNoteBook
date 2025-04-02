

#  std::views::join

The C++20 std::views::join will produce a view over the elements of **sub-ranges.** 

Effectively joining the sub-ranges into a single range.

```c++
#include <ranges>
#include <vector>
#include <numeric>
#include <iostream>

int main() {
    std::vector<std::vector<int>> data{{1,2,3},{4,5,6},{7,8,9}};

    // Simple join over the 2nd dimension of the array
    auto view1 = data | std::views::join;
    // view1 == {1, 2, 3, 4, 5, 6, 7, 8, 9}

    for (auto v : view1)
        std::cout << v << " ";
    std::cout << "\n";

    // Join with a filter, skipping sub-ranges with odd sum
    // std::accumulate 的
	// 功能：对子数组 rng 的所有元素求和，初始值为 0。
    auto view2 = data | std::views::filter([](const auto& rng) {
            return std::accumulate(rng.begin(), rng.end(), 0) % 2 == 0;
        }) | std::views::join;
    // view2 == {1, 2, 3, 7, 8, 9}

    for (auto v : view2)
        std::cout << v << " ";
    std::cout << "\n";
}
```

