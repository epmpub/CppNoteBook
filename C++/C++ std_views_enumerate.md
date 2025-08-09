# std::views::enumerate

```C++
#include <vector>
#include <ranges>
#include <iostream>

int main() {
    std::vector<int> data{1, 2, 3, 4, 5, 6, 7};
    
    // Applying enumerate before other views to inject original indexes
    constexpr auto before = std::views::enumerate | std::views::filter([](auto t) {
        return std::get<1>(t) % 2 == 0;
    });
    for (auto [idx, value] : data | before) {
        // Iterate over: {1,2}, {3,4}, {5,6}
        std::cout << idx << " -> " << value << "\n";
    }
    std::cout << "\n";

    // Applying enumerate after other views to index the resuling range
    constexpr auto after = std::views::filter([](int v) { 
        return v % 2 == 0;
    }) | std::views::enumerate;
    for (auto [idx, value] : data | after) {
        // Iterate over: {0,2}, {1,4}, {2,6}
        std::cout << idx << " -> " << value << "\n";
    }
    std::cout << "\n";

    // Since the second element of the tuple is a reference,
    // we maintain mutability
    for (auto [idx, value] : data | std::views::enumerate) {
        value -= idx;
    }
    // data == {1,1,1,1,1,1,1}

    for (auto v : data)
        std::cout << v << " ";
    std::cout << "\n";
}
```

