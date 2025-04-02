# std::views::keys and std::views::values

The C++20 *std::views::keys* and *std::views::values* are specialized views that represent the views over the keys and values of associate containers.

Reminder: for *std::set* and *std::unordered_set* (and their multi-variants), the elements are keys, not values.

```c++
#include <unordered_map>
#include <ranges>

int main() {
    std::unordered_map<int,int> data{{1,7}, {2,8}, {3,9}};

    for (const auto &v : data | std::views::keys) {
        // iterate over 1, 2, 3
    }

    for (const auto &v : data | std::views::values) {
        // iterate over 7, 8, 9
    }
}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/z7Pqq8WjT)