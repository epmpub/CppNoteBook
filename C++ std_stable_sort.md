#  std::stable_sort

The stable sort algorithm: std::stable_sort.

**The *std::stable_sort* is a slower version of *std::sort* that additionally provides stability,** 

i.e. equivalent elements maintain their relative positions.

This is important, notably for interactive cases when one range can be repeatedly sorted based on different aspects.

```c++
#include <algorithm>
#include <vector>

struct Data {
    int a;
    int b;
    int c;
};

std::vector<Data> data{
    {0, 1, 1},
    {0, 2, 1},
    {1, 1, 2},
    {1, 2, 2},
    {2, 1, 3},
    {2, 2, 3},
};

// Sort by b
std::ranges::stable_sort(data, {}, &Data::b);
// Guaranteed order:
// {{0,1,1},{1,1,2},{2,1,3},{0,2,1},{1,2,2},{2,2,3}}

// Sort by c
std::ranges::stable_sort(data, {}, &Data::c);
// Guaranteed order:
// {{0,1,1},{0,2,1},{1,1,2},{1,2,2},{2,1,3},{2,2,3}}

std::vector<std::string> labels{
   "a", "aa", "aaa", "b", "bb", "bbb", 
   "c", "cc", "ccc", "d", "dd", "ddd"
};

std::ranges::stable_sort(labels, {}, [](const auto& l) {
    return l.length();
});
// Guaranteed order:
// "a", "b", "c", "d", "aa", "bb", "cc", "dd", 
// "aaa", "bbb", "ccc", "ddd"
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/jEsW9Gzsf)