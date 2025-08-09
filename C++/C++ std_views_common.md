# std::views::common

Before C++20, a range was an implicit concept represented by two iterators. With C++20, this concept was formalized and relaxed to an iterator and a sentinel.

To adapt a range for old code that requires a common range (iterator and sentinel of the same type), we can use the *std::views::common* adapter view.



在 C++20 之前，范围是一个隐含的概念，由两个迭代器表示。在 C++20 中，这个概念被形式化并放宽为**迭代器和哨兵**。

为了适应需要公共范围（相同类型的迭代器和哨兵）的旧代码的范围，我们可以使用*std::views::common*适配器视图。



```c++
#include <ranges>
#include <numeric>

// std::views::iota(1) is not a common view
auto view = std::views::iota(1) | std::views::take(3);

// Will not compile, view.begin() and view.end() are of different types
// int sum = std::accumulate(view.begin(), view.end(), 0);

auto common_view = view | std::views::common;
int sum = std::accumulate(common_view.begin(), common_view.end(), 0);
// OK, sum == 6
```

