# std *::partial_sort_copy*

The *std::partial_sort_copy* is an unusual sorting algorithm. It doesn’t require the source range to be random-access while providing *O(n\*logk)* runtime complexity.

The algorithm will “copy” the top *k* elements to the output range (which is required to be random access) in sorted order.

The sorting isn’t stable, i.e. it does not maintain the order of equal elements.

Both C++17 parallel and C++20 range versions are available.



std *::partial_sort_copy*是一种不常见的排序算法。它不需要源范围是随机访问的，同时提供*O(n\*logk) 的*运行时复杂度。

算法将按排序顺序将前*k 个*元素“复制”到输出范围（需要随机访问）。

排序不稳定，即它不能保持相等元素的顺序。

C++17 并行和 C++20 范围版本均可用。



```c++
#include <forward_list>
#include <string>
#include <string_view>
#include <vector>
#include <algorithm>


std::forward_list<std::string> names{
  "Emma", "Liam", "Zara", "Ethan", "Aria", "Mateo", "Ivy", "Finn",
  "Luna", "Kai", "Mila", "Oscar", "Ruby", "Levi", "Nora"};

// The size of the output range determines the number 
// of copied/sorted elements
std::vector<std::string> out1(3);

std::partial_sort_copy(
    names.begin(), names.end(), // source range
    out1.begin(), out1.end());  // destination range
// out1 == {"Aria", "Emma", "Ethan"}


std::vector<std::string> out2(5);

std::partial_sort_copy(
    names.begin(), names.end(), // source range
    out2.begin(), out2.end(),   // destination range
    std::greater<>{});          // custom comparator
// out2 == {"Zara", "Ruby", "Oscar", "Nora", "Mila"}


std::vector<std::string> out3(4);

// Range version
auto length = [](const std::string& s) { return s.length(); };
std::ranges::partial_sort_copy(names, out3, // ranges
    std::greater<>{}, // custom comparator
    length,           // projection for source range
    length);          // projection for destination range
// out3 ~= {"Oscar", "Mateo", "Ethan", "Zara"}


// Because of separate projections the destination range
// can use a different element type

std::vector<std::string_view> out4(4);

auto view_len = [](const std::string_view& s) { return s.length(); };
std::ranges::partial_sort_copy(names, out4, // ranges
    std::greater<>{}, // greater supports std::string/std::string_view
                      // comparison without conversions
    length,           // projection for std::string
    view_len);        // projection for std::string_view
// out4 ~= {"Oscar", "Mateo", "Ethan", "Zara"}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/aYbYovEnv)