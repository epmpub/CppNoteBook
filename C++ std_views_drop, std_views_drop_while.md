# std::views::drop, std::views::drop_while

The C++20 views for omitting leading elements: std::views::drop, std::views::drop_while.

The C++20 *std::views::drop* and *std::views::drop_while* are views that omit leading elements of the underlying range.

*std::views::drop* will omit the specified number of leading elements. *std::views::drop_while* will omit the leading elements that satisfy the provided predicate.

```c++
#include <vector>
#include <ranges>

std::vector<int> data{1,2,3,4,5};

// iterate over {2,3,4,5}
for (auto v : data | std::views::drop(1)) {}

constexpr auto less_three = [](int v) { return v < 3; };
// iterate over {3,4,5}
for (auto v : data | std::views::drop_while(less_three)) {}
```

